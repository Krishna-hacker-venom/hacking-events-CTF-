# TryHackMe: Room 404 - Exposed .git Directory Exploitation
**Challenge Type:** Web Application Security | Information Disclosure  
**Difficulty:** Very Easy  
**Category:** Configuration Misconfiguration  
**Target:** https://tryhackme.com/room/hh-room404-804573bf

---

## Executive Summary

This writeup documents the exploitation of an exposed `.git` directory on the Byte Lotus guest-experience platform running on port 8080. The vulnerability stems from misconfigured web server settings that permit directory listing and direct access to version control system files. Through methodical reconnaissance and automated repository dumping, the complete source code repository was reconstructed offline, resulting in the extraction of sensitive staging credentials and flags.

**Impact Level:** CRITICAL  
**CVSS Score:** 7.5 (High)  
**Vulnerability Classification:** CWE-215 (Information Exposure Through Debug Information)

---

## Hacker Methodology & Mindset

### Phase 1: Information Gathering & Reconnaissance

#### 1.1 Initial Recon Strategy

Before running aggressive automated scans, a professional bug hunter gathers contextual intelligence:

| Reconnaissance Tactic | Purpose | Output |
|---|---|---|
| **Read Challenge Hints** | Extract clues embedded in problem statement | Identified developer negligence as attack vector |
| **Port Scanning** | Map accessible services | Confirmed port 8080 hosting web application |
| **Manual Path Testing** | Test common misconfigurations before automated tools | Direct access to .git directory confirmed |
| **Version Control Enumeration** | Identify exposed VCS artifacts | Discovered directory listing enabled |

#### 1.2 The Critical Thinking Process

The hacker's mindset prioritizes efficiency and pattern recognition:

1. **Analyze the Challenge Narrative:** The room description states: *"The Byte Lotus guest-experience platform went live in a hurry, and the night-shift developer shipped more than the website."*
   - This is a deliberate hint pointing to developer negligence
   - "Shipped more than the website" suggests accidentally exposed files
   - Night-shift deployments often lack proper review processes

2. **Reject Blind Automation:** While tools like `dirsearch` are powerful, they can be inefficient
   - Large wordlists consume time without targeted intelligence
   - Manual testing of common VCS paths (`.git`, `.svn`, `.env`, `.hg`) is faster
   - A professional pentester balances automation with manual reconnaissance

3. **Prioritize Low-Hanging Fruit:** Common misconfigurations before deep exploitation
   - Version control folders are frequently exposed
   - Directory listing is a common server misconfiguration
   - These are high-impact, easy-to-verify vulnerabilities

---

### Phase 2: Vulnerability Discovery & Analysis

#### 2.1 Identifying the Exposed .git Directory

**Manual Testing Approach:**

```bash
# Test for directory listing enabled
curl -v http://<TARGET_IP>:8080/.git/

# Expected response: HTTP 200 with directory index HTML
# Directory listing reveals:
# - .git/HEAD
# - .git/objects/
# - .git/refs/
# - .git/index
# - .git/logs/
```

**Why This Works:**

The web server (likely Apache or Nginx) was misconfigured with:
1. **Directory listing enabled** - `Options +Indexes` in Apache or `autoindex on` in Nginx
2. **No access restrictions** - No `.htaccess` deny rules or authentication requirements
3. **Git folder exposed** - `.git/` not explicitly blocked from public access

#### 2.2 Understanding Git Object Storage Structure

Git stores repository data in a compressed binary format:

| Git Component | Location | Data Type | Purpose |
|---|---|---|---|
| **HEAD** | `.git/HEAD` | Text reference | Points to current branch |
| **Objects** | `.git/objects/` | Zlib compressed binaries | Stores commits, trees, blobs |
| **Index** | `.git/index` | Binary format (v2) | Staging area snapshot |
| **Refs** | `.git/refs/heads/` | Text hash references | Branch pointers |
| **Logs** | `.git/logs/` | Text commit history | Reflog entries |

**Technical Analysis:**

When examining `.git/objects/`, you encounter SHA-1 hashed blob objects:

```bash
# Example object identifier
13550b4cb13e9f30c61d5b342c532d21e45bda

# File type analysis
$ file 13550b4cb13e9f30c61d5b342c532d21e45bda
13550b4cb13e9f30c61d5b342c532d21e45bda: zlib compressed data

# Manual decompression (inefficient at scale)
$ python3 -c "import zlib; print(zlib.decompress(open('13550b4cb13e9f30c61d5b342c532d21e45bda', 'rb').read()))"
```

**Key Insight:** Manual decompression across 100+ objects over HTTP is impractical. Professional attackers automate this process.

---

### Phase 3: Exploitation & Repository Reconstruction

#### 3.1 Tool Selection: git-dumper

**Why git-dumper?**

1. **Automated Recursion** - Fetches all accessible Git objects sequentially
2. **Object Decompression** - Handles zlib decompression automatically
3. **Repository Recreation** - Reconstructs a working `.git/` directory locally
4. **Time Efficiency** - Reduces hours of manual work to minutes

#### 3.2 Exploitation Execution

**Step 1: Deploy git-dumper**

```bash
# Installation
pip install git-dumper

# Execution
python3 -m git_dumper http://<TARGET_IP>:8080/.git/ dumped_repo

# Alternative syntax
git-dumper http://<TARGET_IP>:8080/.git/ dumped_repo
```

**Expected Output:**

```
[*] Fetching common files and directories...
[*] Fetching objects...
[*] Cloning repository...
[+] Successfully dumped git repository
```

**Step 2: Navigate Repository**

```bash
cd dumped_repo
ls -la

# Output structure
-rw-r--r-- app.js
-rw-r--r-- index.html
-rw-r--r-- README.md
drwxr-xr-x .git/
```

#### 3.3 Sensitivity Analysis: What Was Exposed

| File | Sensitivity | Content Type | Threat |
|---|---|---|---|
| **app.js** | Medium | Application logic, API endpoints | Business logic disclosure |
| **index.html** | Low | UI markup | UI/UX strategy exposure |
| **README.md** | CRITICAL | Staging notes, credentials, flags | Direct flag extraction |
| **.git/logs/HEAD** | High | Commit history, developer actions | Operational timeline |

---

### Phase 4: Post-Exploitation Analysis

#### 4.1 Sensitive Data Extraction

**README.md Contents:**

```markdown
# Byte Lotus — Guest Experience Platform
Internal staging repository for the guest app and concierge personalization
service. Do not deploy this folder to production.

Staging flag (remove before launch): THM{************************}
```

**Analysis of Poor Security Practice:**

1. **Credential Embedding** - Sensitive information in documentation
2. **No Pre-deployment Review** - "Remove before launch" indicates rushed process
3. **No Access Controls** - Staging notes should not be in version control
4. **Git Exposure** - Critical failure: `.git/` should never be web-accessible

#### 4.2 Advanced Exploitation Potential

A sophisticated attacker could extract:

1. **Commit History** - Track development timeline and developer actions
   ```bash
   cd dumped_repo && git log --oneline
   ```

2. **Author Information** - Identify developers for targeted social engineering
   ```bash
   git log --format="%an <%ae>"
   ```

3. **Deleted Secrets** - Recover credentials removed in later commits
   ```bash
   git log -p | grep -i "password\|api_key\|token"
   ```

4. **Branch Analysis** - Identify experimental features and vulnerabilities
   ```bash
   git branch -a && git checkout experimental-branch
   ```

---

## Security Lessons & Mitigation

### 5.1 Root Cause Analysis

| Failure Point | Technical Cause | Prevention |
|---|---|---|
| **Directory Listing Enabled** | Misconfigured web server | Disable `Options +Indexes` / `autoindex off` |
| **.git/ Not Blocked** | No explicit access denial | Add deny rules for `.git` in `.htaccess` / nginx config |
| **Staging Secrets in Repo** | Poor secrets management | Use environment variables, `.gitignore`, or secret vaults |
| **No Deployment Review** | Rushed release process | Implement pre-deployment checklists |

### 5.2 Defensive Measures (Defense-in-Depth)

**Web Server Configuration:**

```apache
# Apache (.htaccess)
<FilesMatch "^\.git">
    Order allow,deny
    Deny from all
</FilesMatch>

# Or globally
<Directory ~ "\.git">
    Require all denied
</Directory>
```

```nginx
# Nginx
location ~ /\.git {
    deny all;
}
```

**Application-Level Protections:**

1. **Environment Variable Isolation**
   ```bash
   # Store secrets outside repository
   export DB_PASSWORD="***"
   export API_TOKEN="***"
   ```

2. **`.gitignore` Configuration**
   ```
   .env
   .env.local
   secrets/
   credentials.json
   config/staging.yml
   ```

3. **Git Hooks - Pre-commit Scanning**
   ```bash
   # Prevent secrets from being committed
   git-secrets install
   git secrets --register-aws
   ```

**Deployment Checklist:**

- [ ] All `.git/` directories removed from production servers
- [ ] `.env` files excluded from deployment packages
- [ ] Directory listing disabled on all web servers
- [ ] Security headers configured (CSP, X-Content-Type-Options)
- [ ] Sensitive files removed from version control history

---

## Tools & Resources Used

### Essential Exploitation Tools

| Tool | Purpose | Installation |
|---|---|---|
| **git-dumper** | Automated .git extraction | `pip install git-dumper` |
| **git** | Repository analysis | Pre-installed on most systems |
| **curl/wget** | HTTP reconnaissance | Standard utilities |
| **zlib** (Python library) | Binary decompression | `python3 -c "import zlib"` |

### Additional Security Tools for Similar Scenarios

```bash
# Subdomain enumeration with git endpoints
subfinder -d target.com | grep -E "\.git|git\."

# Sensitive file discovery
dirsearch -u http://target.com -w wordlist.txt -i 200

# Secrets scanning on extracted repos
truffleHog filesystem dumped_repo/

# Credential extraction from git history
grep -rE "password|api_key|token" dumped_repo/.git/objects/
```

---

## Hacker's Takeaways & Professional Insights

### Methodology Principles

1. **Read the Context** - Challenge hints and business logic guide reconnaissance
2. **Manual Before Automated** - Quick manual testing often faster than full scans
3. **Common Misconfigurations First** - Version control exposure is high-probability
4. **Automate Repetitive Tasks** - Use tools for bulk operations (git-dumper)
5. **Analyze Post-Exploitation** - Extract maximum intelligence from compromised systems

### Key Insights for Bug Hunters

1. **VCS Exposure is Critical** - `.git`, `.svn`, `.hg` directories expose entire codebase
2. **Configuration Issues Trump Coding Flaws** - Most vulnerabilities stem from misconfiguration
3. **Developer Negligence Patterns** - Night-shift deployments often lack security reviews
4. **Staged Environments Leak Secrets** - README files, TODO comments, hardcoded credentials
5. **Recursive Exploitation** - One vulnerability (exposed .git) leads to flag extraction

### Professional Impact

**For Security Researchers:**
- Document findings with full context and methodology
- Provide defensive measures and remediation steps
- Demonstrate understanding of both attacker and defender perspectives

**For Organizations:**
- Implement automated pre-deployment security scanning
- Enforce secrets management policies
- Conduct security training on deployment best practices
- Regular penetration testing to identify misconfigurations

---

## References & Further Reading

- **OWASP:** Information Exposure Through Debug Information (CWE-215)
- **Git Security:** https://git-scm.com/book/en/v2/Git-Internals-Git-Objects
- **MITRE ATT&CK:** T1526 (Reconnaissance - Develop Capabilities)
- **git-dumper Project:** https://github.com/arthaud/git-dumper

---
