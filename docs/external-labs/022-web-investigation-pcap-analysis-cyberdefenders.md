# Web Investigation - (CyberDefenders)

## Scenario
You are a cybersecurity analyst working in the Security Operations Center (SOC) of BookWorld, an expansive online bookstore renowned for its vast selection of literature. 
BookWorld prides itself on providing a seamless and secure shopping experience for book enthusiasts around the globe.
Recently, you've been tasked with reinforcing the company's cybersecurity posture, monitoring network traffic, and ensuring that the digital environment remains safe from threats.
Late one evening, an automated alert is triggered by an unusual spike in database queries and server resource usage, indicating potential malicious activity. 
This anomaly raises concerns about the integrity of BookWorld's customer data and internal systems, prompting an immediate and thorough investigation.
As the lead analyst in this case, you are required to analyze the network traffic to uncover the nature of the suspicious activity.
Your objectives include identifying the attack vector, assessing the scope of any potential data breach, and determining if the attacker gained further access to BookWorld's internal systems.

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/web-investigation/

### Q1 - By knowing the attacker's IP, we can analyze all logs and actions related to that IP and determine the extent of the attack, the duration of the attack, and the techniques used. Can you provide the attacker's IP?

For the first question, I started with `Statistics → Conversations` to get a quick idea of which hosts were involved.
Given how small the dataset was, it was already possible to form an initial working assumption: `73.124.22.98` looked like the server side, while `111.224.250.131` stood out as the most suspicious external source because it had sent by far the highest number of packets.

<img width="240" height="106" alt="immagine" src="https://github.com/user-attachments/assets/391a0427-82fb-4334-8b05-b8434d208e21" />

That alone would not be enough in a realistic investigation, especially not to call it the attacker immediately.
However, after filtering the traffic to `http` and scrolling only a little further, the request pattern already started to support that suspicion.
The HTTP `Info` column was showing requests consistent with probable injection activity, which made `111.224.250.131` the obvious candidate to keep following and later confirm through the rest of the analysis.

<img width="1054" height="49" alt="immagine" src="https://github.com/user-attachments/assets/c7fc5a5c-67f9-4135-a238-0f84d1704d9e" />

<img width="1577" height="225" alt="immagine" src="https://github.com/user-attachments/assets/387dacee-a6b4-4a63-b990-f021090a4c90" />

So, somewhat reluctantly, I treated `111.224.250.131` as the attacker IP at this stage. 
In a real case, that kind of attribution would normally be confirmed much later, not at the beginning.
In this lab, however, the dataset was so small and direct that the suspicion could be reinforced almost immediately.

**Answer:** `111.224.250.131`

### Q2 - If the geographical origin of an IP address is known to be from a region that has no business or expected traffic with our network, this can be an indicator of a targeted attack. Can you determine the origin city of the attacker?

For the next question, I used `db-ip.com` to check the geolocation of the IP that had already emerged as the main suspicious source in the previous step. 
Since the lab was asking specifically for the origin city, an external IP geolocation lookup was the most direct way to answer it. 
Using `db-ip.com`, the address `111.224.250.131` was shown as being located in `Shijiazhuang`, which gave the answer.

<img width="797" height="178" alt="immagine" src="https://github.com/user-attachments/assets/9bd101d6-20f1-4324-94e8-5ff71d2b202e" />

**Answer:** `Shijiazhuang`

### Q3 - Identifying the exploited script allows security teams to understand exactly which vulnerability was used in the attack. This knowledge is critical for finding the appropriate patch or workaround to close the security gap and prevent future exploitation. Can you provide the vulnerable PHP script name?

Using the filter `ip.src == 111.224.250.131 && http` and reviewing the packets in chronological order **(the time display can be adjusted from `View → Time Display Format`)**, it was possible to see how the activity escalated over time. The attacker was clearly abusing `search.php`, and the suspected injection pattern began there. What started as normal-looking search requests quickly turned into increasingly suspicious payloads, which made `search.php` the obvious vulnerable script to focus on.

<img width="1718" height="504" alt="immagine" src="https://github.com/user-attachments/assets/b1cd1ed3-a1ea-4826-bce0-9b8869b17a5f" />

**Answer:** `search.php`

### Q4 - Establishing the timeline of an attack, starting from the initial exploitation attempt, what is the complete request URI of the first SQLi attempt by the attacker?

For this question, two things had to be understood first.
The first was what an injection attempt actually looks like: instead of sending a normal search value, the attacker starts inserting operators and conditions such as quotes, comments, or logical comparisons in order to alter how the backend query is interpreted.
The second was that the unusual symbols visible in the `Info` column were not random noise. 
They were URL-encoded characters, which is common when special characters are sent inside HTTP requests and need to be transmitted safely.

Since the traffic had already been arranged in chronological order in the previous step, the next task was simply to identify the first request where that pattern clearly appeared.
Once that URI was isolated, the most direct way to make it readable was to paste it into a basic URL decoder.
After decoding it, the result matched the exact request URI expected by the exercise, which confirmed that this was the first SQL injection attempt rather than just another normal search request.

<img width="1690" height="315" alt="immagine" src="https://github.com/user-attachments/assets/4a41308b-b450-410a-b1f0-ceeebbaa1a4f" />
<img width="675" height="266" alt="immagine" src="https://github.com/user-attachments/assets/5dc8f5eb-3557-4e02-8ab4-c9e2cc65f730" />

**Answer:** `/search.php?search=book and 1=1; -- -`

### Q5 - Can you provide the complete request URI that was used to read the web server's available databases?

For this question, I literally scrolled through the requests, watching the `Info` column and looking for values related to databases, since by that point it was already clear that the attacker was exploiting a SQL injection. 
Additional filtering or correlation might have been needed, but here this slower approach was enough and preserved the overall timeline.
From there, the more reliable step was to keep following the requests in chronological order and identify the point where the attacker moved from simple probing to actual database enumeration.
The decisive clue was the request targeting `INFORMATION_SCHEMA.SCHEMATA` together with `SCHEMA_NAME`, because that is specifically used to list the available databases in MySQL.
Once that URI was decoded, it could be treated as the definitive request used to read the web server’s available databases.

<img width="2487" height="158" alt="immagine" src="https://github.com/user-attachments/assets/7f5d9e36-efc2-49cd-8137-bea19369f93e" />
<img width="2289" height="53" alt="immagine" src="https://github.com/user-attachments/assets/841aa279-8d84-46ce-8ded-46af2ef24020" />
<img width="712" height="55" alt="immagine" src="https://github.com/user-attachments/assets/0cdde936-82e4-4227-94d6-c0a1c9d2e585" />

**Answer:** `/search.php?search=book' UNION ALL SELECT NULL,CONCAT(0x71787a7a71,SCHEMA_NAME,0x7170787a71) FROM INFORMATION_SCHEMA.SCHEMATA-- -`

### Q6 - Assessing the impact of the breach and data access is crucial, including the potential harm to the organization's reputation. What's the table name containing the website users data?


I continued from the previous step instead of starting a new line of analysis. 
Once the request to `INFORMATION_SCHEMA` had already shown that the attacker was enumerating databases, the next logical goal was to understand which database actually contained sensitive application data.

By using **Follow HTTP Stream** on that earlier traffic, the server response exposed the available database names, including `bookworld_db`. At that point, `bookworld_db` became the most relevant lead, because the name strongly suggested an application database rather than a default system schema.




<img width="1007" height="252" alt="immagine" src="https://github.com/user-attachments/assets/fca2a214-8ce2-4b86-aaa4-cd3d5c59327d" />
<img width="948" height="126" alt="immagine" src="https://github.com/user-attachments/assets/4fe1c534-579f-473e-8780-b3c63b4a83ae" />

From there, I filtered the traffic with `frame contains "bookworld_db"` to isolate the requests where the attacker had moved beyond simple enumeration and started interacting with that specific database. 
I then followed those HTTP streams one at a time.
I chose that slower approach on purpose, because a database can contain many different parts, and I wanted to understand where the attacker was only exploring metadata and where he was actually pulling sensitive content. 
Reading the streams individually made that distinction much easier.

Following the streams this way eventually led to a request targeting `bookworld_db.customers`.
That was the important turning point, because it showed not just the database name, but also the specific table being queried.

<img width="1330" height="447" alt="immagine" src="https://github.com/user-attachments/assets/5a14bdbe-3146-4523-be50-91f8fbc6dc23" />

In the corresponding HTTP response, the extracted content included real-looking addresses, emails, names, and other user-related fields.
That made it clear that the sensitive website user data was coming from the `customers` table inside the `bookworld_db` database, which is why `customers` was the correct answer.

<img width="389" height="40" alt="immagine" src="https://github.com/user-attachments/assets/7e552df4-b1b3-4d2d-b90e-61a5dfe67b05" />

**Answer:** `customers`

### Q7 - The website directories hidden from the public could serve as an unauthorized access point or contain sensitive functionalities not intended for public access. Can you provide the name of the directory discovered by the attacker?

Returning to the previous filter (`ip.src == 111.224.250.131 && http`), a quick scroll toward the latest packets already reveals two key points. First, the attacker performed multiple login attempts, indicating a brute-force or credential testing phase. Second, several **POST** requests clearly stand out among the traffic.

It would have been possible to filter specifically for POST requests to isolate them immediately, but I chose not to do that. Instead, I kept the full chronological view because it provides better context on how the activity evolved, and in this case it was sufficient without additional filtering.

<img width="1436" height="232" alt="immagine" src="https://github.com/user-attachments/assets/edfe9d8a-c933-474b-b21e-1297bfc6ea16" />

From the sequence of requests, it becomes clear that `/admin` is the relevant directory. This can be inferred from the repeated access attempts to `/admin` and `/admin/login.php`, followed by multiple POST requests to the same endpoint. The pattern shows the attacker first discovering the directory and then attempting to authenticate, which is consistent with targeting an administrative panel.

**Answer:** `/admin/`

### Q8 - Knowing which credentials were used allows us to determine the extent of account compromise. What are the credentials used by the attacker for logging in?

At this point, the next step is to focus on the first successful POST request.

Even without immediately inspecting the server response, it is possible to infer which login attempt succeeded by looking at what happens right after. Following that POST request, the attacker gains access to `/admin/index.php` and proceeds with further actions inside the panel. This transition indicates that the previous POST was successful.

By opening that specific POST request and inspecting its details, the submitted form data becomes visible. In the `application/x-www-form-urlencoded` section, the credentials are clearly shown:

<img width="1462" height="162" alt="immagine" src="https://github.com/user-attachments/assets/47b10459-71f8-4b6a-8a5a-3c159c3eed09" />
<img width="609" height="75" alt="immagine" src="https://github.com/user-attachments/assets/df3c2f9e-c55b-4504-9cec-85b70f99e3a8" />

This confirms that these were the credentials used by the attacker to successfully authenticate to the admin panel.

**Answer:** `admin:admin123!`

### Q9 - We need to determine if the attacker gained further access or control of our web server. What's the name of the malicious script uploaded by the attacker?

The request to `/admin/uploads/NVri2vhp.php` is sufficient to identify the malicious script because it appears after successful authentication and access to the admin panel, and it targets the upload directory previously interacted with by the attacker. 
The `.php` file inside that directory indicates execution of an uploaded script rather than normal browsing activity.

<img width="1167" height="56" alt="immagine" src="https://github.com/user-attachments/assets/97204df6-7ba6-47fd-8e76-a22826822e87" />

**Answer:** `NVri2vhp.php`
