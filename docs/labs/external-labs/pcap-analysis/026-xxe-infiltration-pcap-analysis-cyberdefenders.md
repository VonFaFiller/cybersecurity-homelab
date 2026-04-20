# XXE Infiltration - PCAP Analysis (CyberDefenders)

## Scenario
An automated alert has detected unusual XML data being processed by the server, which suggests a potential XXE (XML External Entity) Injection attack. 
This raises concerns about the integrity of the company's customer data and internal systems, prompting an immediate investigation.
Analyze the provided PCAP file using the network analysis tools available to you. Your goal is to identify how the attacker gained access and what actions they took.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/xxe-infiltration/

> [!IMPORTANT]
> The questions below are not in the original lab order. I arranged them in the order that best matched my investigation flow.

### Q2 - What's the complete URI of the PHP script vulnerable to XXE Injection?

For the second question, I filtered on **`xml`**, and that already narrowed the traffic down enough to make the pattern obvious. 
In the screenshot, the repeated **HTTP/XML POST** requests all go to **`/review/upload.php`**, so there was no need to overcomplicate it.
Since the question was asking for the complete URI of the PHP script vulnerable to **XXE**, and the XML uploads were clearly being sent there, that was enough to identify **`/review/upload.php`** as the relevant script.

<img width="1129" height="180" alt="immagine" src="https://github.com/user-attachments/assets/7974915a-dc7a-4072-a263-11262d1ee0ce" />

**Answer:** `/review/upload.php`

### Q3 - What's the name of the first malicious XML file uploaded by the attacker?

I stayed on the **`xml`** traffic and opened one of the first upload requests to inspect it more closely. 
In the packet details, under the **MIME Multipart Media Encapsulation** section, the filename is shown directly in the **`Content-Disposition`** line.
The value highlighted there is **`TheGreatGatsby.xml`**, so that is the name of the first malicious XML file uploaded by the attacker.

<img width="1111" height="327" alt="immagine" src="https://github.com/user-attachments/assets/bfe105d7-0c20-47b1-8e85-7fee7be10ead" />

**Answer:** `TheGreatGatsby.xml`

### Q4 - What's the name of the web app configuration file the attacker read?

Still on the **`xml`** traffic, I opened the server response and in the XML section the injected entity is shown as **`SYSTEM "file:///var/www/html/config.php"`**. 
So there was no real need to go further, because the name of the web application configuration file is already visible there as **`config.php`**.

<img width="1179" height="466" alt="immagine" src="https://github.com/user-attachments/assets/7c099977-0e3f-4862-85ec-de77900af9ad" />

**Answer:** `config.php`

### Q5 - What is the password for the compromised database user?

The same packet was also enough for this question.
In the returned content shown above, the response includes the leaked configuration values, and among them you can directly read **`$db_pass = 'Winter2024';`**. 
So the password for the compromised database user can be answered straight from that same response.

**Answer:** `Winter2024`

### Q7 - Can you identify the name of the web shell that the attacker uploaded for remote code execution and persistence?

Switching back to the **`http`** filter and scrolling toward the end of the capture, below the earlier XML-related activity, the answer to **Q7** becomes fairly clear. 
At that stage, the traffic shifts from the XXE upload activity to repeated requests against **`/uploads/booking.php`**, with command parameters such as **`cmd=whoami`**, **`cmd=uname -a`**, and even file reads like **`cat /etc/passwd`** and **`cat /etc/shadow`**. 
That is already enough to show that this file is not just a normal uploaded PHP file, but the web shell the attacker is actively using for command execution. 
So from that point, the relevant filename can be read directly as **`booking.php`**.

<img width="1400" height="251" alt="immagine" src="https://github.com/user-attachments/assets/2fac0826-7e8f-448f-946d-58ebd2a2933e" />


**Answer:** `booking.php`

### Q6 - What is the timestamp of the attacker's initial connection to the MySQL server using the compromised credentials after the exposure?

By filtering on **`ip.src == 210.106.114.183 && mysql`**, I isolated only the MySQL traffic originating from the attacker. 
The remaining packets are all **MySQL Login Request** messages from **`210.106.114.183`** to **`50.239.151.185`**, which shows that the attacker was initiating authentication attempts against the victim’s MySQL server. 
For the question about the attacker’s initial connection after the credential exposure, the relevant timestamp is the later one, **`2024-05-31 12:08:49.165156`**, since that is the first visible MySQL login request after the credentials had already been exposed.

<img width="980" height="191" alt="immagine" src="https://github.com/user-attachments/assets/b794a524-d8f6-4c20-9ceb-7548e683a1d4" />

> [!NOTE]
> The database credentials should be considered exposed when they appear in the **server response**, not when the attacker first sends the malicious XML request.
>  In this case, the relevant moment is the response where **`config.php`** is returned and the value **`$db_pass = 'Winter2024';`** becomes visible.
> Only after that point does it make sense to identify the first MySQL connection made with the compromised credentials.

**Answer:** `2024-05-31 12:08`

### Q1 - Can you identify the highest-numbered port that is open on the victim's web server?

I left this question for the end because, out of all the ones covered today, this was the one I found less immediate.

To answer it, I did not try to guess the open ports from generic traffic. Instead, I focused on the part that actually matters during reconnaissance: the server’s replies to the attacker while the ports are being probed.

I used this filter:

`ip.src == 50.239.151.185 && ip.dst == 210.106.114.183 && tcp.flags.syn == 1 && tcp.flags.ack == 1`

The idea behind it is simple. In TCP, when a client wants to start a connection, it normally begins with a **SYN** packet. **SYN** stands for synchronization, and it is basically the packet used to initiate the connection. If the destination port is open, the server answers with **SYN, ACK**. The **ACK** stands for acknowledgment, meaning the server is acknowledging the client’s request and agreeing to continue the TCP handshake. So, very roughly:

* **SYN** = “I want to start a connection”
* **SYN, ACK** = “I received that, and this port is open, let’s continue”

That is why filtering for packets with both **SYN** and **ACK** is useful here: they are strong indicators that the attacker hit an open port on the victim server.

With that filter applied, the screenshot becomes much easier to read.
We are only seeing packets sent **from the victim server to the attacker**, and specifically only the ones that are **SYN, ACK** replies.
At that point, the important value is the **source port** on the server side, because that is the port on the victim that is replying as open.

<img width="1646" height="278" alt="immagine" src="https://github.com/user-attachments/assets/12d04efe-4444-46c3-9a78-c352a8864757" />

In the screenshot, only two relevant server ports stand out clearly: **80** and **3306**. 
Since the question asks for the **highest-numbered open port** discovered on the victim’s web server, I just compare those visible open ports and take the higher one.

**Answer:** `3306`
