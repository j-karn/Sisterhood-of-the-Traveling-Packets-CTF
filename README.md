# Sisterhood-of-the-Traveling-Packets-CTF
_Flare × SANS × WiCyS | 2026_
<p align="center">
<img width="20%" height="20%" alt="Digital_badge_image_1" src="https://github.com/user-attachments/assets/1e6dc5af-5aed-48be-b98c-021c6b9d83dd" />
</p>
Back in August, I participated in Sisterhood of the Traveling Packets CTF, organised by Flare, SANS, and WiCyS. The challenge centred around investigating a careless fictional ransomware group called Pantalones, uncovering its OPSEC mistakes and finding a way into its infrastructure. 
<br></br>

**1. Getting Started: Exploring the Evidence**

The investigation began when a .onion website was shared to us. The landing page shows a victim list with two sets of downloadable data from the group's alleged victims: QuantumCore Systems and AetherFlow Enterprise. I downloaded both the files. 

One particular file, exfil.sh, from the aetherflow_leak.zip, caught my attention. As I examining its contents, I came across a curl command:
`curl -s -X POST "${PANEL}/api.php?action=upload"`

This was my first indication that the website had an API endpoint at api.php. 

Curious to see what else is available, I checked the standard `robots.txt` file which showed two disallowed paths:
- api.php
- admin.php

These became the starting points for the next stage of my investigation.

**2. Exploring the Exposed Endpoints**

I navigated to both endpoints to understand what functionality was available. The `admin.php` endpoint presented a login page requiring a username and password. The `api.php` endpoint was more interesting. It exposed seven available actions: upload, status, messages, decrypt, wallets, payloads, and exfil. I started enumerating these actions to understand what information could be accessed.
- `upload`: returned an error: "authentication required"
- `status`: returned infrastructure details including uptime, nodes, and storage space. Members online: Vex and Crypt
- `messages`: an error: "missing required parameter: conversation_id"
- `decrypt`: an error: "missing required parameter: victim_id" along with expected id format.
- `wallets`: returned Bitcoin wallet details showing a total of 27.84 BTC received
- `payloads`: revealed cached payloads for three victims: StratifyTech, Lumenisys, and NexaVista. 
- `exfil`: returned details of three recently completed exfiltrations.

What stood out was that all seven endpoints were accessible without authentication. No login or authentication required before interacting with the available API actions. 

The messages action particularly caught my attention because it offered a way to explore conversations between members of the group. The error I encountered previously was a useful clue, it explicitly shared which parameter was expected.

After experimenting with the parameter, I discovered that using `conversation_id=0` returned a conversation. I then tried incrementing the value (1, 2, 3 and so on) to retrieve additional conversations.

**3. Following the Conversations**

The conversations revealed exchanges between the members of the group. As I worked through them, one between Mora and Crypt stood out: Mora was asking Crypt for her FTP server password.

Crypt then shared the password in a Base64 encoded format. I then copied the value into CyberChef and used its From Base64 operation to decode it.

The resulting password was:
`Pantal0n3s_Rul3z!`

Whilst care is shown here to conceal the actual text of the password, Base64 is simply an encoding method, not encryption. Anyone who could access the conversation would be able to decode it easily.

**4. Accessing the Admin Panel**

With the password recovered, I returned to the `admin.php` login page. I used Mora as the username and the decoded password to authenticate successfully.

Once inside the administrative dashboard, the flag was displayed:
`flare{pantal0n3s_g0t_pantsed_2026}`

Challenge completed!

**5. Mapping the Findings to MITRE ATT&CK**

The scenario highlights several OPSEC failures that can be mapped to MITRE ATT&CK techniques:

>**T1190 – Exploit Public-Facing Application:** All seven API actions accessible without authentication</br>
>**T1552.008 – Unsecured Credentials: Chat Messages:** Credentials shared between group members through internal conversations</br>
>**T1078 - Valid Accounts:** The use of legitimate account credentials to gain access to systems and resources, potentially bypassing other access controls.

**6. Key takeaways**

The challenge focused on identifying weaknesses from an offensive perspective, but it also highlighted several opportunities for organisations to strengthen their security practices:

>**Enforce access controls:** Require authentication and authorisation across all API endpoints. Apply least-privilege principles and validate access at the resource level to prevent unauthorised access to internal data.

>**Improve credential management:** Avoid sharing passwords through general messaging platforms. Use approved secrets management solutions, enforce unique passwords/passkeys and implement MFA.

>**Strengthen security monitoring:** Monitor for unusual activity, unauthorised access attempts, and sequential resource enumeration. Correlating application and identity logs can help identify these behaviours early.

Thanks to Flare, SANS, and WiCyS for a fun challenge! And a bonus thank you for the T-shirt for finishing in the top 250 :)



