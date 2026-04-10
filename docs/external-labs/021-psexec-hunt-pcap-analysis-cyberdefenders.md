# PsExec Hunt - (CyberDefenders)

## Scenario
An alert from the Intrusion Detection System (IDS) flagged suspicious lateral movement activity involving PsExec.
This indicates potential unauthorized access and movement across the network. As a SOC Analyst, your task is to investigate the provided PCAP file to trace the attacker’s activities. 
Identify their entry point, the machines targeted, the extent of the breach, and any critical indicators that reveal their tactics and objectives within the compromised environment.

> [!IMPORTANT]
> The questions below are not in the original lab order. I arranged them in the order that best matched my investigation flow.

### Q1 - To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?

For the first question, I first tried to understand which side was most likely acting as the server. 
By checking `Statistics → Conversations`, the `10.x.x.x` hosts were the most obvious internal-side candidates in the context of the lab.

<img width="255" height="225" alt="immagine" src="https://github.com/user-attachments/assets/9483301c-5921-425b-a9df-8cd4d4197091" />

From there, I moved to the most sensible protocol to investigate further, which was `smb2`, and reviewed the packets in chronological order.

<img width="1482" height="516" alt="immagine" src="https://github.com/user-attachments/assets/b03d9fa1-1efd-4167-8d7a-13a91e782fe5" />

**Answer:** `10.0.0.???`

### Q4 - After figuring out how the attacker moved within our network, we need to know what they did on the target machine. What's the name of the service executable the attacker set up on the target?

The second screenshot already exposed several key answers almost directly through the `Info` column, without any real need to go deeper into packet details.
The presence of `PSEXESVC.exe` was already highly suspicious on its own, and the fact that it appeared after access to `IPC$` and `ADMIN$` made the PsExec-related interpretation much more plausible. 
In practice, that same sequence already gave strong candidates for Q4, Q5, and Q6, simply by reading the order in which the events happened.
Because the dataset was small and not very noisy, the flow was easy to follow directly from the packet list.

<img width="634" height="276" alt="immagine" src="https://github.com/user-attachments/assets/a10410b2-e2d2-4289-a193-1eaba4ca9874" />

Further confirmation came from the repeated appearance of `PSEXESVC` in the same direction of activity.
It would have been possible to keep drilling into more protocol details, but for the purposes of this lab that was not really necessary.

**Answer:** `psexesvc.exe`

### Q5 - We need to know how the attacker installed the service on the compromised machine to understand the attacker's lateral movement tactics. This can help identify other affected systems. Which network share was used by PsExec to install the service on the target machine?

**Answer:** `ADMIN$`

### Q6 - We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?

**Answer:** `IPC$`

### Q3 - Knowing the username of the account the attacker used for authentication will give us insights into the extent of the breach. What is the username utilized by the attacker for authentication?

That same reverse reconstruction also made it possible to answer Q3. 
Once the later SMB activity was clear, I could read slightly higher in the same sequence and confirm that the highlighted `NTLMSSP AUTH` line exposed the username `ssales`. 




**Answer:** `ssales`

### Q1 - To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?

At the same time, that same line strongly suggested that `10.0.0.130` was the attacker-side source in this first stage, because it was the host initiating the authentication and the later `IPC$` / `ADMIN$` activity that led into the PsExec deployment.


**Answer:** `10.0.0.130`


### Q2 - To fully understand the extent of the breach, can you determine the machine's hostname to which the attacker first pivoted?

For this question, the next useful step was to follow the specific SMB stream tied to that authentication line rather than the entire conversation **(remember that highlighted packet in the second screenshot?)**.
There was no need to review all the surrounding traffic again, since the goal here was only to isolate the relevant exchange and extract the hostname.
By following that stream, the host name became visible directly in the SMB/NTLM data, which allowed `Sales-PC` to be confirmed.

<img width="263" height="38" alt="immagine" src="https://github.com/user-attachments/assets/e3ddebd4-bf41-4d77-8c1c-c7752becfab9" />

<img width="739" height="86" alt="immagine" src="https://github.com/user-attachments/assets/c14a4a72-cca2-43ca-8f3a-06eec5ab69f1" />

**Answer:** `Sales-PC`


### Q7 - Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?

For the last question, I had more difficulty than with the previous ones, because the hostname was not exposed as immediately as some of the earlier answers. 
However, after isolating the `SMB` traffic, the target side of the second pivot became much easier to follow. 
From there, it was possible to correlate the SMB activity toward `10.0.0.131` with the visible `BROWSER` host announcements showing `MARKETING-PC`, which was enough in the context of the lab to identify `Marketing-PC` as the second machine targeted by the attacker.


<img width="1466" height="198" alt="immagine" src="https://github.com/user-attachments/assets/de6dd8d1-7aba-4587-9638-497b3f3901da" />

> [!NOTE]
> This could be treated as the second pivot target mainly because of **when** it appeared in the traffic.
>  By comparing the events in chronological order, the first compromise and PsExec-related activity from `10.0.0.130` were already visible earlier, while the SMB activity toward `10.0.0.131` appeared later as a new stage in the sequence.
> That timing made it reasonable to interpret `10.0.0.131` as the next pivot target. The remaining step was then just to map that IP to its hostname, which was visible in the `BROWSER` announcements as `MARKETING-PC`.

**Answer:** `Marketing-PC`



