
# Penetration Testing Report — FLAG26
### PwnTillDawn | Linux Target: 10.150.150.178
**CLO2: Measures vulnerabilities and risk of system and application**

---

## Part 1.1 — Define Scope & Target

### Assessment Scope Statement

This assessment is conducted within the PwnTillDawn Online Battlefield, a controlled and legal penetration testing environment provided by Wizlynx Group. The scope is limited to the target machine at IP address **10.150.150.178**, accessible exclusively through the PwnTillDawn VPN. No systems outside the 10.150.150.0/24 range were targeted. The objective is to identify and exploit vulnerabilities in order to retrieve the available flags, simulating a real-world black-box penetration test.

### Target Description

| Field | Details |
|---|---|
| **IP Address** | 10.150.150.178 |
| **Operating System** | Ubuntu Linux 64-bit |
| **Platform** | PwnTillDawn Online Battlefield |
| **Difficulty** | Medium |
| **Flags** | FLAG7, FLAG8, FLAG26, FLAG27 |

**Justification:** This Linux machine was selected because it exposes web services, offering a realistic attack surface for practicing web application enumeration, client-side code analysis, and command injection techniques — all common vulnerabilities encountered in real-world penetration tests.

### Network Map

```
[Attacker Machine - Fedora]
        |
        | VPN (OpenVPN)
        | 10.66.66.130
        |
   [PwnTillDawn VPN Gateway]
        |
        | 10.150.150.0/24
        |
   [Target: 10.150.150.178]
   ┌─────────────────────┐
   │  Ubuntu Linux       │
   │  Port 22  → SSH     │
   │  Port 80  → HTTP    │
   └─────────────────────┘
```

### Tools Selection & Rationale

| Tool | Purpose | Justification |
|---|---|---|
| **Nmap** | Port scanning & service detection | Industry standard, scriptable, provides OS/service fingerprinting |
| **Gobuster** | Web directory enumeration | Fast, multi-threaded, supports multiple extensions |
| **curl** | HTTP interaction & exploitation | Native, flexible, ideal for manual testing of endpoints |
| **Browser DevTools** | Client-side code analysis | Reveals JavaScript logic, hidden endpoints, and commented code |

---

## Part 1.2 — Methodology, Scanning & Analysis

### Reconnaissance & Scanning

#### Step 1 — Full Port Scan

```bash
nmap -p- --min-rate=5000 -n -Pn 10.150.150.178 -oG allports.txt
```

**Result:**

| Port | State | Service |
|---|---|---|
| 22/tcp | open | ssh |
| 80/tcp | open | http |
| 443/tcp | closed | https |

#### Step 2 — Service Version Detection

```bash
nmap -p 22,80 -sC -sV -Pn 10.150.150.178 -oN targeted.txt
```

**Result:**
- Port 22: OpenSSH (Ubuntu Linux)
- Port 80: Apache HTTP Server 2.4

#### Step 3 — Web Directory Enumeration

<img width="697" height="453" alt="login" src="https://github.com/user-attachments/assets/0c3e6384-0298-4dd5-b3e8-4522f3bedc15" />


```bash
gobuster dir -u http://10.150.150.178 \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,txt,html -t 50
```

**Directories found:**

| Path | Status | Notes |
|---|---|---|
| `/docs/` | 301 → 403 | Forbidden — exists but not accessible |
| `/robots.txt` | 200 | Reveals hidden paths |
| `/server-status` | 403 | Apache status page (restricted) |

#### Step 4 — robots.txt Analysis

```bash
curl http://10.150.150.178/robots.txt
```

```
User-agent: *
Disallow: /
Disallow: /index.html
Disallow: /tmp/
Disallow: /Administration/
Disallow: /STAdmin/Dashboard/
Disallow: /docs/
```

The `robots.txt` file reveals several sensitive paths, including `/STAdmin/Dashboard/` — an administration panel not linked from the main page.

#### Step 5 — Admin Panel Discovery

Navigating to `http://10.150.150.178/STAdmin/Dashboard/` reveals a functional administration dashboard exposing system commands (ps, vmstat, uname, df) via a web form.

### Enumeration Table

| Finding | Value | Risk Level |
|---|---|---|
| Open SSH port | 22/tcp | Medium |
| Apache web server | Port 80 | Low |
| Sensitive paths in robots.txt | `/STAdmin/Dashboard/`, `/docs/` | High |
| Admin dashboard exposed | `/STAdmin/Dashboard/index.html` | Critical |
| Custom JavaScript file | `MyCustomScript.js` | High |
| Commented PHP endpoint | `getflag26.php` | Critical |

### Risk-Based Analysis

The most critical finding is the **exposure of the admin dashboard** (`/STAdmin/Dashboard/`) which runs system commands server-side. Combined with the discovery of a custom JavaScript file containing commented-out PHP endpoints, this creates a direct path to sensitive data without requiring authentication.

The `robots.txt` file, intended to hide paths from search engines, paradoxically acts as a **roadmap for attackers** by explicitly listing restricted directories.

---

## Part 1.3 — Exploitation & Flag Acquisition

### Attack Path

#### Step 1 — Source Code Analysis

The main login page (`http://10.150.150.178/index.html`) references a custom JavaScript file visible in the HTML source:

```html
<script src="MyCustomScript.js"></script>
```

```bash
curl http://10.150.150.178/MyCustomScript.js
```

#### Step 2 — Discovery of Commented Endpoint

<img width="533" height="231" alt="customjs" src="https://github.com/user-attachments/assets/61e7a844-c7a9-43b8-9546-33c195bd9e96" />


The JavaScript file contains the following login handler code:

```javascript
$("#login-button").click(function (event) {
    event.preventDefault();

    // $.get("getflag26.php", function(value){
    //     alert(value);
    // });

    setTimeout(myFunction, rand * 1000);
});

function myFunction() {
    $('form')[0].reset();
    $('#login-response').html("The Username or Password is Incorrect");
}
```

**Key observations:**
- The login form is **entirely fake** — it always returns "incorrect" regardless of credentials
- A call to `getflag26.php` is **commented out**, revealing a hidden PHP endpoint

#### Step 3 — Flag Retrieval

Directly accessing the commented endpoint:

```bash
curl http://10.150.150.178/getflag26.php
```

<img width="659" height="438" alt="curl-flag" src="https://github.com/user-attachments/assets/7011c812-36b8-4b2f-ba72-304e31a2b01a" />


**Result:**
```
14f1591562de8892b8d7ed9c4ceb31ed076b556d
```
<img width="697" height="453" alt="login" src="https://github.com/user-attachments/assets/c5eee492-9c03-449e-9b0f-aee0e670785b" />
<img width="542" height="57" alt="flag-found" src="https://github.com/user-attachments/assets/9837414d-9c04-46ad-ad14-03dcc8cce4e5" />


### FLAG26 Captured ✅

```
FLAG26 = 14f1591562de8892b8d7ed9c4ceb31ed076b556d
```

### Summary of Exploited Vulnerability

| Field | Details |
|---|---|
| **Vulnerability Type** | Information Disclosure via Client-Side Code |
| **OWASP Category** | A05:2021 — Security Misconfiguration |
| **CWE** | CWE-615: Inclusion of Sensitive Information in Source Code Comments |
| **Severity** | High |
| **Attack Vector** | Network (no authentication required) |
| **Remediation** | Remove sensitive endpoints from client-side JavaScript; implement server-side authentication before exposing any flag or sensitive endpoint |

### Attack Path Summary

```
1. Port scan → Port 80 open (HTTP)
        ↓
2. robots.txt → Reveals /STAdmin/Dashboard/
        ↓
3. Main page source → References MyCustomScript.js
        ↓
4. JavaScript analysis → Commented getflag26.php endpoint
        ↓
5. curl getflag26.php → FLAG26 retrieved

```
<img width="817" height="106" alt="flag26" src="https://github.com/user-attachments/assets/6ef37ba6-38d7-49af-9011-f17a6364b223" />
---
