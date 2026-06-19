# 📖 Hacker101 CTF — Level 6: Postbook

**Platform:** Hacker101 (HackerOne)
**Flags Found:** 1 / ?
**Vulnerability Class:** IDOR (Insecure Direct Object Reference) — URL Parameter Manipulation

---

## 🗺️ Table of Contents

- [Overview](#-overview)
- [Recon](#-recon)
- [Flag 1 — IDOR via `id` Parameter](#-flag-1--idor-via-id-parameter)
  - [Steps](#steps)
  - [URL Analysis](#url-analysis)
  - [Exploitation](#exploitation)
  - [Flag](#flag)
- [Vulnerability Breakdown](#-vulnerability-breakdown)
- [Bug Hunter Mindset](#-bug-hunter-mindset)
- [Key Takeaways](#-key-takeaways)

---

## 📌 Overview

**Postbook** is a simulated social blog application where users can sign up, log in, and view blog posts. The vulnerability exploited here is a classic **IDOR (Insecure Direct Object Reference)** — the application exposes a numeric `id` parameter in the URL that directly maps to blog post records, with no access control enforcing who can view what.

By simply incrementing or changing the `id` value, an attacker can access posts belonging to other users.

---

## 🔍 Recon

- Navigated to the Postbook web application
- **Signed up** with:
  - Username: `test`
  - Password: `test`
- Submitted and logged in successfully
- Observed the **blog post view page** — noticed the URL structure contained a numeric `id` parameter

---

## 🚩 Flag 1 — IDOR via `id` Parameter

### Steps

1. After login, navigate to a blog post
2. Observe the URL structure:
   ```
   https://<instance>.ctf.hacker101.com/index.php?page=view.php&id=3
   ```
3. Note that `id=3` corresponds to your own post or a default post
4. **Manually change the `id` parameter** to other values (`1`, `2`, `4`, etc.)
5. At `id=2`, the application returns a post belonging to another user — and reveals the flag

---

### URL Analysis

| Parameter | Value | Purpose |
|---|---|---|
| `page` | `view.php` | Specifies which page/module to load |
| `id` | `3` → `2` | Numeric ID referencing a specific blog post record |

**No session check or ownership validation** is performed on the `id` parameter. Any authenticated (or unauthenticated) user can access any post by changing the number.

---

### Exploitation

**Original URL (your post):**
```
https://<instance>.ctf.hacker101.com/index.php?page=view.php&id=3
```

**Tampered URL (another user's post):**
```
https://<instance>.ctf.hacker101.com/index.php?page=view.php&id=2
```

> 💡 **Tip:** Always try `id=0`, `id=1`, `id=2` first — lower IDs often belong to admin or system-generated posts which are higher value targets.

---

### Flag

```
^FLAG^340d6e0293d24db228023468c6a20b148e4e2eef652d4aa7b045dfea44daa9eb$FLAG$
```

**Found at:**
```
https://1eb650c8986b70a0fa89ed79e38d0864.ctf.hacker101.com/index.php?page=view.php&id=2
```

---

## 🧠 Vulnerability Breakdown

| Property | Detail |
|---|---|
| **Vulnerability Type** | IDOR (Insecure Direct Object Reference) |
| **OWASP Category** | A01:2021 – Broken Access Control |
| **Attack Vector** | URL query parameter manipulation (`id`) |
| **Root Cause** | No server-side ownership/access check on the `id` parameter |
| **Impact** | Unauthorized access to other users' blog posts / data |
| **Tool Used** | Browser (manual URL manipulation) |

---

## 🎯 Bug Hunter Mindset

When you see any **numeric ID or reference in a URL or request**, ask:

- **Can I access other users' objects by changing this ID?** — Try `id=1`, `id=2`, sequential enumeration
- **Is access control enforced server-side?** — Changing the ID should return a 403/404 if proper controls exist
- **What other parameters reference objects?** — Not just `id`; look for `post_id`, `user_id`, `file`, `doc`, `order`, etc.
- **What's the lowest ID?** — ID=1 or ID=0 often belongs to admin or contains sensitive data

### Checklist for IDOR Testing

- [ ] Identify all numeric/object reference parameters in URLs and request bodies
- [ ] Create two test accounts (e.g., `test` / `test2`) to cross-reference ownership
- [ ] Enumerate IDs sequentially — start from `0` or `1`
- [ ] Try accessing other users' resources while authenticated as a different user
- [ ] Check if unauthenticated access is also possible (no login required)
- [ ] Test POST/PUT/DELETE requests too — not just GET/view

---

## 📝 Key Takeaways

- **IDOR is the #1 most common web vulnerability** (OWASP A01: Broken Access Control)
- The fix is simple: **always verify on the server side that the logged-in user owns or has permission to access the requested object**
- Never rely on the client to enforce access — IDs in URLs, hidden fields, or cookies are all tamper-able
- Even a basic app like a blog must check: *"Does this user own post ID X before showing it?"*
- IDOR bugs in real bug bounty programs can be **P1/P2 severity** depending on the data exposed

---

