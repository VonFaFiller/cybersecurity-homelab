# T1595 – PCAP Analysis (CyberDefenders)


## Scenario
Adversaries may execute active reconnaissance scans to gather information that can be used during targeting.
In these scans, the adversary probes the victim infrastructure via network traffic, as opposed to other forms of reconnaissance that do not involve direct interaction.

Adversaries may perform different forms of active scanning depending on what information they seek to gather. 
These scans can also be performed in various ways, including using native features of network protocols such as ICMP. 
Information from these scans may reveal opportunities for other forms of reconnaissance (ex: Search Open Websites/Domains or Search Open Technical Databases),
establishing operational resources (ex: Develop Capabilities or Obtain Capabilities), and/or initial access (ex: External Remote Services or Exploit Public-Facing Application).

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/t1595/

### Q1 - What is the Zero-tier network ID?

For this question, I went to `Statistics → Capture File Properties` and checked the **Interfaces** section. In that window, the `ZeroTier One` entry already showed the network ID in brackets, which corresponded directly to the answer: `544aaeed1673158e`.

<img width="241" height="144" alt="immagine" src="https://github.com/user-attachments/assets/69c0aaa5-91eb-49fd-b25f-145be0945a7f" />

> [!NOTE]
>ZeroTier is a virtual network service that lets devices in different places join the same logical network, almost like a private LAN over the internet.

**Answer:** `544aaeed1673158e`

### Q2 - What is the size of ARP packets in bytes?

I then opened `Protocol Hierarchy` from the `Statistics` menu.
For readability in the screenshot, I moved the `Bytes` column further to the left, then read the value directly from the relevant protocol entry.

<img width="441" height="96" alt="immagine" src="https://github.com/user-attachments/assets/4635addf-0cd0-4aac-8609-8bec3f6f0478" />

> [!NOTE]
> `Address Resolution Protocol` is commonly abbreviated as `ARP`.

**Answer:** `9184`

### Q3 - What is the address that sent the most packets?

For this question, I first sorted the conversations by packet count from highest to lowest in `Statistics → Conversations`, then moved the packet-direction columns so the flow would be easier to read.

<img width="505" height="74" alt="immagine" src="https://github.com/user-attachments/assets/12dbe154-a840-420a-9f15-ab698413929f" />

I selected `243.208.197.200` because that conversation clearly stood out: `Packets A → B` was `0`, while `Packets B → A` was `821`.
That showed that all the packets in that exchange were being sent from `243.208.197.200`, which made it the address that had sent the most packets in the context of the lab.

**Answer:** `243.208.197.200`

### Q4 - What is the City of the IP is connected to in the Philippines?

Again, I stayed within the `Statistics` menu but opened `Endpoints` this time.
As usual, I moved the columns around for better readability in the screenshot, then sorted the entries by `Country`, located `Philippines`, and read the corresponding value in the `City` column directly.

<img width="320" height="105" alt="immagine" src="https://github.com/user-attachments/assets/e3818e30-2df5-4150-b16a-caf180fb49b1" />

**Answer:** `La Trinidad`

### Q5 - How many DHCP Discover messages are in the PCAPNG file?

For this question, I filtered the traffic to `dhcp`, and the total number of matching packets was shown directly at the bottom of the window, where the red box is highlighted.

<img width="626" height="509" alt="immagine" src="https://github.com/user-attachments/assets/527f049b-2f8f-4dc3-9410-fbb52f8564cd" />

**Answer:** `27`

### Q6 - What is the "Target MAC address" for packet 37?

For this question, the wording was specific enough that I could use the filter `frame.number == 37` to jump directly to the packet being referenced. 
Once that packet was isolated, the target MAC address was immediately visible because MAC addresses are part of the Ethernet layer and are shown directly in the packet summary as source and destination hardware addresses.

<img width="595" height="74" alt="immagine" src="https://github.com/user-attachments/assets/1e6ca511-44ca-4fe2-8246-2acc13831ec4" />


**Answer:** `0e:9d:89:55:b2:02`

### Q7 - How many ARP reply packets are present in the PCAPNG file?

I used the same basic approach as in the previous DHCP question.
I filtered the traffic to `arp`, and the total number of matching packets was shown directly at the bottom of the window.

<img width="521" height="469" alt="immagine" src="https://github.com/user-attachments/assets/8028f390-87e8-4375-be33-526b4f1e02e0" />

**Answer:** `164`

### Q8 - What is the time packet 55 sent?

For this question, I first changed the time display format from `View → Time Display Format` so the timestamp would match the format required by the lab. 
I then used the filter `frame.number == 55` to jump directly to the referenced packet.
Once that packet was isolated, the timestamp was visible directly in the `Time` column, which gave the answer.

<img width="311" height="73" alt="immagine" src="https://github.com/user-attachments/assets/0bfbedab-b255-4111-aeb9-ec820788fee0" />

**Answer:** `2022-06-25 04:29`

### Q9 - What is the range of the targeted network IP addresses?

I checked the relevant IPs in `Statistics → Endpoints`. 
All the visible target addresses belonged to the `192.168.196.x` range, and the presence of `192.168.196.255` also pointed to the broadcast address of that subnet.
In the context of the lab, that was enough to identify the targeted network range as `192.168.196.0/24`.

<img width="127" height="96" alt="immagine" src="https://github.com/user-attachments/assets/0053766a-df0c-4093-a28d-4f965994ed68" />

**Answer:** `192.168.196.0/24`

### Q10 - What are the port numbers targeted by the attacker?

For this question, I first went to `Statistics → Conversations` and reviewed the TCP conversations, because that view makes the flow between source and destination addresses much easier to interpret than reading single packets one by one. 
I then focused on the destination ports, since those are the ports that indicate which services on the target side were being contacted, while the source ports are mostly temporary client-side ports.

<img width="1160" height="954" alt="immagine" src="https://github.com/user-attachments/assets/94d19488-1069-428e-85a1-53ca06eb942d" />

In the screenshot, the repeated conversations from the external host toward the internal `192.168.196.x` range consistently showed destination port `445`, and additional conversations also showed destination port `1433`.
Since those were the service ports repeatedly contacted on the target machines, I inferred that `445` and `1433` were the ports targeted by the attacker.
Puoi aggiungere così:

From the same view, it was also possible to infer that `185.245.85.178` was the main suspicious source, because those two destination ports were being stressed specifically by that IP across multiple conversations. 

> [!NOTE]
>In a more realistic investigation, this would still need stronger validation before treating it as a firm attribution, but in the context of this lab it was the most obvious attacker candidate.

**Answer:** `445,1433`

### Q11 - What is the country where the attacker is located?

I returned to `Statistics → Endpoints`, and there was no need to search the IP online.
The country information was already visible directly in that view.

<img width="323" height="70" alt="immagine" src="https://github.com/user-attachments/assets/34e382cb-4b76-4fb4-8fe8-ca5b506f8daa" />

**Answer:** `Slovakia`

### Q12 - What is the name of the Threat Actor with which this technique is associated?

> [!CAUTION]
> This question required broader knowledge that goes beyond pure practical analysis.
> A simple search was not enough here, because the objective was not entirely clear and more than one MITRE path could have fit the observed activity.
> Since there was no direct attack in the PCAP, the exact sub-technique was less obvious. I also avoided searching the lab name directly, because that would have turned the exercise into a shortcut rather than a real learning step.


I went to the MITRE ATT&CK website at `attack.mitre.org` and started from the `Reconnaissance` tactic.
From there, I opened `Active Scanning`, then the `Vulnerability Scanning` sub-technique, because that path best matched the activity observed in the PCAP.

<img width="224" height="516" alt="immagine" src="https://github.com/user-attachments/assets/c3416448-073d-4de2-8a96-36f4ffd41bd4" />

<img width="1872" height="694" alt="immagine" src="https://github.com/user-attachments/assets/cfc9cf26-0597-4857-b9d6-d15f36807565" />

<img width="1258" height="513" alt="immagine" src="https://github.com/user-attachments/assets/8539a991-9adf-458b-8263-77ee9c6af9eb" />

<img width="1260" height="498" alt="immagine" src="https://github.com/user-attachments/assets/264ec5fa-a59b-4385-bfc0-4eba5b918ba6" />

In the `Procedure Examples` section, `APT28` was explicitly listed. 
Given my limited experience with this kind of MITRE correlation, I used it as a quick confirmation step and did not go much further than that.
I chose this path because it seemed to be the closest match to the behavior observed in the exercise.

**Answer:** `APT28`
