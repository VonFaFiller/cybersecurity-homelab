# JetBrains Lab PCAP Analysis (CyberDefenders)

## Scenario
During a recent security incident, an attacker successfully exploited a vulnerability in our web server, allowing them to upload webshells and gain full control over the system. 
The attacker utilized the compromised web server as a launch point for further malicious activities, including data manipulation. 

As part of the investigation, You are provided with a packet capture (PCAP) of the network traffic during the attack to piece together the attack timeline and identify the methods used by the attacker. The goal is to determine the initial entry point, the attacker's tools and techniques, and the compromise's extent.

## References
-https://cyberdefenders.org/blueteam-ctf-challenges/jetbrains/

### Q1 - Identifying the attacker's IP address helps trace the source and stop further attacks. What is the attacker's IP address?

For the first question, I immediately isolated the HTTP traffic and then went to Statistics → Conversations, where I sorted the conversations by packet count.

<img width="330" height="229" alt="immagine" src="https://github.com/user-attachments/assets/ef2f5092-df51-46c1-83ba-7b6c4793cb94" />

Because the exercise is a closed lab, this was enough to get a quick idea of which hosts were likely acting as the server, but it was not a real attribution by itself.
The decisive clue appeared almost too early: while reviewing the HTTP requests, I quickly scrolled down and saw a suspicious POST request referencing a plugin path.

<img width="883" height="514" alt="immagine" src="https://github.com/user-attachments/assets/8b2832f6-5994-4e36-b4bd-a7c5d60ee8f4" />

After opening it, I saw a "whoami" command, which strongly suggested that this was not normal application behavior but a malicious plugin being used for command execution. 
I also checked the commands immediately above it first, to confirm that this request fit into the same suspicious sequence rather than being an isolated artifact.
I then checked the source of that request, saw that it came from the 23.158.56.196 host, reviewed some of the surrounding traffic from other hosts, and concluded that this was the attacker IP in the context of the lab. 

<img width="466" height="64" alt="immagine" src="https://github.com/user-attachments/assets/02fc6fe1-532f-4756-8e6f-1834f7c0f2c8" />

### Answer: 23.158.56.196
> [!NOTE]
This reasoning worked here only because the dataset is closed and the question is asked far too early.
In a realistic investigation, identifying the attacker IP would make sense much later, only after reconstructing the roles, requests, actions, and sequence of events. 
As a first question, it is poorly placed, because once the attacker IP is known, a simple source-IP filter makes much of the remaining exercise easy to answer.
>
### Q6 - When did the attacker execute their first command via the web shell?

I used the filter `ip.src == 23.158.56.196 && frame contains "cmd" && http` to isolate the visible web-shell requests. Once those requests were grouped together, it was straightforward to read them in chronological order and identify when the first command was executed through the web shell.

<img width="1191" height="141" alt="immagine" src="https://github.com/user-attachments/assets/66251a79-b4d4-4fe9-8f5b-3171233ca64e" />

### Answer: 2024-06-30 08:03
### Q9 - The attacker tampered with a text file that contained the credentials of the admin user of the webserver. What new username and password did the attacker write in the file?

For this question, I scrolled further down the request list, since there were only a few iterations to review. The modified text file appeared almost immediately, making the new username and password easy to identify. For a clearer view of the full context, Follow HTTP Stream can also be used.

<img width="817" height="75" alt="immagine" src="https://github.com/user-attachments/assets/9b383c58-13c7-459e-9650-9f07a69a0ed6" />

### Answer: docker run --rm -it -v /:/host ubuntu chroot /host
### Q7 - The attacker tried to escape from the container but he didn’t succeed, What is the command that he used for that?

This question could be answered quickly by keeping the requests in chronological order and following the cmd sequence. 
By reading the commands in order, it was easy to spot which one appeared to fail just before the next action that seemed to succeed, which exposed the attempted container-escape command.

<img width="1038" height="135" alt="immagine" src="https://github.com/user-attachments/assets/b7a1048f-7704-4788-8615-1d1d1a4d8468" />

<img width="603" height="70" alt="immagine" src="https://github.com/user-attachments/assets/d2750eff-5f42-41a3-a0df-fc5127bf4f4e" />

### Answer: a1l4m:youarecompromised
> [!NOTE]
>I Repeat: this reasoning was only possible because the attacker host had already been identified much earlier in the lab. In a realistic investigation, that attribution would normally come later, after a fuller reconstruction of the traffic and actions. Here, the closed nature of the dataset made this kind of shortcut possible.

### Q2 - To identify potential vulnerability exploitation, what version of our web server service is running?

For the second question, I filtered the traffic with the attacker source `IP and HTTP: ip.src == 23.158.56.196 && http`.
From there, I opened `Follow HTTP Stream` on one of the packets that already contained the suspicious plugin path, `/plugins/NSt8bHTg/NSt8bHTg.jsp.`
At that stage, there was no real need to reconstruct the exact beginning of the activity, because once the attacker IP and the plugin path were already known, jumping into any of those requests was enough to expose the relevant context directly in the stream.

<img width="1184" height="29" alt="immagine" src="https://github.com/user-attachments/assets/25b572fb-80ce-4ebd-899c-5ab643a32097" />

In that stream, the server version `2023.11.3` was clearly visible, so the answer did not require deeper investigation.

<img width="1152" height="201" alt="immagine" src="https://github.com/user-attachments/assets/4a09146f-c2e8-4c0a-9b96-852276881683" />

### Answer: 2023.11.3
### Q4 - The attacker exploited the vulnerability to create a user account. What credentials did he set up?

Immediately below, the credentials requested in the question are also visible.

<img width="527" height="67" alt="immagine" src="https://github.com/user-attachments/assets/387a3738-3381-4242-8425-b6aca6160862" />

> [!NOTE]
> The answer format was also underspecified here. The lab expected the credentials in the username:password form, with a colon between the two values, but this was not clearly stated in the question itself.
> This made the task partly dependent on guessing the expected input format rather than only identifying the correct data.

### Answer: c91oyemw:CL5vzdwLuK
### Q5 - The attacker uploaded a webshell to ensure his access to the system. What is the name of the file that the attacker uploaded?

The same stream also made this question easy to answer, because the uploaded file name `NSt8bHTg.zip` was visible directly in the multipart upload content.

<img width="801" height="101" alt="immagine" src="https://github.com/user-attachments/assets/8732f2f8-072d-4961-a104-29c871ac41da" />

### Answer: NSt8bHTg.zip
> [!NOTE]
>This again shows how much the first question weakens the rest of the lab: once the attacker IP is known, a simple source-IP filter plus Follow HTTP Stream reveals multiple answers with very little additional analysis.

> [!CAUTION]
> Some of the following questions demand knowledge that the lab does not meaningfully train beforehand. Failing to answer them independently is therefore normal.
> In practice, searching the observed details online can quickly lead to public solutions, which makes this approach weak from a learning perspective and exposes a clear limitation of this kind of lab.

### Q3 - After identifying the version of our web server service, what CVE number corresponds to the vulnerability the attacker exploited?

I saw that the version was 2023.11.3, and I also saw that the attacker was able to perform unauthorized actions and create a new admin account.
I then searched those details on Google, and the answer appeared almost immediately. 

<img width="653" height="283" alt="immagine" src="https://github.com/user-attachments/assets/db3bd025-45e9-4fc1-8b99-49f69caac612" />

### Answer: CVE-2024-27198
> [!NOTE]
>This was not a very realistic way to reach the result, but that is how this lab worked in practice.

### Q8 - What is the MITRE Technique ID for the attacker's action in the previous question (Q7) when tampering with the text file?

see above, almost the same

<img width="649" height="272" alt="immagine" src="https://github.com/user-attachments/assets/82696e30-b1e5-49df-bd4d-e1048c1862f4" />

### Answer: T1565.001
