# Poisoned Credentials – PCAP Analysis (CyberDefenders)

## Scenario
Your organization's security team has detected a surge in suspicious network activity. 
There are concerns that LLMNR (Link-Local Multicast Name Resolution) and NBT-NS (NetBIOS Name Service) poisoning attacks may be occurring within your network.
These attacks are known for exploiting these protocols to intercept network traffic and potentially compromise user credentials. 
Your task is to investigate the network logs and examine captured network traffic.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/poisonedcredentials/

> [!IMPORTANT]
> The questions below are not in the original lab order. I arranged them in the order that best matched my investigation flow.

### Q1 - In the context of the incident described in the scenario, the attacker initiated their actions by taking advantage of benign network traffic from legitimate machines. Can you identify the specific mistyped query made by the machine with the IP address 192.168.232.162?

For this question, I immediately filtered on the source IP with `ip.src == 192.168.232.162` and scrolled through the results.
Since the visible traffic was already very limited, there was no need to apply any additional filters. By checking the Info column, I could see that the mistyped query was FILESHAARE.

<img width="1444" height="104" alt="immagine" src="https://github.com/user-attachments/assets/6ab61ec4-43b6-4c10-9801-61aa05dca8e4" />

**Answer:** `FILESHAARE`

### Q2 - We are investigating a network security incident. To conduct a thorough investigation, We need to determine the IP address of the rogue machine. What is the IP address of the machine acting as the rogue entity?

<img width="1576" height="416" alt="immagine" src="https://github.com/user-attachments/assets/d1866b14-3e04-43b3-bed1-c25e99d2227f" />

I started by filtering the traffic so that I could focus only on the communication relevant to this part of the incident, and then I reviewed the packets in chronological order. 
My goal here was not just to look for the answer directly, but to observe how the communication was unfolding between the machines.
By doing that, I could see that `192.168.232.162` first sent the mistyped NBNS query, and that` 192.168.232.215` replied to it. 
In the context of the lab, that response pattern was enough to identify `192.168.232.215` as the rogue machine, because it was the host taking advantage of the benign name-resolution traffic and sending the poisoned reply.

**Answer:** `192.168.232.215`

### Q3 -As part of our investigation, identifying all affected machines is essential. What is the IP address of the second machine that received poisoned responses from the rogue machine?

Later in the same sequence, I observed that `192.168.232.176` also sent `NBNS` queries and received responses from `192.168.232.215`. 
That made it possible to identify `192.168.232.176` as the second machine that received poisoned responses from the rogue host.
In this case, directly observing the communication pattern was more useful than adding extra filters, because the sequence itself already showed who was asking, who was answering, and which machine was acting maliciously.

**Answer:** `192.168.232.176`

>[!NOTE]
This conclusions were possible mainly because the lab is simple and the relevant traffic is short and easy to isolate. 
In a more realistic investigation, stronger confirmation would usually require broader context and additional validation.
>

### Q4 - We suspect that user accounts may have been compromised. To assess this, we must determine the username associated with the compromised account. What is the username of the account that the attacker compromised?

For this question, I filtered the traffic to `SMB2` because I expected the compromised username to appear in the authentication exchanges.
After isolating the `SMB2` traffic, I reviewed the packets and found an `NTLMSSP AUTH` entry in the Info column showing `cybercactus.local\janesmith`.
That was enough to identify the compromised username as `janesmith`.

<img width="1950" height="108" alt="immagine" src="https://github.com/user-attachments/assets/707499e8-275c-415d-bc91-0f3b6782f396" />

**Answer:** `janesmith`

### Q5 - As part of our investigation, we aim to understand the extent of the attacker's activities. What is the hostname of the machine that the attacker accessed via SMB?

For this question, I reused the same `SMB2` authentication sequence from the previous step.
I only had to move one packet above, because the `NTLMSSP_AUTH` packet was useful for the username, while the preceding `NTLMSSP_CHALLENGE` packet was the one carrying the target host information. 
In that response, the `Target Info` section exposed the machine identity, and the hostname was visible directly under NetBIOS computer name as `ACCOUNTINGPC`.

<img width="2535" height="538" alt="immagine" src="https://github.com/user-attachments/assets/e5a274e4-a596-4d3c-9c0f-187af3146ff2" />

**Answer:** `AccountingPC`
