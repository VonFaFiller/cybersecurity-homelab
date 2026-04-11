# PacketMaze - PCAP Analysis (CyberDefenders)

## Scenario
A company's internal server has been flagged for unusual network activity, with multiple outbound connections to an unknown external IP.
Initial analysis suggests possible data exfiltration.
Investigate the provided network logs to determine the source and method of compromise.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/packetmaze/

### Q1 - What is the FTP password?

I first went to **Statistics → Conversations** to get a quick idea of the hosts present in the dataset, and from there I reasonably assumed that **`192.168.1.x`** was the internal subnet in this PCAP. 
Then, in the second screenshot, I filtered on **`ftp`** and kept the packets in chronological order.
At that point it was enough to read the **Info** column: the client sends **`PASS AfricaCTF2021`**, and the packet right after that says **`230 Login successful`**. 
That implies the username and password used in that exchange were valid, so the FTP password can be answered directly from there.

<img width="196" height="389" alt="immagine" src="https://github.com/user-attachments/assets/9f247141-aa27-42fb-a9a0-64ba8ee3166b" />
<img width="1216" height="257" alt="immagine" src="https://github.com/user-attachments/assets/4f356df9-cc59-4ec9-afef-492a3c9710c4" />

**Answer:** `AfricaCTF2021`

### Q2 - What is the IPv6 address of the DNS server used by `192.168.1.26`?

For this question, it helps to be a bit clever about it. 
The lab is asking for the **IPv6 address of the DNS server** that the host is using. 
Briefly, **DNS** is the service that translates domain names into IP addresses.
So I filtered on **`dns`**, and in the screenshot I highlighted two very similar lines.

<img width="1755" height="390" alt="immagine" src="https://github.com/user-attachments/assets/2c32b6ec-cbef-40ec-9d62-03815d8578bf" />

The important point is not just that they look similar, but that they are clearly tied to the same DNS activity: same type of query, same requested hostname, and in this case even the same **transaction ID `0x6820`**.
From the IPv4 traffic above, we can already see that **`192.168.1.26`** is using **`192.168.1.10`** as its DNS server.
Then, by looking at the highlighted IPv6 line below, we get their IPv6 addresses as well, and the **destination** address is the one we need. 
So the IPv6 address of the DNS server used by the client is **`fe80::c80b:adff:feaa:1db7`**.

**Answer:** `fe80::c80b:adff:feaa:1db7`

### Q3 - What domain is the user looking up in packet `15174`?

I kept the same **DNS** filter and just scrolled until I reached the packet mentioned in the question.
Since the dataset is fairly small, there was no real need to apply any extra filtering. 
At that point, it was enough to open the packet details and look under **Queries**, where the requested domain name is shown directly. From there, you can read **`www.7-zip.org`**.

<img width="486" height="460" alt="immagine" src="https://github.com/user-attachments/assets/21353df6-9537-4314-952b-c0be866153a4" />

**Answer:** `www.7-zip.org`

### Q4 - How many UDP packets were sent from `192.168.1.26` to `24.39.217.246`?

For the next question, I stayed in **Conversations** and isolated the two IP addresses mentioned in the prompt. 
Once filtered to that pair, it was just a matter of counting the packets shown there: **1 + 9 = 10**, so the total number of UDP packets sent from **`192.168.1.26`** to **`24.39.217.246`** is **10**.

<img width="341" height="87" alt="immagine" src="https://github.com/user-attachments/assets/213979e6-ad07-4fad-8a23-92115ede36a6" />

**Answer:** `10`

### Q5 - What is the MAC address of the system under investigation in the PCAP file?

Going back to the earlier traffic where I was comparing the IPv4 and IPv6 activity, I just highlighted one of those packets, and I preferred using the first one. 
The **source IP** there is the suspicious host, and in the same frame, under **Ethernet II**, the **source MAC address** is shown as well.
So that packet is enough to read the MAC address of the system under investigation directly.

<img width="629" height="280" alt="immagine" src="https://github.com/user-attachments/assets/8ccc3fdc-3155-49af-b929-0c9b4dc812a5" />

**Answer:** `c8:09:a8:57:47:93`

### Q6 - What was the camera model name used to take picture `20210429_152157.jpg`?

Here I first tried to be a bit more precise by using a **`frame contains`** filter, but it did not work the way I wanted. 
The reason is simply that the useful string is not always visible as a complete readable value in the exact frame you expect, so instead of forcing a filter that was not helping, I just adapted.

<img width="1702" height="96" alt="immagine" src="https://github.com/user-attachments/assets/414a79fe-3820-4bb4-a23d-e83a0fa0e98b" />

I kept the **`ftp-data`** filter, because at that point the traffic was already clean enough and most of the relevant packets were clearly the ones carrying the **`STOR`** command for the requested JPEG. From there I manually opened one of the many packets related to that file upload and used **Follow TCP Stream**. 

<img width="1977" height="200" alt="immagine" src="https://github.com/user-attachments/assets/15d9a6c8-0a54-4584-b4d0-85268ec8306d" />
<img width="2009" height="132" alt="immagine" src="https://github.com/user-attachments/assets/a86f588c-fc26-4699-99a5-aacff572dcf8" />
<img width="1342" height="198" alt="immagine" src="https://github.com/user-attachments/assets/acdab722-fb8b-467c-953a-44d0cc943a3f" />

In that stream, the transferred content becomes visible, and that is where the camera model asked by the question can be read, since that kind of information can appear inside the image metadata.

**Answer:** `LM-Q725K`

### Q7 - What is the ephemeral public key provided by the server during the TLS handshake in the session with the session ID `da4a0000342e4b73459d7360b4bea971cc303ac18d29b99067e46d16cc07f4ff`?

This one was straightforward once I filtered on the session ID like this: `tls.handshake.session_id == da:4a:00:00:34:2e:4b:73:45:9d:73:60:b4:be:a9:71:cc:30:3a:c1:8d:29:b9:90:67:e4:6d:16:cc:07:f4:ff`.
That immediately narrowed it down to the relevant TLS handshake packet. 
From there, the **Info** column already shows that this is the server-side handshake packet containing **Server Hello**, **Certificate**, **Certificate Status**, and **Server Key Exchange**.
So I just expanded **Server Key Exchange**, then **EC Diffie-Hellman Server Params**, and the value is shown directly under **Pubkey**. 
That is the ephemeral public key requested by the question.

<img width="2134" height="404" alt="immagine" src="https://github.com/user-attachments/assets/27a9da7f-828e-44fc-b0e0-68bc3e50c36e" />

**Answer:** `04edcc123af7b13e90ce101a31c2f996f471a7c8f48a1b81d765085f54805`

### Q8 - What is the first TLS 1.3 client random that was used to establish a connection with `protonmail.com`?

For this question, I filtered with **`tls && frame contains "proton"`**, which was already enough to narrow the view down to the relevant **TLSv1.3 Client Hello** packets for **`protonmail.com`**. 
From there, the packet details make it straightforward: under **Transport Layer Security**, inside the **Handshake Protocol: Client Hello** section, the value shown under **Random** is exactly what the question is asking for.
So there was no need to overcomplicate it, since the answer is directly visible there.

<img width="1522" height="528" alt="immagine" src="https://github.com/user-attachments/assets/03f21479-f99b-4a0c-869e-4c8782404a6a" />

**Answer:** `24e92513b97a0348f733d16996929a79be21b0b1400cd7e2862a732ce7`

### Q9 - Which country is the manufacturer of the FTP server’s MAC address registered in?

I went back to the **`ftp`** filter, since that traffic already gave me the FTP server involved in the exchange. 
I highlighted one of the early FTP packets, and in the **Ethernet II** section the **destination MAC address** is shown as **`08:00:27:a6:1f:86`**, which is the FTP server’s MAC in that packet. From there, I just used a MAC address lookup on that value. 
The lookup result shows the vendor as **PCS Systemtechnik GmbH** and the registered country as **US**, so the answer is **United States**.

<img width="756" height="273" alt="immagine" src="https://github.com/user-attachments/assets/97acce89-022f-417f-8261-a30bcbd7e57f" />
<img width="941" height="1034" alt="immagine" src="https://github.com/user-attachments/assets/860de3bc-cb92-47fa-8797-b2d2f371fa3b" />

**Answer:** `United States`

### Q10 - What time was a non-standard folder created on the FTP server on the 20th of April?

I first filtered on **`ftp.request.command == "LIST"`** to isolate only the moments where the client was asking for a directory listing. 
That was useful because it immediately showed me the relevant **LIST** requests without having to scroll through all the other FTP control traffic.
At that stage, though, doing **Follow TCP Stream** there still mainly showed the FTP control channel, so I was mostly seeing commands and server replies such as **`LIST`**, **`150 Here comes the directory listing`**, **`226 Directory send OK`**, and the surrounding FTP activity.

<img width="1080" height="184" alt="immagine" src="https://github.com/user-attachments/assets/08402698-d31e-4f43-94aa-59120136bcac" />
<img width="394" height="946" alt="immagine" src="https://github.com/user-attachments/assets/627380b2-99a9-4bb6-883b-0e83e962319f" />

Once that picture was clear enough, I switched approach for convenience.
Since the control channel was too noisy and I already had a good idea of what commands the attacker had executed, I moved to the actual data channel by filtering the **FTP-DATA** traffic related to the directory listings.
In practice, I used the FTP-DATA side with the **LIST** traffic visible, which made the view much cleaner because at that point I was no longer looking at the command conversation itself, but at the transferred listing content. 
From there, I simply picked the **first relevant FTP-DATA packet** and did **Follow TCP Stream** on it. 

<img width="1352" height="253" alt="immagine" src="https://github.com/user-attachments/assets/a4dc1647-40d5-4e5a-b93f-6d52a0eaf06f" />
<img width="495" height="193" alt="immagine" src="https://github.com/user-attachments/assets/1e19cf6b-db40-4251-997e-2b5337f7eb32" />

Inside that stream, the unusual entry appears with the timestamp **`Apr 20 17:53`**, which is exactly the detail the question was asking for.
So the answer comes from following the correct **FTP-DATA LIST** stream rather than staying on the FTP control packets, because the control side tells you that a listing happened, while the data side actually shows the listing content and therefore the folder name and its time.

**Answer:** `17:53`

### Q11 - What URL was visited by the user and connected to the IP address `104.21.89.171`?

This one was possible mainly because the question was very specific. I filtered with **`ip.addr == 104.21.89.171 && http`**, which immediately reduced the view to the relevant HTTP exchange with that IP. 
From there, the traffic was easy to read: the client sends a **`GET /`** request, and the server replies with **`HTTP/1.1 301 Moved Permanently`**.
In the response details, the important part is the **`Location`** header, because that is where the visited URL is shown after the redirect.

<img width="1322" height="414" alt="immagine" src="https://github.com/user-attachments/assets/5e1ea159-7f55-4b74-b5b9-f88829fe60cd" />

> [!NOTE]
> The observed request to `104.21.89.171` was made over **HTTP** as `GET /`, while the server replied with a **301 redirect** to `https://dfir.science/`.
>  For this reason, the answer was kept as `http://dfir.science/`, since that is the URL actually requested in the visible exchange.

**Answer:** `http://dfir.science/`

