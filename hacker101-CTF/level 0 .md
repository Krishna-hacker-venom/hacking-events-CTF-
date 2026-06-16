## Level 0

### Challenge Description
A little something to get you started.

---

## Approach

When starting a CTF challenge, the first step is always to inspect the webpage.

### Step 1: View Page Source
I opened the page source using:
- Right click → **View Page Source**
- Or `Ctrl + U`

### Step 2: Look for Hidden Resources
While reviewing the HTML and CSS, I found the following line:

```css
background-image: url("background.png");
````

This indicates that the webpage is loading an image file named `background.png`.

---

## Exploitation

### Step 3: Access the Resource Directly

Since the file path is relative, I manually accessed it by appending it to the base URL:

```
https://15e6f696bbd88ef76ea08ccce04aeca1.ctf.hacker101.com/background.png
```

Opening this URL in the browser revealed the hidden content (flag).

---

## Flag

```
^FLAG^46c1439cab66a6f1d30c2014473b6f340035e7ec06e68c9703fa0c3d0844fe5d$FLAG$
```

---

## Key Learning

* Always check the **page source** for hidden files or endpoints.
* Look for:

  * Images (`.png`, `.jpg`)
  * Scripts (`.js`)
  * Stylesheets (`.css`)
* Developers sometimes leave sensitive information in static files.
* Direct access to resources can expose hidden data.

---

## Explanation (Beginner Friendly)

* Websites use **HTML + CSS** to render content.

* The line:

  ```css
  background-image: url("background.png");
  ```

  means the site is loading an image from the server.

* Even if it's just a background image, you can:

  1. Copy the file path
  2. Open it directly in the browser

* In CTFs, these files often contain:

  * Hidden flags
  * Clues
  * Encoded data

---

## Alternative Methods

* Use browser **DevTools → Network tab** to see all loaded files
* Use tools like:

  * `curl`
  * `wget`
  * Burp Suite (Proxy → HTTP History)

Example:

```bash
curl https://15e6f696bbd88ef76ea08ccce04aeca1.ctf.hacker101.com/background.png
```

---

## Conclusion

This level demonstrates a basic but important concept:

> **Never trust that hidden files are actually hidden.**

