# Hackyholidays CTF | Writeup

## Challenge Summary

**CTF:** Hackyholidays  
**Type:** Web Security

---

## Flags

### Flag 1
```
^FLAG^75127fe0aa8953fe8f1615a98360d29d4f2853facfe90e0c67c994af94c2ac01$FLAG$
```

### Flag 2
```
^FLAG^86f2f7888f732694b7f140cd4cd0d76a79993090568c5a396253e6918d0175e1$FLAG$
```

---

## Vulnerabilities

1. **Information Disclosure via robots.txt**
2. **Sensitive Data Exposed in HTML Comments/Body**

---

## Solution

### Flag 1: robots.txt Reconnaissance

#### Vulnerability
The `robots.txt` file is publicly accessible and contains paths that should not be indexed. While intended for search engines, this file often reveals sensitive directories and endpoints.

#### Steps

1. **Navigate to robots.txt**
   ```
   https://276829be96169391d30b2b1ab51dd306.ctf.hacker101.com/robots.txt
   ```

2. **Analyze the file**
   - The `robots.txt` reveals disallowed paths that hint at hidden functionality
   - These paths would be intentionally hidden from search engines but accessible via direct URL

3. **Discover the Flag**
   - The robots.txt file contains or references the flag content
   - Simply visiting the URL and inspecting the response reveals: `^FLAG^75127fe0aa8953fe8f1615a98360d29d4f2853facfe90e0c67c994af94c2ac01$FLAG$`

#### Technical Details

**robots.txt Purpose:**
- Instructs web crawlers which paths to skip
- Not a security mechanism—all listed paths are accessible
- Often contains valuable information for reconnaissance

**Best Practices:**
```
# ❌ DON'T do this:
User-agent: *
Disallow: /admin/
Disallow: /api/secret/
Disallow: /s3cr3t-ar3a/
```

---

### Flag 2: Hidden Path Discovery & HTML Inspection

#### Vulnerability
Sensitive information exposed in HTML source code (body tag attributes or comments) without proper access controls.

#### Steps

1. **Navigate to Hidden Path**
   ```
   https://276829be96169391d30b2b1ab51dd306.ctf.hacker101.com/s3cr3t-ar3a/
   ```

2. **Inspect the Page Source**
   - Right-click on the page → **Inspect Element** (or press `F12`)
   - Navigate to the **Elements/Inspector** tab
   - Examine the `<body>` tag and its contents

3. **Locate the Flag**
   - The flag is embedded directly in the HTML body
   - It may be hidden in:
     - HTML comments: `<!-- ^FLAG^...$ -->`
     - Data attributes: `<body data-flag="...">`
     - Visible or hidden text nodes within the body element
   
4. **Extract the Flag**
   ```
   ^FLAG^86f2f7888f732694b7f140cd4cd0d76a79993090568c5a396253e6918d0175e1$FLAG$
   ```

#### Technical Details

**Why This Happens:**
- Developers often hardcode secrets in HTML for testing
- Client-side "hiding" via CSS (`display: none`, `opacity: 0`) provides no security
- JavaScript can easily access any HTML content
- No access control on the endpoint

**Common Mistake:**
```html
<!-- ❌ DON'T do this -->
<body>
  <div style="display: none;">
    Flag: ^FLAG^secret$FLAG$
  </div>
  <!-- Secret API key: sk-12345abcde -->
</body>
```

---

## Exploitation Chain

```
1. Reconnaissance
   ↓
   Visit robots.txt → Discover /s3cr3t-ar3a/
   ↓
2. Enumeration
   ↓
   Navigate to hidden path
   ↓
3. Information Extraction
   ↓
   Inspect HTML source → Extract embedded secrets
   ↓
4. Flag Capture
   ↓
   ✅ Both flags obtained
```

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Browser (Chrome/Firefox)** | Navigation, inspection |
| **Developer Tools (F12)** | HTML source inspection |
| **curl/wget** | Command-line request to robots.txt |
| **Burp Suite** | Traffic analysis (optional) |

### Command-Line Approach
```bash
# Flag 1: Fetch robots.txt
curl https://276829be96169391d30b2b1ab51dd306.ctf.hacker101.com/robots.txt

# Flag 2: Fetch hidden path HTML
curl https://276829be96169391d30b2b1ab51dd306.ctf.hacker101.com/s3cr3t-ar3a/ | grep -i flag
```

---

## Security Concepts

### Information Disclosure (CWE-200)

**Definition:**  
The application exposes sensitive information to an actor who is not explicitly authorized to access that information.

**Attack Surface in This Challenge:**
1. ✅ **robots.txt** - Intentionally public but reveals structure
2. ✅ **HTML Source** - Client-side exposure without access control
3. ✅ **Path Enumeration** - Predictable directory names

### Remediation Best Practices

#### For Flag 1 (robots.txt):
```
✅ DO:
- Keep robots.txt minimal and generic
- Never list sensitive paths
- Remember: it's informational, not protective

❌ DON'T:
- List admin panels, APIs, or secret directories
- Assume robots.txt provides security
- Store secrets in comments within it
```

#### For Flag 2 (HTML Exposure):
```
✅ DO:
- Server-side render sensitive data (don't send to browser)
- Use authentication/authorization gates
- Implement proper access controls
- Never embed secrets in HTML, CSS, or JavaScript

❌ DON'T:
- Hide secrets with CSS (`display: none`)
- Put flags/keys in HTML comments
- Trust client-side data hiding
- Assume source inspection prevents access
```

---
