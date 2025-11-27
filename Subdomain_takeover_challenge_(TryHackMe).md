
## 📝 Walkthrough Steps

### 1. Deploy the Machine and Connect via VPN
- Connect to the TryHackMe network using your **OpenVPN configuration file**.  
- Start the machine to get its **IP address**.

---

### 2. Add Domain to `/etc/hosts` File
- Map the main domain `futurevera.thm` to the machine's IP address for proper name resolution.  
- Example entry (replace `<THM-IP>` with the actual machine IP):

```bash
sudo echo <THM-IP> futurevera.thm >> /etc/hosts
```

---

### 3. Initial Reconnaissance (Optional but Recommended)
- Run a basic **nmap scan** to identify open ports (commonly 22, 80, 443).  
- Visit [https://futurevera.thm](https://futurevera.thm) and inspect the source code.  
  *(In this room, the source won’t reveal much.)*

---

### 4. Subdomain Enumeration
- Use tools like **ffuf**, **gobuster (vhost mode)**, or **dnsrecon** with a wordlist (e.g., from SecLists).  
- Example `ffuf` command:

```bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/namelist.txt \
-H "Host: FUZZ.futurevera.thm" -u http://<THM-IP> -fs <size>
```

- This should reveal:
  - `blog.futurevera.thm`  
  - `support.futurevera.thm`

---

### 5. Investigate Discovered Subdomains
- Add the new subdomains to your `/etc/hosts` file.  
- Visit them in your browser:
  - **Blog** → dead end.  
  - **Support** → described as “being rebuilt” (critical clue).

---

### 6. Analyze the SSL Certificate
- Visit [https://support.futurevera.thm](https://support.futurevera.thm).  
- Inspect the **SSL certificate details** → check **Subject Alternative Names (SANs)**.  
- You’ll find a hidden subdomain, e.g.:

```
secrethelpdesk934752.support.futurevera.thm
```

*(Exact name may vary per room instance.)*

---

### 7. Access the Final Subdomain
- Add the secret subdomain to your `/etc/hosts` file.  
- Access it via **HTTP (not HTTPS)**.  
- The page redirects to an **AWS S3 error page** → the flag is visible directly in the **redirection URL** in your browser’s address bar.
