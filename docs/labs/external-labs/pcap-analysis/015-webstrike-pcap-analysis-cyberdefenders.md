# WebStrike - PCAP Analysis (CyberDefenders)

## Scenario
A suspicious file was identified on a company web server, raising alarms within the intranet.
The Development team flagged the anomaly, suspecting potential malicious activity. 
To address the issue, the network team captured critical network traffic and prepared a PCAP file for review.
Your task is to analyze the provided PCAP file to uncover how the file appeared and determine the extent of any unauthorized activity.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/webstrike/

> [!IMPORTANT]
> The questions below are not in the original lab order. I arranged them in the order that best matched my investigation flow.

> [!NOTE]
The packet list is being read in chronological order. Wireshark time formatting can be changed from View → Time Display Format, depending on how you want the sequence to be displayed.
>

### Q1 - Identifying the geographical origin of the attack facilitates the implementation of geo-blocking measures and the analysis of threat intelligence. From which city did the attack originate?

I first filtered the traffic to `HTTP`, then opened `Statistics → Conversations`. Since there was only one visible conversation, that immediately narrowed the scope.

<img width="203" height="78" alt="immagine" src="https://github.com/user-attachments/assets/bdac5211-09f8-44f0-9149-433d1aab96df" />

I reviewed the packets in that exchange and scrolled through them until I reached the later part of the traffic, where the behavior became clearly suspicious, as shown in the screenshot.

<img width="1040" height="345" alt="immagine" src="https://github.com/user-attachments/assets/6da0e833-8612-464b-a1fc-14c47481d145" />

Based on that, I treated the external source IP as the attacker IP in the context of the lab. After that, I used an external IP geolocation service outside the lab environment to look up the city associated with that address.

<img width="645" height="144" alt="immagine" src="https://github.com/user-attachments/assets/5629cda3-7a22-4124-931e-9af4a4d18695" />

<img width="1402" height="298" alt="immagine" src="https://github.com/user-attachments/assets/5a20ab59-2863-410e-a84b-1917fa1e7a1c" />

**Answer:** `Tianjin`

> [!NOTE]
In a realistic investigation, identifying the attacker host or attributing an IP as the source of the attack would usually come much later, after reconstructing the traffic, roles, actions, and timeline. Here, that conclusion was reached much earlier only because the lab is closed and heavily simplified.

### Q2 - Knowing the attacker's User-Agent assists in creating robust filtering rules. What's the attacker's Full User-Agent?

Since we had already identified this host as the attacker in the context of the lab, we could open any of his related packets and see that the User-Agent was the following:

<img width="1864" height="284" alt="immagine" src="https://github.com/user-attachments/assets/c3b4faaf-7f20-4e00-a1fa-bc641e486dd1" />

**Answer:** `Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0`

### Q4 - Identifying the directory where uploaded files are stored is crucial for locating the vulnerable page and removing any malicious files. Which directory is used by the website to store the uploaded files?

<img width="2115" height="239" alt="immagine" src="https://github.com/user-attachments/assets/03c4be49-5ffb-4e71-9035-4c4f13b89e3a" />

Here, the attacker appears to be testing possible upload directories. 
After a few `404 Not Found` responses, the server gave the answer by redirecting the request for `/reviews/uploads`, which revealed the correct upload directory.

**Answer:** `/reviews/uploads`

### Q3 - We need to determine if any vulnerabilities were exploited. What is the name of the malicious web shell that was successfully uploaded?

At this point, the uploaded file can already be identified directly from the request path, `/reviews/uploads/image.jpg.php`, so there was no need for a more complex reconstruction. 
The key information was already visible in the packet list itself.

Immediately after the uploaded file becomes visible in the request path, the packet list also shows TCP traffic to port `8080`, so the key elements were already available without needing deeper reconstruction.

<img width="2048" height="66" alt="immagine" src="https://github.com/user-attachments/assets/539f51d4-6128-4485-ad1d-657a12a5a8e9" />

**Answer:** `image.jpg.php`
### Q5 - Which port, opened on the attacker's machine, was targeted by the malicious web shell for establishing unauthorized outbound communication?

Remember the screenshot used for the first question? We had already focused on the right packet there.
By opening it with Follow HTTP Stream, we can now read the relevant content more clearly and use it to answer this question as well.

<img width="1856" height="252" alt="immagine" src="https://github.com/user-attachments/assets/af7ab3ce-9cc5-4e45-85d7-934ab43c9ef2" />

<img width="414" height="261" alt="immagine" src="https://github.com/user-attachments/assets/2cff5103-47f5-49d0-aa4f-f8e0d5072da1" /> 

<img width="700" height="63" alt="immagine" src="https://github.com/user-attachments/assets/ff1b78cd-ac2b-49bc-8349-668122b0d818" />

The attacker used the shell to perform basic host reconnaissance, verify the execution context, and...

**Answer:** `8080`
### Q6 -  Recognizing the significance of compromised data helps prioritize incident response actions. Which file was the attacker attempting to exfiltrate?

...exfiltrate `/etc/passwd` to the remote host.

**Answer:** `passwd`

> [!NOTE]
I intentionally avoided adding unnecessary complexity here, because in this lab it would have introduced more noise than value.
The relevant information was already visible with simple filtering and direct packet review.
>



