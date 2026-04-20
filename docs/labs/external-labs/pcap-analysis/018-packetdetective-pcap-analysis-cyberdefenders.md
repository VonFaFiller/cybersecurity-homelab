# PacketDetective – PCAP Analysis (CyberDefenders)

## Scenario
In September 2020, your SOC detected suspicious activity from a user device, flagged by unusual SMB protocol usage.
Initial analysis indicates a possible compromise of a privileged account and remote access tool usage by an attacker.

Your task is to examine network traffic in the provided PCAP files to identify key indicators of compromise (IOCs) and gain insights into the attacker’s methods, persistence tactics, and goals.
Construct a timeline to better understand the progression of the attack by addressing the following questions.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/packetdetective/

> [!IMPORTANT]
This lab is split across three separate files, and the questions are effectively distributed across them.
It is also worth noting that many of the answers are relatively easy to reach because the dataset is small and limited in scope.
In several cases, the relevant evidence can be identified directly without needing deeper reconstruction or broader correlation.
>


### Q1 - The attacker’s activity showed extensive SMB protocol usage, indicating a potential pattern of significant data transfer or file access. What is the total number of bytes of the SMB protocol?

For the first question, I went to `Statistics → Protocol Hierarchy` and read the total number of `bytes` directly from the `SMB (Server Message Block Protocol)` entry. That value corresponded directly to the answer.

<img width="946" height="117" alt="immagine" src="https://github.com/user-attachments/assets/933a5a5b-0f33-482d-b488-3452b5025ab4" />


**Answer:** `4406`

### Q2 - Authentication through SMB was a critical step in gaining access to the targeted system. Identifying the username used for this authentication will help determine if a privileged account was compromised. Which username was utilized for authentication via SMB?

Then I filtered the traffic to `SMB` and scrolled through the packets in chronological order to understand what was happening in the authentication sequence.
By reviewing the Info column, I could see a Session Setup AndX Request, `NTLMSSP AUTH` entry showing `User: \Administrator`. 
In the context of the lab, that was enough to identify the username used for `SMB` authentication as Administrator.

<img width="1482" height="112" alt="immagine" src="https://github.com/user-attachments/assets/fd56cc14-1e5e-4503-8124-af1b6ba60aee" />


**Answer:** `Administrator`

### Q3 - During the attack, the adversary accessed certain files. Identifying which files were accessed can reveal the attacker's intent. What is the name of the file that was opened by the attacker?

For this question, I kept reviewing the `smb` traffic in chronological order. In the **Info** column, one of the packets showed an `NT Create AndX Request` with `Path: \eventlog`, and the same value was also visible in the packet details under `File Name`. In the context of the lab, that was enough to identify `eventlog` as the name of the file opened by the attacker.

<img width="1427" height="113" alt="immagine" src="https://github.com/user-attachments/assets/f841b240-b495-4481-99fe-4a31d9154107" />

**Answer:** `eventlog`

### Q4 - Clearing event logs is a common tactic to hide malicious actions and evade detection. Pinpointing the timestamp of this action is essential for building a timeline of the attacker’s behavior. What is the timestamp of the attempt to clear the event log? (24-hour UTC format)

For this question, I first changed the time display format from `View → Time Display Format` so the timestamp would be easier to read in the format required by the lab.
I then filtered the traffic as shown in the screenshot and reviewed the `EVENTLOG` packets in chronological order.
From there, it was easy to identify the `ClearEventLogW request` and read the corresponding timestamp directly.

<img width="1136" height="115" alt="immagine" src="https://github.com/user-attachments/assets/b0f32973-5f52-4e08-8789-a4fd573f8f6a" />


> [!NOTE]
> This could have been filtered in more precise ways, but in this case there was no real need to do so.
>  The important point here is to recognize that the `EVENTLOG` protocol is present and that attackers may try to remove traces in different ways, including by clearing event logs through this type of request.


**Answer:** `2020-09-23 16:50`

### Q5 - The attacker used "named pipes" for communication, suggesting they may have utilized Remote Procedure Calls (RPC) for lateral movement across the network. RPC allows one program to request services from another remotely, which could grant the attacker unauthorized access or control. What is the name of the service that communicated using this named pipe?

For this question, I used the `isystemactivator` filter to isolate the traffic related to remote activation and object creation. 
I chose it because the question was asking about the service communicating through a named pipe, so it made sense to focus on the part of the RPC/DCOM exchange where that information was more likely to appear directly.
This step was not as immediate as some of the previous ones, because the answer was not exposed in a simple filename or username field.
Instead, it had to be identified inside the protocol details, where the named pipe appeared as `\PIPE\atsvc`.

<img width="917" height="718" alt="immagine" src="https://github.com/user-attachments/assets/d4cc52ff-3a32-4688-a4a3-7148b710fe91" />


> [!NOTE]
> A named pipe is a communication channel often used by Windows services and RPC mechanisms to exchange data locally or remotely.
> In this case, `atsvc` is the named pipe associated with the Windows Task Scheduler / AT service .
> It is used to create and manage scheduled tasks remotely, which makes it relevant for remote execution and persistence.
> (this is a much larger topic on its own)

<img width="319" height="121" alt="immagine" src="https://github.com/user-attachments/assets/2c5c23d0-2efd-420e-bf5d-d8a54fa0394c" />


**Answer:** `atsvc`

### Q6 - Measuring the duration of suspicious communication can reveal how long the attacker maintained unauthorized access, providing insights into the scope and persistence of the attack. What was the duration of communication between the identified addresses 172.16.66.1 and 172.16.66.36?

For this question, I went to `Statistics → Conversations` and moved the `Duration` column further to the left so it would be easier to read.
This step was straightforward because the question had already specified exactly which two IP addresses to focus on, so the only thing left was to read the corresponding conversation duration directly.

<img width="395" height="85" alt="immagine" src="https://github.com/user-attachments/assets/9fe01912-cefe-45c3-badc-a606296c7ef8" />

**Answer:** `11.7247`

### Q7 - The attacker used a non-standard username to set up requests, indicating an attempt to maintain covert access. Identifying this username is essential for understanding how persistence was established. Which username was used to set up these potentially suspicious requests?

<img width="1340" height="260" alt="immagine" src="https://github.com/user-attachments/assets/b4834be2-2574-4af5-a3af-3766953020e6" />

Opening the third file already made the answer visible almost immediately.
The `Info` column showed `User: 3B\backdoor` within the first packets of the SMB sequence.
I still reviewed the surrounding traffic to confirm it, but in the context of this easy lab the answer was already exposed very early.

**Answer:** `backdoor`

### Q8 - The attacker leveraged a specific executable file to execute processes remotely on the compromised system. Recognizing this file name can assist in pinpointing the tools used in the attack. What is the name of the executable file utilized to execute processes remotely? 

Within the first SMB packets, the `Info` column showed `Create Request File: PSEXESVC.exe`, which was enough to identify the executable used for remote execution. 
I then confirmed the context by looking at the nearby packets in the same SMB sequence, where the authentication, share access, and file creation activity all lined up with the same remote action.

**Answer:** `PSEXESVC.exe`
