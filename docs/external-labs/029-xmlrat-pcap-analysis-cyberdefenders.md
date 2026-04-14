 # XLMRat – PCAP Analysis (CyberDefenders)

## Scenario
A compromised machine has been flagged due to suspicious network traffic. 
Your task is to analyze the PCAP file to determine the attack method, identify any malicious payloads, and trace the timeline of events. 
Focus on how the attacker gained access, what tools or techniques were used, and how the malware operated post-compromise.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/xlmrat/

### Q1 - The attacker successfully executed a command to download the first stage of the malware. What is the URL from which the first malware stage was installed?

First I went to Conversations just to get a feel for the IPs in the dataset.

<img width="241" height="94" alt="immagine" src="https://github.com/user-attachments/assets/68f02a7e-f914-429c-9725-4bd5d71936eb" />

Then I filtered for HTTP since the dataset was small. 
From there I followed the HTTP stream for both GET requests. 
A quick look was enough to see they were not normal requests and were clearly part of the malicious activity, which was something to dig into more in the next questions.

<img width="1411" height="181" alt="immagine" src="https://github.com/user-attachments/assets/fb7e2e1f-5f27-43e1-9802-c7a3b4d901e1" />

For this one though, I just needed the exact URL.
So I opened the packet details shown in the screenshot below and took it directly from the **Full request URI** field.

<img width="765" height="441" alt="immagine" src="https://github.com/user-attachments/assets/f6c11761-888f-4b6b-b988-c08709521500" />

**Answer:** `http://45.126.209.4:222/mdm.jpg`

### Q2 - Which hosting provider owns the associated IP address?

I just searched on Google for a hosting provider lookup, opened one of the results, pasted the IP `45.126.209.4`, and checked what provider it resolved to. 

<img width="695" height="273" alt="immagine" src="https://github.com/user-attachments/assets/c2ffac78-83df-4f9a-ae61-dc4fecce8766" />

It came back as **ReliableSite.Net LLC**.

<img width="648" height="206" alt="immagine" src="https://github.com/user-attachments/assets/ab583e08-fbb4-4929-84ae-8d3abdbc9f04" />

**Answer:** `ReliableSite.Net`

### Q3 - By analyzing the malicious scripts, two payloads were identified: a loader and a secondary executable. What is the SHA256 of the malware executable?

First I went to **File → Export Objects → HTTP** in Wireshark and exported only `mdm.jpg`, because after looking at the available objects it was already clear that this was the right file to work on.
I had already checked the content enough to know it contained the malicious script, so there was no reason to add noise by exporting anything else.

<img width="421" height="126" alt="immagine" src="https://github.com/user-attachments/assets/5b5bd70e-e1dd-46a4-b3c2-406d96b864a0" />

Then I worked from the assumption that `$pe` was the loader and `$NKbb` was the secondary executable.
The variable names alone were not enough, but they were a decent starting lead.
What made it hold up was the execution flow later: `$pe` gets loaded as a .NET assembly, then its `Execute` method is called and `$NKbb` is passed into it together with `RegSvcs.exe`.
That is why I treated `$NKbb` as the actual malware payload to hash.

<img width="2055" height="290" alt="immagine" src="https://github.com/user-attachments/assets/a6283bad-0787-42bf-a008-c8ed2692be24" />

Then I loaded the exported file into PowerShell from the Desktop:

```powershell id="jwws8h"
$content = Get-Content .\mdm.jpg -Raw
```

This was just to load the whole exported object into a variable as raw text, because I needed to search inside the script content instead of reading it line by line.

Next I pulled out the value of `$hexString_bbb` with a regex.

```powershell id="lhdt5a"
$hexString_bbb = [regex]::Match($content, '(?s)\$hexString_bbb\s*=\s*["'']([^"'']+)["'']').Groups[1].Value
```

The point was to extract the long hex blob directly from the script without manually copying thousands of characters.

```powershell id="kxcv5t"
$hexString_bbb.Length
```

This was just a sanity check.
It returned `199679`, so I knew the extraction had actually worked and I was not dealing with an empty or broken match.

```powershell id="tf84tl"
[Byte[]]$NKbb = $hexString_bbb -split '_' | ForEach-Object { [byte]([Convert]::ToInt32($_,16)) }
```

This is the key step.
`$hexString_bbb` was not the executable yet, just a long text string of hex values separated by underscores.
So I split it on `_`, converted each hex chunk into a real byte, and rebuilt the payload as a byte array in memory.

```powershell id="6dv7q0"
[IO.File]::WriteAllBytes(".\NKbb.bin", $NKbb)
```
This was meant to save the reconstructed byte array as an actual binary file so I could hash the real payload, not the text representation.

> [!CAUTION]
> The next steps can be skipped, because in the following questions I found a more efficient way to recover the same kind of information.
> Still, I want to keep this older method here because, even if it was clearly less efficient, it was still a useful and fairly interesting learning path.


```powershell id="ij0vca"
(Get-FileHash ".\NKbb.bin" -Algorithm SHA256).Hash
```

That should have given me the SHA256 immediately, but this is where I hit the path issue.

The error was:

```text id="zev2ls"
Cannot find path 'C:\Users\Administrator\Desktop\NKbb.bin' because it does not exist.
```

So the problem was not with the extraction logic itself, but with the file path handling on the first write attempt.
I had used a relative path (`.\NKbb.bin`), and then `Get-FileHash` could not find the file where PowerShell was resolving it.
I also checked whether it had ended up in `C:\Windows\System32\NKbb.bin`, but that came back `False`, so I stopped guessing and just forced a full absolute path.

```powershell id="cwuy8v"
Test-Path C:\Windows\System32\NKbb.bin
```

That check was only there to see whether the file had been written somewhere else because of the relative path ambiguity.
It was not there.

So I redid the write with an explicit full path:

```powershell id="2f9o1q"
[IO.File]::WriteAllBytes("C:\Users\Administrator\Desktop\NKbb.bin", $NKbb)
(Get-FileHash "C:\Users\Administrator\Desktop\NKbb.bin" -Algorithm SHA256).Hash
```

<img width="1894" height="538" alt="immagine" src="https://github.com/user-attachments/assets/2e918e41-85c0-4066-ad78-26172e679fe1" />


**Answer:** `1EB7B02E18F67420F42B1D94E74F3B6289D92672A0FB1786C30C03D68E81D798`


### Q4 - What is the malware family label based on Alibaba?

After rebuilding the final payload and calculating its SHA256, I searched the hash on VirusTotal and looked specifically at the Alibaba detection name. 
From there I used the family name shown by Alibaba as the answer.

<img width="991" height="273" alt="immagine" src="https://github.com/user-attachments/assets/f7e501d5-ae8d-42a7-8c23-8194a3f280fc" />
<img width="2010" height="116" alt="immagine" src="https://github.com/user-attachments/assets/5495bbad-0619-43f2-b2ac-6efed2fb43d4" />

**Answer:** `AsyncRat`

### Q5 - What is the timestamp of the malware's creation?

At that point I was tired of digging around for more PowerShell commands, so I asked an AI to point me to some damn software that could handle this stuff more directly.
I installed **PE-bear**, opened the rebuilt payload there, and it made things much easier right away.
Instead of wasting more time on manual parsing, I could check the **SHA256** and the **PE timestamp** almost immediately from the file itself.
For this kind of task it was just faster and cleaner than memorizing more commands for basic PE metadata.

<img width="1672" height="977" alt="immagine" src="https://github.com/user-attachments/assets/5e24cb38-eafd-45b7-aa91-18c19507c43f" />
<img width="681" height="275" alt="immagine" src="https://github.com/user-attachments/assets/dc27bd7f-867a-4f2b-adf1-4c8f74e898d3" />
<img width="792" height="237" alt="immagine" src="https://github.com/user-attachments/assets/ce7c4ed5-f721-449e-bdf9-464794fdf4c2" />

**Answer:** `2023-10-30 15:08`

### Q6 - Which LOLBin is leveraged for stealthy process execution in this script? Provide the full path.

Since the exported `mdm.jpg` file was already sitting on the Desktop, I opened it in **Notepad++** just to have a cleaner and more readable view of the script than scrolling through it in raw form. 
From there it was easier to follow the relevant part of the execution flow and spot the path being built through obfuscated strings.
At that point it was basically enough to remove the `#` characters from the two string fragments and join them back together, which gives the full LOLBin path.

<img width="1960" height="190" alt="immagine" src="https://github.com/user-attachments/assets/a6974268-2c4d-4a46-a85b-2421ad62f0d5" />


**Answer:** `C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe`

### Q7 - The script is designed to drop several files. List the names of the files dropped by the script.

I first scrolled through the script normally to understand what it was actually creating on disk and what was just execution or persistence logic. 
At that point it was already pretty clear that the repeated `Conted` references were tied to real files, because I could see explicit `WriteAllText` calls writing them under `C:\Users\Public\`. 
After that, just to keep things cleaner and avoid missing one, I used **Find** in Notepad++ on `Conted`. 
That let me jump between the matches and confirm the different dropped files without adding extra noise.
From there I could separate the actual files from the later scheduled task logic and answer with the three dropped files:

<img width="1164" height="1250" alt="immagine" src="https://github.com/user-attachments/assets/53250439-b485-4a8e-954a-6b6bc997b8ef" />

**Answer:** ``Conted.ps1, Conted.bat, Conted.vbs``

