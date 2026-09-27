# google-forms-phishing-abuse
Technical analysis, IOCs, and MITRE ATT&amp;CK mapping for a phishing campaign abusing legitimate Google Forms infrastructure and Cloudflare Pages to evade secure email gateways.
# Threat Intelligence Report: Google Forms Abuse Exploiting Trusted Infrastructure

## Executive Summary
This repository documents a sophisticated phishing campaign that abuses legitimate Google Forms automated response infrastructure to bypass corporate email gateways. By exploiting automated form receipts (`forms-receipts-noreply@google.com`), threat actors successfully land malicious links directly in user mailboxes. The campaign leverages trusted cloud-hosting domains (`pages.dev`) and utilizes shared CDN infrastructure to evade traditional signature and network-layer reputation controls.

---

## Technical Analysis & Execution Flow

### 1. Delivery & Infrastructure Abuse
The threat actor builds a malicious input form hosted on Google Forms. By supplying the target's email address and triggering the **"Send copy of responses"** setting, the platform autogenerates a notification mail. 

Because the email is physically dispatched by Google's infrastructure, it passes standard authentication checks:
* **SPF:** `PASS` (signed by `google.com`)
* **DKIM:** `PASS` (signed by `google.com`)

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
Detection scripts, including custom **YARA rules** for raw email parser routing and **Splunk queries** for corporate proxy logging, can be found in the https://github.com/Zezanderson98/google-forms-phishing-abuse/blob/738c00669fd179a3ebacccafa1a84e8cc63f1d57/detections directory of this repository.

## Screenshots & Visual Evidence

### 1. Phishing Delivery & Mail Headers
Evidence of the initial delivery mechanism showing the automated Google Forms sender identity 
![Gmail Header & Payload View](assets/01_gmail_delivery_evidence.png)

<img width="1365" height="754" alt="mailcoinn" src="https://github.com/user-attachments/assets/07023917-f80e-4aea-8038-bd06e7f4c94b" />


