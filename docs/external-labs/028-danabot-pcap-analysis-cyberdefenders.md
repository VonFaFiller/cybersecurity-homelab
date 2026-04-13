# DanaBot – PCAP Analysis (CyberDefenders)


## Scenario
The SOC team has detected suspicious activity in the network traffic, revealing that a machine has been compromised.
Sensitive company information has been stolen.
Your task is to use Network Capture (PCAP) files and Threat Intelligence to investigate the incident and determine how the breach occurred.


## References
- https://cyberdefenders.org/blueteam-ctf-challenges/danabot/

### Q1 - Which IP address was used by the attacker during the initial access?

I first went to **Statistics → Conversations** to get an initial sense of who was talking to whom, and from there I reasonably assumed that the **`10.x.x.x`** address was the internal host. 
After that, I isolated the **HTTP** traffic and started looking at the requests that already looked a bit suspicious just from the **Info** column.
One of them stood out immediately: a **`GET /login.php`** request to **`62.173.142.148`**.



<img width="244" height="155" alt="immagine" src="https://github.com/user-attachments/assets/4442d06d-766e-477b-aa8a-b65ae029ee53" />

From there I opened **Follow TCP Stream**, and that is where it became clearly abnormal.
A file called **`login.php`** being requested with **GET** is not strange by itself, but the response is what makes it suspicious: instead of returning a normal HTML login page, the server replies with **`200 OK`**, **`Content-Type: application/octet-stream`**, a **`Content-Disposition: attachment`** header with a **`.js`** filename, and then a large block of heavily obfuscated JavaScript.
That is not what a legitimate login page should look like. 
At that point, the stream is already enough to make the traffic look malicious rather than normal web browsing.

<img width="1422" height="286" alt="immagine" src="https://github.com/user-attachments/assets/62d1bb4f-1dae-4a87-98cc-459cbebd270c" />

Considering that response, and also the surrounding HTTP activity right after it, the hypothesis becomes much stronger that **`62.173.142.148`** is the IP the question is pointing to.
In other words, I did not rely on **Conversations** alone, but used it only to orient myself first, then confirmed the suspicion through the actual HTTP exchange that appears to deliver the malicious content during the initial access stage.

<img width="1323" height="775" alt="immagine" src="https://github.com/user-attachments/assets/01ad033e-b0b9-4efa-9ab3-79e393aa9594" />


**Answer:** `62.173.142.148`

### Q2 - What is the name of the malicious file used for initial access?

To answer this question, it is enough to look at the **filename** shown in the previous **Follow HTTP Stream** (screenshot above).
<img width="547" height="207" alt="immagine" src="https://github.com/user-attachments/assets/0ff3a903-b686-4927-8b42-fb5ec46e1575" />

**Answer:** `allegato_708.js`

### Q3 - What is the SHA-256 hash of the malicious file used for initial access?

For this question, I had to go one step further, because the **SHA-256 hash** is not something you just read directly from the packet details. 
Starting from the earlier HTTP activity, I went to **File → Export Objects → HTTP** and looked for the object that matched the malicious file used for the initial access.
In the export list, the relevant entry stands out clearly as **`login.php`** with **`application/octet-stream`**, which matches what had already looked suspicious in the HTTP stream.

<img width="452" height="543" alt="immagine" src="https://github.com/user-attachments/assets/9ae18151-b116-4e14-bd3f-8a861954b776" />

Once exported, I calculated its hash outside Wireshark using PowerShell with:

`Get-FileHash .\login.php -Algorithm SHA256`

That gave the final SHA-256 value directly.

<img width="746" height="87" alt="immagine" src="https://github.com/user-attachments/assets/9a4a2267-8f91-4d05-9d52-8d14198713a3" />
<img width="1065" height="317" alt="immagine" src="https://github.com/user-attachments/assets/237970d4-3956-40c2-91a0-182c6e53158c" />

> [!NOTE]
> In **Export Objects → HTTP**, the relevant object may appear under the requested path, such as **`login.php`**, rather than under the attachment filename shown in the HTTP response.
> In this case, the malicious file delivered during the initial access was identified from the **`Content-Disposition`** header as **`allegato_708.js`**, but the exported object was still saved from the HTTP transaction associated with **`login.php`**.
> So the hash was calculated on the downloaded malicious content itself, not on the requested path name.

**Answer:** `847B4AD90B1DABA2D9117A8E05776F3F902DDA593FB1252289538ACF476C4268`

### Q4 - Which process was used to execute the malicious file?

For this one, I deduced it by simply searching for script inside the malicious content, as shown in the screenshot. 

<img width="686" height="454" alt="immagine" src="https://github.com/user-attachments/assets/08325a23-50f2-4680-b646-fb15fc9918d9" />

That immediately exposed the reference to WScript, which is a strong enough indicator that the JavaScript file was being executed through the Windows Script Host, meaning the relevant process was wscript.exe.

**Answer:** `wscript.exe`

### Q5 - What is the file extension of the second malicious file utilized by the attacker?

Staying on the **`http`** filter, near the traffic that follows the initial malicious JavaScript, another suspicious request stands out: **`GET /resources.dll`**. That already strongly suggests the second malicious file, since the filename is visible directly in the **Info** column.

<img width="1604" height="282" alt="immagine" src="https://github.com/user-attachments/assets/e78d3cb7-863f-4685-b12b-22a53b125b7c" />

To avoid assuming too much, I then followed the HTTP stream for that request.
The returned content is clearly not normal text or a web page: it begins with the typical **`MZ`** header and even shows **“This program cannot be run in DOS mode”**, which is a classic sign of a Windows PE executable.
So, between the requested filename **`resources.dll`** and the binary executable content shown in the stream, the file extension asked by the question is clearly **`.dll`**.

<img width="1140" height="951" alt="immagine" src="https://github.com/user-attachments/assets/dcd53046-375b-447c-b041-3e2b5b73f1ba" />


**Answer:** `.dll`

### Q6 - What is the MD5 hash of the second malicious file?

After identifying **`resources.dll`** as the second malicious file, I exported it and calculated its hash outside Wireshark.
Since the question specifically asked for the **MD5** hash, I used PowerShell with:

`Get-FileHash .\resources.dll -Algorithm MD5`

<img width="527" height="146" alt="immagine" src="https://github.com/user-attachments/assets/322dff10-cdf4-48c8-8a22-58d7e2206e32" />
<img width="1047" height="267" alt="immagine" src="https://github.com/user-attachments/assets/a5800b60-6539-4d87-af0f-a33238c92a00" />

**Answer:** `E758E07113016ACA55D9EDA2B0FFEEBE`
