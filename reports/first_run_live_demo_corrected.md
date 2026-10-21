# Live Demo – Corrected Assessment of a New Alert

> This is the two-agent live demo from notebook §12, run on 20 Oct 2026. The alert is **hypothetical**. Below is a manually corrected version of the agents' answer, followed by what was changed and why.

**Alert text**
> Employee laptop 10.0.0.42 downloaded `invoice_oct.pdf.exe` from an email attachment; shortly after, the host started connecting every 30 seconds to 77.91.124.5 on port 8080.

## Observed facts (stated in the alert)
- The file `invoice_oct.pdf.exe` was downloaded on 10.0.0.42 from an email attachment. Its double extension disguises an executable as a PDF.
- Shortly afterwards, the host began connecting to 77.91.124.5 on port 8080 every 30 seconds.

## Hypotheses (not shown by the alert)
- **Execution.** The file was opened or run. The alert reports a download, not a process start.
- **Causation.** The connections were caused by the file. The alert gives timing only.
- **Protocol.** The traffic was HTTP. Port 8080 is often used for HTTP, but the alert does not show the protocol.
- **Exfiltration.** Data left the host. The alert gives no transfer sizes or content.

## Candidate MITRE ATT&CK techniques

| Technique | Confidence | Reason |
| :-- | :-- | :-- |
| T1036.007 Masquerading: Double File Extension | Direct | The filename itself is observed. |
| T1566.001 Phishing: Spearphishing Attachment | Tentative | Delivery by email attachment is stated, but the file has not been confirmed as malicious. |
| T1204.002 User Execution: Malicious File | Tentative | Execution is not shown. |
| T1071.001 Application Layer Protocol: Web Protocols | Tentative | The protocol is not observed. |
| T1571 Non-Standard Port | Tentative | Port 8080 is also common for legitimate proxies. |
| Exfiltration (any) | Not mapped | There is no evidence of data transfer. |

## Severity
**High (provisional).** A disguised executable from email, followed closely by regular outbound connections, is a strong reason to investigate urgently. The severity should be raised or lowered once the checks below are done.

## Recommended response (not executed)
**Verify first**
1. Check EDR or process-creation logs (for example Sysmon Event ID 1) for execution of `invoice_oct.pdf.exe`, and get its hash.
2. Check proxy and firewall logs for the protocol, URLs and byte counts of the 8080 traffic, and look up the reputation of 77.91.124.5.
3. Find other recipients of the same email.

**Containment.** Isolate the host immediately as a precaution if policy allows; otherwise, isolate it once execution is confirmed.
1. Isolate 10.0.0.42 through EDR while keeping it powered on.
2. Block 77.91.124.5 at the firewall and proxy.
3. Quarantine the email for every recipient.
4. Reset the user's credentials only if there is evidence they were exposed.

**Follow-up**
1. Collect memory and disk images if execution is confirmed.
2. Hunt for the file hash and destination IP across other endpoints.
3. Re-image the laptop if compromise is confirmed, and give the user awareness training.

## What was corrected from the agents' original answer
- The Threat Intelligence agent said the file's "execution ... represents a successful initial access and execution vector" and that the endpoint was "actively compromised". Both overstate the alert, which reports a download and network timing only.
- The agent mapped the alert to **T1071 Application Layer Protocol** as a firm finding. The protocol is not observed, so this is now tentative.
- Its top retrieved document was **T1048 Exfiltration** at a weak similarity of 0.28. The T1071 answer therefore came from the model's own knowledge, not from retrieval. The knowledge base had no document on email attachments, user execution or non-standard ports.
  - **Fix:** four documents were added (T1566.001, T1204.002, T1571, T1041).
  - **Fix:** the demo prompt now asks for facts and hypotheses separately, with a confidence label on each mapping.
  - T1036.007 and T1071.001 in the table above come from manual review; they are not in the knowledge base.
- The Incident Responder's first actions assumed compromise, for example "Reset the compromised user account's password". Verification steps now come first.
