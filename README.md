# google-forms-phishing-abuse
Technical analysis, IOCs, and MITRE ATT&amp;CK mapping for a phishing campaign abusing legitimate Google Forms infrastructure and Cloudflare Pages to evade secure email gateways.

# Threat Intelligence Report: Google Forms Abuse Exploiting Trusted Infrastructure

## 🎯 Objective
The primary objective of this investigation was to **detect, analyze, and document an ongoing phishing evasion technique** that exploits legitimate cloud automation platform rules to deliver malicious hyperlinks. Specifically, the intent was to identify how security boundary gateways handle system-generated automated emails from `google.com`, trace the downstream redirection architecture, and build functional defense engineering metrics to proactively hunt for similar campaigns.

---

## 📊 Data Sources
The triage and behavioral analysis relied on the correlation of the following logs and telemetry sources:
* **Email Gateway Telemetry & Metadata:** Inspection of raw SMTP internet headers, envelope sender configurations, and SPF/DKIM verification results.
* **Network Proxy & DNS Logs:** Analysis of outward-bound web traffic connections, HTTP 302 redirection paths, and fully qualified domain names (FQDNs).
* **Static & Dynamic Threat Intelligence Feeds:** Querying multi-vendor signature engines via **VirusTotal** and cross-referencing global network abuse records via **AbuseIPDB**.



## 💻 Environment Used
The verification, telemetry parsing, and initial triage of this campaign were conducted entirely within a standard operational environment using the following stack:

* **Host Operating System:** Windows 10 Pro
* **Mail Client & Analyzer:** Native Gmail Web Interface (Spam Folder Triage)
* **Threat Intelligence & Reputation Pivots:** 
  * **VirusTotal** (For multi-vendor file/URL scanning and redirection chain mapping)
  * **AbuseIPDB** (For infrastructure-level reverse proxy identification and network reputation lookups)

---

## 🛠️ Steps
The investigation was completed sequentially using the following methodology:

```text
[Step 1: Header Triage] ──► [Step 2: Payload Extraction] ──► [Step 3: Reputation Pivot] ──► [Step 4: Rule Generation]
```

1. **Header Triage & Authentication Audit:** Extracted and parsed the raw email headers from the target payload to evaluate authentication mechanisms (`Authentication-Results`).
2. **Payload Extraction & Sandboxing:** Isolated the initial URL safely and tracked the automated downstream application-layer redirects to determine the final landing site.
3. **Reputation Pivoting:** Evaluated the staging URL against centralized indicator databases (VirusTotal) and queried the underlying host network infrastructure IP address against crowd-sourced abuse indexes (AbuseIPDB).
4. **Detection Engineering Modeling:** Codified the distinct attack behaviors into logical, deployable rules for enterprise-wide threat hunting.

---

## 🔍 Findings
The investigation revealed a highly successful **Defense Evasion** and **Initial Access** campaign:

* **Infrastructure Exploitation:** The attacker successfully manipulated the legitimate automated "Forms response receipts" mechanism of Google Forms. Because the email physically generated from `forms-receipts-noreply@google.com`, it yielded pristine **SPF and DKIM PASS** certifications, successfully avoiding signature-based gateway blocks.
* **Low-Detection Footprint:** The embedded staging URL (`uvgei2qcvavk[.]pages[.]dev`) yielded a **1/92 malicious detection ratio** on VirusTotal. This indicates that fresh deployments easily evade standard blocklists.
* **Network-Layer Camouflage:** The destination resolves to IP `104.21.71.88`, a shared Cloudflare Reverse Proxy with a **0% Abuse Confidence Score** on AbuseIPDB. Blocking this indicator at the firewall layer is impossible without triggering extensive false-positive business disruptions.
* **Final Intent:** The delivery infrastructure leads through a seamless javascript/HTTP redirect chain culminating in an unvetted gambling/monetization landing page (`maxbetwin[.]me`).

## Executive Summary
This repository documents a sophisticated phishing campaign that abuses legitimate Google Forms automated response infrastructure to bypass corporate email gateways. By exploiting automated form receipts (`forms-receipts-noreply@google.com`), threat actors successfully land malicious links directly in user mailboxes. The campaign leverages trusted cloud-hosting domains (`pages.dev`) and utilizes shared CDN infrastructure to evade traditional signature and network-layer reputation controls.

---

## Technical Analysis & Execution Flow

### 1. Delivery & Infrastructure Abuse
The threat actor builds a malicious input form hosted on Google Forms. By supplying the target's email address and triggering the **"Send copy of responses"** setting, the platform autogenerates a notification mail. 

Because the email is physically dispatched by Google's infrastructure, it passes standard authentication checks:
* **SPF:** `PASS` (signed by `google.com`)
* **DKIM:** `PASS` (signed by `google.com`)


[ Threat Actor ]
       │  (1) Inputs target email & malicious URL
       ▼
[ Google Forms Engine ] 
       │  (2) Generates response receipt mail (DKIM / SPF PASS)
       ▼
[ Secure Email Gateway (SEG) ] ──▶ [ YARA Rule Intercepts Pattern ]
       │  (3) Evades traditional signature checks
       ▼
[ User Inbox / Spam Folder ]
       │  (4) User interacts with embedded link
       ▼
[ uvgei2qcvavk[.]pages[.]dev ] ──▶ [ SOC Proxy Hunt / VT 1/92 Flagged ]
       │  (5) Resolves via Cloudflare Staging Subdomain
       ▼
[ maxbetwin[.]me/?promo=GIFT888 ] ──▶ [ Final Landing / Host Isolation ]
       (6) Shared CDN Reverse Proxy Masking (IP: 104.21.71.88 | AbuseIPDB 0%)



### 2. Network Evasion & Redirection Chain
Once the user interacts with the link inside the form payload, the following redirection architecture is observed:

```text
[Legitimate Google Email Receipt] 
       │
       ▼ (User clicks link)
[https://uvgei2qcvavk[.]pages[.]dev]  <-- Cloudflare Pages Subdomain (Staging)
       │
       ▼ (HTTP 302 / JS Redirect)
[https://maxbetwin[.]me/?promo=GIFT888] <-- Final Landing Page (Gambling/Payload)
```

---

## Defensive Metrics & Resource Reputation

A critical highlight of this campaign is its low defensive footprint, allowing it to evade automated Security Operations Center (SOC) triage rules.

### Resource Reputation Matrix

| Indicator | Type | Platform | Assessment / Detection Score | Tactical Interpretation |
| :--- | :--- | :--- | :--- | :--- |
| `uvgei2qcvavk[.]pages[.]dev` | Domain | VirusTotal | **1/92 Malicious** | **True Positive.** Staging URL bypassing signature engines due to fresh deployment. |
| `104.21.71.88` | IPv4 | AbuseIPDB | **0% Abuse Confidence** | **Shared Infrastructure.** Reverse Proxy IP assigned to millions of Cloudflare sites; unblockable at network layer. |

---

## MITRE ATT&CK® Mapping

| Tactic | Technique ID | Technique Name | Context / Use Case in Campaign |
| :--- | :--- | :--- | :--- |
| **Initial Access** | [T1566.002](https://mitre.org) | Phishing: Spearphishing Link | Delivering malicious URLs inside legitimate system automated messages. |
| **Defense Evasion** | [T1564](https://mitre.org) | Hide Artifacts: Trusted Infrastructure | Using `google.com` automation and valid SPF/DKIM to blind email filters. |
| **Defense Evasion** | [T1548](https://mitre.org) | Abuse Elevation Control Mechanism | Exploiting high-reputation shared CDNs (`pages.dev`) to mask final payloads. |
| **Command & Control**| [T1071.001](https://mitre.org) | Application Layer Protocol: Web Traffic | Exfiltrating telemetry or routing traffic over standardized HTTPS tunnels. |

---

## Indicators of Compromise (IoCs)
*Note: Indicators have been defanged (`[.]`) to ensure safety during analysis.*

* **Sender Address (Abused):** `forms-receipts-noreply@google[.]com`
* **Mailed-By Domain:** `trix[.]bounces[.]google[.]com`
* **Phishing Subject Line:** `Thanks for filling in this form: 🔔Special reward available: check the test results!' #169fke208`
* **Initial Staging Domain:** `https://uvgei2qcvavk[.]pages[.]dev`
* **Final Payload Redirect:** `https://maxbetwin[.]me/?promo=GIFT888`
* **Shared Infrastructure IP:** `104.21.71.88`




## Detection & Hunting Engineering
Detection scripts, including custom **YARA rules** for raw email parser routing and **Splunk queries** for corporate proxy logging, can be found in the https://github.com/Zezanderson98/google-forms-phishing-abuse/blob/739867f356c2fe1eaa8bbc00f7088d4a6a671abd/detections directory of this repository.

**View Incident Response Playbook**
https://github.com/Zezanderson98/google-forms-phishing-abuse/blob/2543c52b6a27f9f8045964cddadf6644f52c33c4/playbooks/google-forms-abuse-playbook.md

## Screenshots & Visual Evidence

https://github.com/Zezanderson98/google-forms-phishing-abuse/blob/e6c12c7c1a14c1381780b733635faf87f2b02e99/screenshots.md
