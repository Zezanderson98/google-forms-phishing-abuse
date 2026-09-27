# Incident Response Playbook: Legitimate Cloud Infrastructure Abuse (Google Forms / Pages.dev)

## 1. Overview & Objective
* **Playbook ID:** IR-PB-042
* **Target Vector:** Input Form Platform Abuse (Google Forms Automated Receipts)
* **Objective:** Standardize triage, isolation, scoping, and eradication steps when a user receives or interacts with a phishing link arriving via high-reputation, legitimate automated cloud infrastructure.

---

## 2. Playbook Workflow (NIST Lifecycle)

```text
  [1. Preparation] ──► [2. Detection & Analysis] ──► [3. Containment & Eradication] ──► [4. Post-Incident]
  (SIEM/YARA Rules)       (Scope Link Clicks)          (Purge Mail / Block Subdomain)     (Adjust Policies)
```

### Phase 1: Preparation
Ensure defensive rules are active to flag this specific Delivery Technique ([MITRE T1566.002](https://mitre.org)):
1. Deploy the `Detect_Google_Forms_Phishing_Abuse` **YARA rule** across mail inspection gateways.
2. Ensure proxy/EDR logs index all egress network calls to cloud-hosting subdomains (`*.pages.dev`, `*.workers.dev`, `*.vercel.app`).

### Phase 2: Detection & Analysis (Triage & Scoping)
Upon receipt of a alert or user submission:

1. **Verify Mail Authenticity:**
   * Confirm the mail stems from `forms-receipts-noreply@google.com`.
   * Check SPF/DKIM validation. If it shows `PASS`, the campaign is actively abusing legitimate Google infrastructure.
2. **Determine Scope (Blast Radius):**
   * Execute a SIEM hunt using the subject string pattern `Thanks for filling in this form:*` to see how many users received the email.
3. **Analyze Payload & Redirects:**
   * Extract the embedded URL (e.g., `uvgei2qcvavk[.]pages[.]dev`).
   * Run a sandboxed analysis or VirusTotal lookup to find the down-stream redirect target domain (e.g., `maxbetwin[.]me`).
4. **Identify User Interaction:**
   * Cross-reference Web Proxy and DNS logs against user source IPs to check if any internal assets attempted connections to the staging domain or final redirect URL.

### Phase 3: Containment & Eradication
If network logs indicate **No Clicks** occurred:
* **Email Purge:** Issue an administrative delete command via your mail platform (e.g., `Search-Mailbox` / `ComplianceSearch` in O365) to remove the email from all identified user inboxes.

If network logs indicate **Active Clicks / User Interaction**:
* **Host Isolation:** Immediately isolate the impacted endpoint via the EDR console if payload execution or credential entry is suspected.
* **Credential Reset:** Forcibly expire active sessions and trigger an emergency password reset if the final page was a credential harvester.
* **Network-Layer Controls:**
  * **Do Not** block the serving IP (`104.21.71.88`) as it is a shared Cloudflare CDN resource and will cause widespread business disruption.
  * **Do** block the specific malicious subdomain layer explicitly on your Secure Web Gateway (SWG) or DNS firewall (e.g., block `uvgei2qcvavk[.]pages[.]dev`).

### Phase 4: Post-Incident Activity
1. **Threat Takedown:** Submit an abuse report to the cloud provider hosting the redirect staging page (e.g., [Cloudflare Abuse Reporting](https://cloudflare.com)).
2. **Review Lessons Learned:** Determine if Secure Email Gateway (SEG) content filters should be updated to strictly hold Google Form receipts containing external hyperlinks for manual SOC approval.

