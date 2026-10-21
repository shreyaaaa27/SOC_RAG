# Live Demo – Alert Assessment

> This is the unedited output of the two-agent live demo, from the notebook run of 21 Oct 2026 on `gemini/gemini-3.5-flash-lite`. The alert is **hypothetical**. For comparison, [`first_run_live_demo_corrected.md`](first_run_live_demo_corrected.md) shows the first run's answer: it treated execution as confirmed and relied on a weak retrieval match (0.28).
>
> In this run, retrieval returned T1571 Non-Standard Port (0.38) and T1566.001 Spearphishing Attachment (0.63). Both documents were added to the knowledge base after the first run.

---

**Alert:** Employee laptop 10.0.0.42 downloaded 'invoice_oct.pdf.exe' from an email attachment; shortly after, the host started connecting every 30 seconds to 77.91.124.5 on port 8080.

---

### Assessment

### Observed Facts
* An employee laptop (IP address `10.0.0.42`) downloaded a file named `invoice_oct.pdf.exe` from an email attachment.
* Shortly after the download, the host initiated network connections every 30 seconds to the external IP address `77.91.124.5` on port `8080`.

### Hypotheses
* The downloaded file `invoice_oct.pdf.exe` may have been executed by the user or automatically, given the executable extension disguised with a double extension.
* The regular 30-second connection intervals to `77.91.124.5` on port `8080` suggest that a process on the host (potentially the downloaded executable) may be beaconing to a command-and-control server.
* Data may have been exfiltrated or instructions may have been retrieved during these periodic network connections, though the content and protocol of the traffic are not shown in the logs.

### Candidate MITRE Techniques
* **T1566.001 (Phishing: Spearphishing Attachment)** – *Direct*. The alert explicitly states that the host downloaded a file with a suspicious double extension (`invoice_oct.pdf.exe`) via an email attachment.
* **T1571 (Non-Standard Port)** – *Tentative*. The host connects to an external IP on port `8080`, which is often used for alternative web traffic or command-and-control, but the specific application-layer protocol has not been verified from packet or proxy logs.
* **T1071 (Application Layer Protocol)** – *Tentative*. The regular 30-second connection interval suggests potential command-and-control beaconing, but the actual protocol used is not confirmed by the observed facts.

### Severity & Justification
**Severity:** Medium  
**Justification:** The download of a disguised executable attachment combined with subsequent periodic outbound connections to an external IP address indicates suspicious activity consistent with malware installation and potential command-and-control beaconing. However, because endpoint execution telemetry and payload analysis are absent, full compromise cannot be confirmed, warranting a Medium severity rating pending further investigation.

---

### Recommended response

### Severity
**Severity:** Medium  
**Justification:** The download of a disguised executable attachment combined with subsequent periodic outbound connections to an external IP address indicates suspicious activity consistent with malware installation and potential command-and-control beaconing. However, because endpoint execution telemetry and payload analysis are absent, full compromise cannot be confirmed, warranting a Medium severity rating pending further investigation.

---

### Verify First
* **Confirm execution with EDR/process logs:** Query endpoint telemetry (EDR) or process creation logs on `10.0.0.42` to determine whether `invoice_oct.pdf.exe` was actually executed by the user or any automated process.
* **Identify the protocol from proxy/firewall logs:** Inspect network proxy, firewall, or packet capture logs for connections from `10.0.0.42` to `77.91.124.5` on port `8080` to confirm the application-layer protocol and assess whether data transmission occurred.
* **Inspect email gateway telemetry:** Review email server or gateway logs to verify the delivery of the email containing `invoice_oct.pdf.exe` and identify any other recipients across the organisation who received the same message.

---

### Recommended Containment
* **Isolate the host upon execution confirmation:** If EDR logs confirm the execution of `invoice_oct.pdf.exe` or suspicious child processes, recommend immediate network isolation of host `10.0.0.42` while keeping the machine powered on for forensic analysis.
* **Block external indicators of compromise:** Add external IP address `77.91.124.5` and file hash/name indicators to perimeter firewalls, web proxies, and EDR blocklists to prevent further communication.
* **Revoke active sessions and reset credentials:** Reset the impacted user's account password and terminate active authentication tokens or sessions to mitigate potential credential misuse.
* **Block and purge phishing emails:** Search email mailboxes for the specific phishing message containing the malicious attachment and purge or delete matching emails organisation-wide.

---

### Follow-Up
* **Perform forensic analysis:** Acquire memory and disk images from host `10.0.0.42` to analyze the downloaded payload, identify the malware family, and determine the exact scope of activity.
* **Conduct enterprise threat hunting:** Search all endpoint and network logs across the organisation for indicators associated with `invoice_oct.pdf.exe` and connections to `77.91.124.5`.
* **Remediate and restore:** Re-image host `10.0.0.42` following forensic data collection, verify system integrity, and provide targeted security awareness training to the affected employee.
---

### Reviewer notes
- The answer separates observed facts from hypotheses. It treats execution and exfiltration as unconfirmed, marks T1571 and T1071 as tentative, and puts verification steps before containment.
- **T1566.001 is marked "Direct".** That is reasonable, since delivery by email attachment is stated in the alert, but the file has not been confirmed as malicious.
- **Credential reset is unconditional.** The containment step recommends resetting credentials without first checking whether credentials were exposed.
- **Severity is repeated.** The response step repeats the assessment's severity section word for word.
