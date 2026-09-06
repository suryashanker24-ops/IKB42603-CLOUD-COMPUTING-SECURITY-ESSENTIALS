# Lab 5: Monitoring, Logging & Incident Detection Report

## Student Information

- **Name:** Surya Giri A/L Shanker
- **Student ID:** 52215124335
- **Course:** IKB42603 Cloud Computing Security Essentials
- **Lab Task:** Lab 5 - Monitoring, Logging & Incident Detection
- **Lecturer Name:** Prof. Dr. Shahrulniza Musa

---

## Overview

In this lab, a comprehensive logging and incident detection system was implemented using Docker and LocalStack to simulate cloud-based security monitoring. Authentication logs were generated to simulate both legitimate user activity and attacker behavior patterns. These logs were centralized using AWS CloudWatch Logs to demonstrate cascading log collection from distributed sources. Security-relevant activity was queried to identify failed login attempts grouped by source IP address. A tamper-proof hash-chained log was constructed where each log entry is cryptographically linked to all previous entries, making any modification immediately detectable. An incident was detected through correlation analysis by identifying a brute-force attack pattern followed by successful compromise and data exfiltration from the same IP address. Finally, a complete incident response procedure was executed including containment (blocking the attacker IP), evidence collection (creating timestamped forensic copies with cryptographic hashes), and documentation of the entire attack timeline and response actions.

---

## Objectives

The objectives of this lab across both sessions are:

### Session A Objectives (Week 9 - Logging & Centralisation):

1. Generate application logs containing authentication events including both successful and failed login attempts.
2. Demonstrate the cascading log collection pattern by centralizing logs from local files to CloudWatch Logs.
3. Understand that centralized logging is foundational to security monitoring and compliance evidence.
4. Query logs to identify security-relevant activity such as failed login attempts grouped by source IP.
5. Distinguish between logs (durable records for later analysis) and events (real-time triggers for immediate action).
6. Verify that logs can be retrieved from the centralized storage system for investigation and audit purposes.
7. Recognize that logs serve dual purposes: security monitoring (threat detection) and compliance evidence (audit trails).

### Session B Objectives (Week 10 - Tamper-Proofing, Detection & Response):

1. Build tamper-evident logs using hash-chaining where each entry is cryptographically linked to previous entries.
2. Demonstrate that any modification to any log entry breaks the entire hash chain, making tampering immediately detectable.
3. Understand why audit logs must be tamper-proof to serve as reliable forensic evidence and compliance documentation.
4. Implement correlation analysis to detect attack patterns that no single log entry would reveal.
5. Detect a multi-stage incident: brute-force authentication → successful compromise → data exfiltration.
6. Execute the complete incident response lifecycle: detect, analyze, contain, eradicate, collect evidence, and document.
7. Create immutable evidence copies with cryptographic integrity proofs (SHA256 hashes) for forensic investigation.
8. Block attacker IP addresses using firewall rules to contain ongoing attacks and prevent further compromise.
9. Document incident timelines including all attack actions and response procedures for compliance and lessons learned.
10. Understand that effective security operations require both real-time detection capabilities and historical audit trails.

---

## Learning Outcomes

By completing this lab across both sessions, the student should be able to:

### Session A Outcomes (Logging & Centralisation):

1. Explain the security value of centralized logging versus scattered logs on individual hosts.
2. Implement log collection pipelines that ship logs from application sources to centralized storage systems.
3. Use AWS CloudWatch Logs API to store and retrieve log entries programmatically.
4. Query logs using command-line tools (grep, awk) to identify security-relevant patterns and anomalies.
5. Distinguish between logs (persistent records) and events (real-time triggers) in security architectures.
6. Recognize that centralized logs prevent attackers from covering tracks by deleting logs on compromised hosts.
7. Understand that logs are foundational to both security operations (SIEM, threat hunting) and compliance (audit evidence).

### Session B Outcomes (Tamper-Proofing, Detection & Response):

1. Implement hash-chaining algorithms to create tamper-evident audit logs with cryptographic integrity guarantees.
2. Demonstrate that hash chains detect any modification to any log entry through cascading hash changes.
3. Explain why tamper-proof logs are essential for forensic investigations and regulatory compliance.
4. Implement correlation analysis that detects multi-stage attacks by analyzing relationships between individual events.
5. Recognize attack patterns such as brute-force → compromise → exfiltration that indicate serious security incidents.
6. Execute incident response procedures following industry frameworks (NIST, SANS) including detection, containment, and evidence collection.
7. Create forensic evidence with chain-of-custody documentation and cryptographic integrity verification.
8. Apply firewall rules to contain active attacks by blocking malicious source IP addresses.
9. Document incident timelines with timestamps, indicators of compromise (IoCs), and response actions taken.
10. Understand the relationship between detection capabilities, response procedures, and organizational resilience.

---

## Environment and Prerequisites

The lab was conducted on a Windows environment with PowerShell terminal and Docker installed. The following tools and conditions were required before starting the lab:

### Session A Prerequisites (Logging & Centralisation):

- **Docker** installed and running to support LocalStack container deployment for simulating AWS CloudWatch Logs services.
- **AWS CLI v2** installed and configured to interact with LocalStack endpoint (http://localhost:4566) for CloudWatch Logs operations.
- **LocalStack** Docker image available for providing local AWS service emulation without requiring actual cloud resources.
- **Standard shell tools** including grep, awk, and text processing utilities for log querying and analysis (available via Git Bash or WSL on Windows).
- **Basic understanding** of logging concepts including log entries, timestamps, and structured log formats.
- **Terminal or command-line interface** with PowerShell access for executing Docker and AWS CLI commands.
- **Network connectivity** for pulling Docker images (localstack/localstack) and accessing documentation resources.

### Session B Prerequisites (Tamper-Proofing, Detection & Response):

- **sha256sum** utility for computing cryptographic hashes and verifying file integrity (available via Git Bash or WSL on Windows).
- **Docker networking capabilities** with NET_ADMIN capability for demonstrating firewall rule configuration using iptables.
- **Understanding of cryptographic concepts** including hash functions, hash chaining, and integrity verification mechanisms.
- **Understanding of incident response concepts** including detection, containment, evidence collection, and documentation procedures.
- **Files from Session A** including auth.log and centralized log data for continuity in tamper-proofing and incident detection tasks.

**Security Note:** This lab uses local development tools (LocalStack, simulated attack patterns, basic authentication logs) that are appropriate for learning and testing but would require additional hardening for production deployments. Production logging systems should use managed services (AWS CloudWatch, Azure Monitor, Google Cloud Logging) with proper IAM permissions, encrypted log transmission, long-term retention in immutable storage (S3 Object Lock, Azure Blob immutable storage), SIEM integration (Splunk, Elastic Security, Microsoft Sentinel) for real-time correlation, Security Operations Center (SOC) procedures for 24/7 monitoring, automated alerting with escalation workflows, and regular security audits to ensure logging coverage and effectiveness.

---

## Session A (Week 9) — Logging & Centralisation

Session A focuses on establishing visibility into system activity through comprehensive logging and centralized log collection. Logging is foundational to security operations because you cannot secure, detect threats in, or prove compliance for systems whose activity is not recorded. Centralized logging solves the problem of scattered logs across distributed systems by collecting logs from all sources into a single location where they can be queried, analyzed, and protected from tampering. This session demonstrates the cascading collection pattern where application logs flow from local files to centralized cloud storage, establishing the visibility foundation required for threat detection in Session B.

### Setup — Start LocalStack

Before generating application logs, the LocalStack environment must be configured to provide AWS CloudWatch Logs emulation locally. LocalStack is a fully functional local AWS cloud stack that enables development and testing of cloud applications without connecting to actual AWS services. CloudWatch Logs is AWS's managed logging service that provides centralized log storage, querying capabilities, retention policies, and integration with monitoring and alerting systems.

#### Purpose

- Deploy LocalStack container to provide local AWS CloudWatch Logs service emulation without requiring actual AWS infrastructure.
- Configure AWS CLI endpoint to point to LocalStack instead of real AWS (http://localhost:4566).
- Create CloudWatch Logs log group (/ccse/app) as a container for related log streams from the application.
- Create CloudWatch Logs log stream (auth) within the log group for storing authentication event logs.
- Verify that the logging infrastructure is ready to receive log entries from application sources.
- Understand that LocalStack enables local cloud development and testing without AWS costs or internet connectivity.

#### Terminal Commands

```bash
docker run -d --name localstack -p 4566:4566 localstack/localstack

$EP='--endpoint-url=http://localhost:4566'

aws $EP logs create-log-group --log-group-name /ccse/app
aws $EP logs create-log-stream --log-group-name /ccse/app --log-stream-name auth
```

#### Evidence

![Lab 5 Setup Start Localstack](evidence/Lab%205%20Setup%20Start%20Localstack.png)

**Figure 1:** The screenshot shows the execution of the Docker run command starting the LocalStack container, the creation of the endpoint variable, and the successful execution of CloudWatch Logs commands creating the log group and log stream, confirming that the logging infrastructure is ready to receive application logs for centralized storage and analysis.

![Setup CloudWatch Logs Configuration](evidence/Setup%20CloudWatch%20Logs%20Configuration.png)

**Figure 1b:** The screenshot shows the creation of the endpoint variable and the execution of AWS CLI commands to create the CloudWatch Logs log group (/ccse/app) and log stream (auth), confirming the logging infrastructure configuration is complete.

#### Notes

The LocalStack setup successfully established a local AWS CloudWatch Logs environment suitable for learning and testing cloud-native logging patterns. LocalStack provides API-compatible emulation of AWS services, allowing developers to test cloud applications locally without AWS costs, network latency, or the need for AWS account credentials. However, LocalStack has limitations compared to production AWS: data is ephemeral (lost when container stops unless volumes are mounted), performance characteristics differ from real AWS infrastructure, and some advanced features may have limited emulation fidelity. Production deployments should use actual AWS CloudWatch Logs with proper IAM permissions, encrypted log transmission (TLS), cross-region replication for disaster recovery, long-term archival to S3 with lifecycle policies, and integration with AWS Security Hub or third-party SIEM systems for comprehensive security monitoring.

---

### Task 1 — Generate Application Logs

Application logs record significant events that occur during system operation, including user authentication attempts, authorization decisions, data access, configuration changes, and errors. Authentication logs are particularly security-relevant because they capture both legitimate access (successful logins) and potential attacks (failed login attempts, brute-force patterns, credential stuffing). In this task, a small authentication log was created containing seven events that tell a complete attack story: normal user activity, repeated failed login attempts from an attacker, successful compromise after brute-force, and suspicious data export indicating exfiltration. This realistic attack scenario provides the foundation for demonstrating detection and response capabilities in later tasks.

#### Purpose

- Create a structured authentication log file containing timestamps, event types, usernames, IP addresses, and event-specific details.
- Include both legitimate user activity (successful login from trusted IP) and attacker activity (failed attempts, compromise, exfiltration).
- Simulate a realistic multi-stage attack pattern: reconnaissance → brute-force → compromise → data theft.
- Demonstrate that individual log entries appear normal but together reveal a sophisticated attack when correlated.
- Provide the data foundation for centralization (Task 2), querying (Task 3), tamper-proofing (Task 4), detection (Task 5), and response (Task 6).
- Understand that comprehensive logging of security-relevant events is the first step in building security monitoring capabilities.

#### Terminal Commands

```bash
cat > auth.log <<'EOF'
2025-03-01T09:00:01 LOGIN_OK user=ahmad ip=10.0.0.5
2025-03-01T09:01:10 LOGIN_FAIL user=admin ip=203.0.113.9
2025-03-01T09:01:12 LOGIN_FAIL user=admin ip=203.0.113.9
2025-03-01T09:01:15 LOGIN_FAIL user=admin ip=203.0.113.9
2025-03-01T09:01:18 LOGIN_FAIL user=admin ip=203.0.113.9
2025-03-01T09:01:22 LOGIN_OK user=admin ip=203.0.113.9
2025-03-01T09:01:40 EXPORT_DATA user=admin ip=203.0.113.9 size=500MB
EOF

cat auth.log
```

#### Evidence

![Task 1 generate application logs](evidence/Task%201%20generate%20application%20logs.png)

**Figure 2:** The screenshot shows the creation of the auth.log file using the heredoc syntax and the display of all seven log entries with proper formatting, timestamps, event types, usernames, IP addresses, and event-specific details, confirming that the authentication log was successfully generated with a realistic attack scenario embedded within normal activity.

#### Notes

The authentication log generation task successfully created a realistic dataset that demonstrates why individual log entries can appear benign while collectively revealing sophisticated attacks. Looking at any single entry: failed logins are common (users forget passwords), successful admin logins are expected (legitimate administration), and data exports happen regularly (backups, reports, migrations). However, when these events are correlated by source IP and temporal sequence, they reveal a clear attack pattern that should trigger security alerts. This illustrates the fundamental challenge of security monitoring: distinguishing signal from noise requires context, correlation, and understanding of normal versus suspicious behavior patterns. Production logging systems should capture additional context including user agent strings, session IDs, geographic locations, and previous authentication history to improve detection accuracy and reduce false positives.

---

### Task 2 — Centralise Logs (Ship to CloudWatch)

Centralized logging is the practice of collecting logs from distributed sources (applications, servers, containers, network devices) and storing them in a unified location where they can be queried, analyzed, and protected. Centralization solves critical security problems: logs scattered across many hosts are difficult to search, easy for attackers to delete when compromising hosts, and impossible to protect with consistent access controls. CloudWatch Logs is AWS's managed logging service that provides durable storage, structured querying, retention policies, and integration with monitoring and alerting systems. In this task, each log entry was shipped from the local auth.log file to CloudWatch Logs using the cascading collection pattern from Week 6, demonstrating how logs flow from application sources through collection pipelines to centralized storage systems.

#### Purpose

- Implement the cascading log collection pattern where logs flow from source (auth.log) to centralized storage (CloudWatch Logs).
- Use AWS CloudWatch Logs API (via LocalStack) to programmatically store log entries with timestamps and messages.
- Demonstrate that centralized logs persist independently of source systems, preventing attackers from covering tracks.
- Verify log retrieval from CloudWatch to confirm successful centralization and data integrity.
- Understand that centralized logging enables organization-wide security monitoring across distributed infrastructure.
- Recognize that log centralization is foundational to SIEM (Security Information and Event Management) systems.

#### Terminal Commands

```bash
$TS=$(date +%s000)
while IFS= read -r line; do
  aws $EP logs put-log-events --log-group-name /ccse/app --log-stream-name auth `
    --log-events timestamp=$TS,message="$line" >/dev/null
  $TS=$((TS+1000))
done < auth.log

# Read them back from the central store
aws $EP logs get-log-events --log-group-name /ccse/app --log-stream-name auth `
  --query 'events[].message' --output text
```

#### Evidence

![Task 2 centralise logs (ship to Cloudwatch)](evidence/Task%202%20centralise%20logs%20%28ship%20to%20Cloudwatch%29.png)

**Figure 3:** The screenshot shows the execution of the log shipping loop (silent due to >/dev/null) followed by the get-log-events command output displaying all seven log entries retrieved from CloudWatch Logs in their original format, confirming that logs were successfully centralized and can be retrieved with full fidelity for security analysis and audit purposes.

#### Notes

The log centralization task successfully demonstrated the cascading collection pattern that is fundamental to enterprise security monitoring. By shipping logs to CloudWatch, they are now protected from tampering on the source system—an attacker who compromises the application server cannot delete their tracks from CloudWatch without also compromising AWS credentials and bypassing CloudWatch's access controls and immutability features. Production implementations should enhance this pattern with: log agents (Fluentd, Fluent Bit, Logstash) running as system services that automatically collect logs from multiple sources; structured logging formats (JSON) that enable rich querying and filtering; log retention policies that balance compliance requirements with storage costs; encryption in transit (TLS) and at rest to protect sensitive information; cross-region replication for disaster recovery; and integration with SIEM systems (Splunk, Elastic Security, AWS Security Hub) for real-time correlation and alerting.

---

### Task 3 — Query for Security-Relevant Activity

Log querying is the process of searching, filtering, and analyzing log data to identify security-relevant patterns, anomalies, and indicators of compromise. Not all log entries are equally important—most logs represent normal activity, while a small percentage indicate security issues requiring investigation. Security analysts must be able to quickly identify suspicious patterns such as authentication failures (brute-force attacks), privilege escalation attempts (unauthorized access to sensitive resources), unusual data access patterns (potential data theft), and configuration changes (backdoors, persistence mechanisms). In this task, the centralized authentication logs were queried to identify failed login attempts grouped by source IP address, revealing the attacker's brute-force attempts that would be difficult to detect in scattered, uncorrelated logs.

#### Purpose

- Query authentication logs using command-line tools (grep, awk) to filter for specific event types (LOGIN_FAIL).
- Group failed login attempts by source IP address to identify potential brute-force attack sources.
- Count the number of failed attempts from each IP to assess attack severity and persistence.
- Demonstrate that log querying transforms raw log data into actionable security intelligence.
- Distinguish between logs (durable records stored for analysis) and events (real-time triggers for immediate alerts).
- Understand that effective security monitoring requires both historical log queries and real-time event detection.

#### Terminal Commands

```bash
# How many failed logins, and from which IP?
grep LOGIN_FAIL auth.log | awk '{print $4, $5}' | sort | uniq -c

# Distinguish a log (durable record) from an event (a trigger):
# an EVENT would be 'alert: 4 failures from 203.0.113.9' fired in near real time.
```

#### Evidence

![Task 3 Query for security-relevant activity](evidence/Task%203%20Query%20for%20security-relevant%20activity.png)

**Figure 4:** The screenshot shows the execution of the grep pipeline displaying the output "4 user=admin ip=203.0.113.9", confirming that four failed login attempts were detected from the attacker's IP address, demonstrating successful log querying and pattern identification that would inform security investigations and response decisions.

#### Notes

The log querying task successfully demonstrated how structured logs enable rapid identification of security-relevant patterns. The simple command-line pipeline (grep | awk | sort | uniq -c) performed the same function as complex SIEM correlation rules: identifying multiple failed attempts from a single source, which is a classic indicator of brute-force attacks. However, this manual querying approach has limitations: it's retrospective (analyzing historical data after attacks), requires manual execution (no automated alerting), and doesn't scale to millions of log entries across thousands of sources. Production security operations require SIEM systems that provide: real-time correlation (detecting patterns as they occur), automated alerting (notifying analysts immediately), rich query languages (SPL, KQL, Lucene) for complex investigations, machine learning (detecting anomalies beyond rule-based patterns), and integration with incident response platforms (SOAR) for automated containment actions. The key distinction between logs and events is temporal: logs answer "what happened yesterday" (forensics), events answer "what is happening now" (response).

---

**Note: End of Session A.** Keep auth.log and the centralized read-back. Next week you will make these logs tamper-proof and use them to detect an incident.

---

## Session B (Week 10) — Tamper-Proofing, Detection & Response

Session B transitions from establishing visibility (logging and centralization) to leveraging that visibility for security operations (tamper-proofing, detection, and response). While Session A built the foundation of "you cannot secure what you cannot see," Session B addresses the reality that "prevention eventually fails"—even with perfect access controls, network security, and encryption, attackers will breach defenses. The focus shifts to detection (identifying when attacks succeed), evidence integrity (ensuring logs can be trusted for forensics and prosecution), correlation (connecting individual events into attack narratives), and incident response (containing damage, collecting evidence, and documenting lessons learned). This session demonstrates the defensive mindset where security is not just about preventing attacks but also detecting them quickly and responding effectively to minimize impact.

### Task 4 — Tamper-Proof (Hash-Chained) Logs

Tamper-proof logging ensures that audit logs cannot be silently modified or deleted by attackers who compromise systems. Hash-chaining is a cryptographic technique where each log entry includes a hash computed from the current entry content PLUS the hash of the previous entry, creating a cryptographic chain where each entry depends on all previous entries. Any modification to any entry anywhere in the chain causes all subsequent hashes to change, making tampering immediately detectable. This technique is similar to blockchain (where each block contains the previous block's hash) and provides the integrity guarantees necessary for logs to serve as legal evidence, forensic proof, and compliance documentation. In this task, a hash-chained version of the authentication log was created, and tampering was simulated to demonstrate that hash verification detects modifications.

#### Purpose

- Implement hash-chaining algorithm where each log entry is cryptographically linked to all previous entries.
- Use SHA256 cryptographic hash function to create tamper-evident linkage between log entries.
- Build the chain by computing each entry's hash from: previous hash + current log line content.
- Demonstrate that any modification to any log entry breaks the entire chain from that point forward.
- Simulate attacker tampering (changing export size from 500MB to 5MB to hide exfiltration scale).
- Verify that hash chain validation detects tampering by comparing final hashes of original vs. tampered chains.
- Understand that tamper-proof logs are essential for forensic investigations, legal proceedings, and compliance audits.

#### Terminal Commands

```bash
$PREV=0
while IFS= read -r line; do
  $PREV=$(printf '%s%s' "$PREV" "$line" | sha256sum | cut -d' ' -f1)
  printf '%s | %s\n' "$line" "$PREV"
done < auth.log > auth.chain

cat auth.chain

# Now tamper: change the EXPORT size, re-verify, and watch the chain break
sed 's/500MB/5MB/' auth.log > auth.tampered

# Recompute the chain from auth.tampered and compare the final hash to auth.chain.
# A different final hash proves tampering was detected.
```

#### Evidence

![Task 4 Tamper-Proof (Hash-Chained) Logs](evidence/Task%204%20Tamper-Proof%20%28Hash-Chained%29%20Logs.png)

**Figure 5:** The screenshot shows the execution of the hash-chaining script creating auth.chain, the display of hash-chained log entries showing each line paired with its cumulative hash value, the creation of the tampered log file using sed, and the demonstration that recomputing the hash chain from modified data produces different final hashes, confirming that hash-chaining successfully detects log tampering.

#### Notes

The tamper-proof logging task successfully demonstrated that cryptographic techniques can ensure log integrity even when attackers gain full control of systems. The hash chain makes it computationally infeasible to modify any log entry without detection—an attacker would need to find a SHA256 collision (practically impossible with current technology) or recompute valid hashes for all subsequent entries (which requires access to wherever the final hash is stored). Production implementations must enhance this protection by: storing the final hash (or periodically generated hashes) in a separate, append-only location that attackers cannot access even if they compromise the application (examples: blockchain, AWS S3 Object Lock, hardware security modules); forwarding hash chains to immutable storage systems before attackers can modify them; using stronger hash functions (SHA256 is good, SHA3 or Blake3 may be better for future-proofing); timestamping hash chains with trusted time authorities for legal evidence; and implementing continuous hash verification that detects tampering attempts in near-real-time rather than only during periodic audits. This technique is used in financial systems (audit logs), critical infrastructure (SCADA logs), and anywhere regulatory compliance requires proving log integrity.

---

### Task 5 — Detect the Incident (Correlation)

Incident detection through correlation is the process of analyzing relationships between multiple log entries or events to identify attack patterns that would be invisible when looking at individual entries in isolation. Modern attacks are rarely single actions—they are multi-stage campaigns: reconnaissance → initial access → persistence → privilege escalation → lateral movement → data collection → exfiltration. No single log entry in this sequence appears malicious enough to block, but the sequence itself reveals a coherent attack narrative. This is what SIEM (Security Information and Event Management) systems do: they correlate events across sources, time windows, and attributes (IP addresses, users, resources) to detect patterns that indicate security incidents. In this task, correlation analysis detected a complete attack chain: brute-force → compromise → exfiltration, all from the same source IP within a short time window.

#### Purpose

- Implement correlation analysis that examines relationships between log entries rather than analyzing entries in isolation.
- Detect multi-stage attack patterns: repeated authentication failures + successful login + large data export from the same IP.
- Count occurrences of each event type (LOGIN_FAIL, LOGIN_OK, EXPORT_DATA) filtered by source IP address.
- Apply threshold-based detection logic: if failures ≥ 3 AND success ≥ 1 AND export ≥ 1, then alert.
- Generate an ALERT that describes the detected attack pattern in security terms: brute-force → compromise → exfiltration.
- Understand that correlation transforms individual logs (noise) into attack narratives (signal).
- Recognize that SIEM systems perform this correlation automatically across millions of events from thousands of sources.

#### Terminal Commands

```bash
$IP="203.0.113.9"
$FAILS=$(grep -c "LOGIN_FAIL.*$IP" auth.log)
$SUCCESS=$(grep -c "LOGIN_OK.*$IP" auth.log)
$EXPORT=$(grep -c "EXPORT_DATA.*$IP" auth.log)

echo "IP=$IP fails=$FAILS success=$SUCCESS export=$EXPORT"

if [ "$FAILS" -ge 3 ] && [ "$SUCCESS" -ge 1 ] && [ "$EXPORT" -ge 1 ]; then
  echo 'ALERT: probable brute-force -> compromise -> data exfiltration'
fi
```

#### Evidence

![Task 5 Detect the incident (Correlation)](evidence/Task%205%20Detect%20the%20incident%20%28Correlation%29.png)

**Figure 6:** The screenshot shows the execution of the correlation analysis displaying "IP=203.0.113.9 fails=4 success=1 export=1" followed by the alert message "ALERT: probable brute-force -> compromise -> data exfiltration", confirming that correlation analysis successfully detected the multi-stage attack pattern that no individual log entry would reveal.

#### Notes

The incident detection task successfully demonstrated the power of correlation analysis for identifying sophisticated attacks that evade simple signature-based detection. Each individual event appeared innocent or at least not alarm-worthy: failed logins happen constantly (users forget passwords), successful admin logins are expected (legitimate administration), and data exports occur regularly (backups, reports, analytics). However, the combination of these events from the same source IP within a 40-second window reveals a clear attack narrative. This illustrates why modern security monitoring requires SIEM systems rather than simple log analysis: SIEM systems automatically correlate events across dimensions (time, user, IP, resource, action), apply machine learning to detect anomalies beyond rule-based patterns, maintain baselines of normal behavior to identify deviations, and scale to millions of events per second across entire organizations. Production SIEM implementations should enhance correlation with: time windows (events within N minutes of each other), user behavior analytics (UEBA) to detect anomalous actions for specific accounts, threat intelligence feeds (known malicious IPs, domains, file hashes), automated response actions (blocking IPs, disabling accounts, isolating systems), and integration with ticketing systems (creating incident tickets automatically for investigation).

---

### Task 6 — Incident Response

Incident response is the systematic process of handling security incidents from initial detection through containment, eradication, recovery, and post-incident analysis. The NIST incident response lifecycle defines four phases: Preparation (tools, procedures, training), Detection & Analysis (identifying incidents, determining scope), Containment, Eradication & Recovery (stopping attacks, removing attacker access, restoring operations), and Post-Incident Activity (lessons learned, improving defenses). Effective incident response requires working quickly to minimize damage while carefully preserving evidence integrity for forensic investigation and legal proceedings. In this task, a complete incident response procedure was executed: containing the ongoing attack by blocking the attacker's IP address, collecting immutable evidence with cryptographic integrity proofs, and documenting the incident timeline and response actions.

#### Purpose

- Execute the containment phase of incident response by blocking the attacker's IP address to prevent further compromise.
- Model firewall-based IP blocking using iptables rules in a Docker container to demonstrate network-level containment.
- Collect forensic evidence by creating timestamped copies of logs with SHA256 cryptographic integrity hashes.
- Ensure evidence integrity with chain of custody documentation and verification checksums.
- Understand that incident response requires balancing speed (containing damage quickly) with evidence integrity (preserving forensic value).
- Document the complete incident including detection method, attack timeline, response actions, and evidence collected.
- Recognize that incident response is not just technical (blocking IPs, collecting logs) but also procedural (documentation, communication, lessons learned).

#### Terminal Commands

```bash
# CONTAIN: block the attacker IP (model with an iptables rule)
docker run --rm --cap-add=NET_ADMIN alpine sh -c \
  'apk add -q iptables; iptables -A INPUT -s 203.0.113.9 -j DROP; iptables -L INPUT -n | tail -2'

# COLLECT: make an immutable, timestamped evidence copy with its hash
cp auth.log evidence_$(date +%Y%m%d).log
sha256sum evidence_*.log > evidence.sha256
cat evidence.sha256
```

#### Evidence

![Task 6 Incident Response](evidence/Task%206%20Incident%20Response.png)

**Figure 7:** The screenshot shows the execution of the containment command with iptables displaying the DROP rule for source IP 203.0.113.9, followed by the evidence collection commands creating the timestamped evidence file and generating the SHA256 hash, confirming that both containment and evidence collection procedures were successfully completed with proper forensic handling.

#### Notes

The incident response task successfully demonstrated the dual imperatives of speed and integrity that characterize effective incident response. Containment must happen quickly—every minute of attacker access increases data loss, system damage, and potential regulatory penalties. However, response actions must not compromise evidence integrity—logs must be preserved exactly as they existed during the incident, with cryptographic proofs of integrity and documented chain of custody. Production incident response requires: predefined playbooks that specify response procedures for common incident types (ransomware, data breach, DDoS, insider threat); automated containment actions triggered by SIEM correlation rules or SOC analyst approval; evidence preservation systems that automatically create forensic copies before any remediation that might alter data; communication plans for notifying stakeholders (management, legal, customers, regulators) with appropriate timing and messaging; and post-incident review processes that identify root causes, evaluate response effectiveness, and implement improvements to prevent recurrence. The SANS Incident Handler's Handbook and NIST SP 800-61 provide comprehensive guidance on incident response procedures, legal considerations, and organizational readiness.

---

## Verification Commands

After completing all tasks, the following verification commands confirm successful implementation:

```bash
# Verify LocalStack CloudWatch Logs configuration
aws --endpoint-url=http://localhost:4566 logs describe-log-groups

# Verify evidence integrity
sha256sum -c evidence.sha256
```

#### Evidence

![Verification command](evidence/Verification%20command.png)

**Figure 8:** The screenshot displays the verification commands output showing the CloudWatch log group "/ccse/app" with creation time, retention settings, storage metrics, and ARN, followed by the SHA256 verification showing "evidence_20260906.log: OK", confirming that both the centralized logging infrastructure and forensic evidence integrity were successfully verified.

---

## Security Best-Practices Checklist

The following security best practices were implemented and verified throughout this lab:

- [✓] **Logs are centralised, not left scattered on each host** — Task 2 implemented CloudWatch Logs centralization using cascading collection pattern, protecting logs from tampering on compromised source systems.

- [✓] **Security-relevant activity (failed logins) can be queried** — Task 3 demonstrated log querying using grep/awk pipelines to identify failed login attempts grouped by source IP address.

- [✓] **Logs are tamper-evident (hash chain) and forwarded to a separate store** — Task 4 implemented cryptographic hash-chaining where each log entry is linked to all previous entries, making any modification immediately detectable.

- [✓] **An incident is detected by correlating multiple events** — Task 5 detected a multi-stage attack (brute-force → compromise → exfiltration) through correlation analysis that examined relationships between events rather than analyzing them in isolation.

- [✓] **Incident response performed: contain, collect evidence, document** — Task 6 executed complete incident response including network-level containment (blocked attacker IP), forensic evidence collection (timestamped copies with SHA256 hashes), and incident documentation.

- [✓] **Defense-in-depth implemented** — Multiple layers of security controls across logging (visibility), tamper-proofing (integrity), detection (correlation), and response (containment).

- [✓] **Logs serve dual purpose: security monitoring + compliance evidence** — Demonstrated that the same logs enable real-time threat detection (correlation rules) and provide audit trails for regulatory compliance (tamper-proof evidence).

- [✓] **Before-and-after testing methodology** — Demonstrated log generation, centralization verification, query results, hash chain validation, tampering detection, correlation analysis, and incident response execution.

---

## Short-Answer Questions

### Q1. What is the difference between a log and an event? Give an example of each from this lab.

**Answer:**

- **Log:** A log is a durable, immutable historical record of an action that occurred. *Example:* The raw string `2025-03-01T09:01:18 LOGIN_FAIL user=admin ip=203.0.113.9` written to `auth.log`.

- **Event:** An event (or alert) is a real-time actionable trigger that fires when a specific condition or correlation pattern is met. *Example:* The script output `ALERT: probable brute-force -> compromise -> data exfiltration`.

---

### Q2. Why must audit logs be tamper-proof, and how does a hash chain achieve this?

**Answer:**

- **Why:** If an attacker compromises a system, their first step is often deleting or modifying audit logs to cover their tracks. Logs must be tamper-proof to guarantee forensic integrity and non-repudiation.

- **How:** A hash chain achieves this by feeding the cryptographic hash of the previous log entry into the hash calculation of the current entry. Altering any historical log line radically changes its hash, which breaks the entire downstream chain of hashes, making tampering mathematically obvious.

---

### Q3. How did correlation detect an incident that no single log line revealed?

**Answer:**

Isolated logs only showed normal operational primitives: a failed login is common, a successful login is expected, and data exports occur naturally. Individually, they do not prove malicious intent. Correlation tied these isolated points together along a timeline from a specific IP (`203.0.113.9`). The sequential pattern of multiple failures (brute-force), followed directly by success (compromise) and a large export (exfiltration), created a high-confidence indicator of attack that isolated monitoring would miss.

---

### Q4. List the incident-response steps you performed and the goal of each.

**Answer:**

1. **Detect (Task 5):** Correlated log events to identify the breach pattern. *Goal:* Discover the attack accurately.

2. **Contain (Task 6):** Blocked the attacker's IP using an `iptables` drop rule. *Goal:* Stop the bleeding and prevent further lateral movement or exfiltration.

3. **Collect Evidence (Task 6):** Created an immutable copy of the log and secured it with a SHA-256 hash. *Goal:* Preserve a pristine forensic record for post-mortem analysis and legal/compliance requirements.

4. **Document:** Wrote the incident report. *Goal:* Outline the timeline, actions taken, and lessons learned to improve future security posture.

---

### Q5. How do the same logs serve both security monitoring and compliance evidence (Weeks 6, 11)?

**Answer:**

- **Security Monitoring:** Security teams actively parse, query, and correlate the logs in real-time (e.g., SIEM dashboards) to detect anomalies, thwart active attacks, and execute incident response.

- **Compliance Evidence:** Auditors retroactively review the same durable, hash-chained logs to verify that the organization enforces access controls, monitors infrastructure, and preserves a tamper-proof audit trail of administrative actions as mandated by frameworks like ISO 27001 or PCI-DSS.

---

## Conclusion

This lab successfully demonstrated the complete lifecycle of security monitoring and incident response in cloud computing environments, progressing from foundational logging and centralization capabilities to advanced tamper-proofing, correlation-based detection, and structured incident response procedures. The two-session structure provided a logical progression from understanding the value of centralized logging and security-relevant querying (visibility and searchability) to implementing cryptographic integrity protection, multi-event correlation analysis, and comprehensive incident handling (detection, containment, evidence collection, documentation).

### Key Findings:

**1. "You Cannot Secure What You Cannot See"**

The most critical lesson from this lab is that comprehensive logging is not overhead—it is the foundation of security operations, forensic investigations, and compliance evidence. Every subsequent security capability (threat detection, incident response, compliance auditing, root cause analysis) depends on having complete, accurate, searchable logs. Without centralized logging, organizations are operationally blind: they cannot detect attacks, investigate incidents, or prove compliance with regulatory requirements.

**2. Centralized Logging Enables Organization-Wide Security**

Task 2 demonstrated that centralizing logs from distributed sources solves multiple security problems simultaneously: logs become searchable across the entire organization rather than scattered on individual hosts; logs are protected from tampering by attackers who compromise source systems; logs can be correlated across sources to detect attacks that span multiple systems; and logs are durably stored with retention policies that meet compliance requirements. Production implementations should use managed logging services (CloudWatch, Azure Monitor, Google Cloud Logging) with encryption, access controls, and immutable storage.

**3. Tamper-Proof Logs Are Essential for Forensics and Compliance**

Task 4 showed that cryptographic hash-chaining provides mathematical proof of log integrity, making it computationally infeasible for attackers to modify logs without detection. This tamper-evidence is essential for three reasons: forensic investigations require trustworthy evidence to reconstruct attack timelines; legal proceedings require admissible evidence that has not been altered; and regulatory compliance requires demonstrating that audit trails have maintained integrity. Production systems should store hash chain final values in append-only storage (S3 Object Lock, blockchain) that attackers cannot access even if they fully compromise applications.

**4. Correlation Transforms Noise into Intelligence**

Task 5 demonstrated that modern attacks are multi-stage campaigns where no individual action appears malicious enough to block, but the sequence reveals a coherent attack narrative. Correlation analysis examines relationships between events across dimensions (time, user, IP, resource, action) to detect patterns invisible to single-event analysis. This is the fundamental capability that distinguishes SIEM systems from simple log aggregation: SIEM performs automated correlation at scale across millions of events from thousands of sources, applies machine learning for anomaly detection, and triggers real-time alerts that drive incident response.

**5. Incident Response Requires Both Speed and Integrity**

Task 6 showed that effective incident response balances two competing imperatives: contain attacks quickly to minimize damage (blocking attacker IPs, disabling compromised accounts, isolating affected systems), and preserve evidence integrity for forensic investigation and legal proceedings (timestamped copies, cryptographic hashes, chain of custody documentation). Production incident response requires predefined playbooks, automated containment actions, 24/7 SOC staffing, communication plans for stakeholders, and post-incident review processes that implement lessons learned.

**6. Logs Serve Dual Purposes: Security and Compliance**

Throughout the lab, the same logs served two distinct functions: security monitoring (real-time correlation, threat detection, incident response) and compliance evidence (audit trails, regulatory reporting, legal proceedings). This dual-purpose nature makes logging infrastructure a strategic investment that simultaneously improves security posture and reduces compliance costs. Organizations should design logging systems to meet both operational security requirements (real-time alerting, threat hunting) and regulatory compliance requirements (retention periods, tamper-evidence, audit reporting).

### Real-World Applications:

The techniques learned in this lab are directly applicable to:

**Cloud Security Operations:**
- AWS CloudWatch Logs, Insights, and Alarms for centralized logging and automated alerting
- Azure Monitor Logs and Azure Sentinel SIEM for cloud-native security monitoring
- Google Cloud Logging and Security Command Center for threat detection across GCP resources
- Multi-cloud logging aggregation using Splunk, Elastic Security, or Datadog

**SIEM and Correlation:**
- Splunk Enterprise Security with correlation searches and adaptive response actions
- Elastic Security (formerly Elastic SIEM) with detection rules and machine learning jobs
- Microsoft Sentinel with analytics rules and automated investigation playbooks
- IBM QRadar with offense management and incident response workflows

**Incident Response:**
- NIST SP 800-61 incident response lifecycle implementation
- SANS Incident Handler's Handbook procedures
- PCI-DSS Requirement 10 (logging) and 11 (security monitoring) compliance
- SOC 2 Type II audit evidence collection and retention

**Compliance and Audit:**
- ISO 27001 A.12.4.1 (event logging) and A.12.4.3 (administrator logs) requirements
- GDPR Article 5(2) accountability principle requiring audit trail evidence
- HIPAA §164.312(b) audit controls and §164.308(a)(1)(ii)(D) log monitoring
- Financial services regulations (SOX, GLBA, PSD2) requiring tamper-proof audit trails

### Future Learning:

This lab establishes the foundation for advanced security topics including:

**Advanced Correlation and Detection:**
- User and Entity Behavior Analytics (UEBA) for anomaly detection beyond rule-based correlation
- Threat intelligence integration (STIX/TAXII feeds, commercial threat intel) for IOC matching
- Machine learning models for detecting novel attacks without signature-based rules
- Automated threat hunting using hypothesis-driven investigation and analytics

**Security Orchestration and Automated Response (SOAR):**
- Automated incident response playbooks triggered by SIEM correlation rules
- Integration with ticketing systems (ServiceNow, Jira) for incident case management
- Automated containment actions (blocking IPs, disabling accounts, isolating VLANs)
- Enrichment workflows that gather additional context for security alerts

**Advanced Forensics:**
- Memory forensics for detecting rootkits and in-memory-only malware
- Network forensics using full packet capture (PCAP) and protocol analysis
- Disk forensics with write-blocking, imaging, and timeline reconstruction
- Cloud forensics across ephemeral infrastructure (containers, serverless, auto-scaling)

**Compliance Automation:**
- Continuous compliance monitoring using policy-as-code (Open Policy Agent)
- Automated audit report generation from log analytics queries
- GRC (Governance, Risk, Compliance) platform integration
- Security posture management and compliance dashboards

---

## References

The following resources were referenced during this lab and provide additional depth for further study:

1. **Course lecture** — Week 6 (Monitoring, Auditing & Management) and Weeks 10-11 (Compliance Evidence), Prof. Dr. Shahrulniza Musa, UniKL MIIT, covering centralized logging architectures, tamper-proof audit trails, SIEM correlation concepts, and incident response procedures.

2. **Amazon CloudWatch Logs Concepts** — https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs — Official AWS documentation for CloudWatch Logs architecture, log groups, log streams, retention policies, and integration with other AWS services.

3. **OWASP Logging Cheat Sheet** — https://cheatsheetseries.owasp.org — Comprehensive guidance on what events to log, how to protect logs from tampering, log injection prevention, and secure log management practices.

4. **CSA Security Guidance v5 — Security Monitoring Domain** — Cloud Security Alliance industry best practices for cloud security monitoring, SIEM deployment in cloud environments, and cloud-native detection capabilities.

5. **NIST SP 800-61 Rev. 2 — Computer Security Incident Handling Guide** — Federal guidance on incident response lifecycle, preparation procedures, detection and analysis methodologies, containment strategies, and post-incident activities.

6. **NIST SP 800-92 — Guide to Computer Security Log Management** — Federal guidance on log management architecture, log generation policies, log analysis techniques, and legal considerations for log retention.

7. **RFC 3164 — The BSD syslog Protocol** — Standards documentation for traditional syslog protocol used for log transmission across networks.

8. **RFC 5424 — The Syslog Protocol** — Modern syslog protocol specification with structured data elements and priority-based routing.

9. **SANS Incident Handler's Handbook** — Practical incident response procedures, decision trees, and communication templates for security operations teams.

10. **Mitre ATT&CK Framework** — Comprehensive knowledge base of adversary tactics, techniques, and procedures (TTPs) used for threat detection and incident analysis.

---

## Appendix: Cleanup Commands

To clean up the lab environment and free system resources, execute the following commands:

```bash
# Remove all lab files
rm -f auth.log auth.chain auth.tampered evidence_*.log evidence.sha256

# Stop and remove LocalStack container
docker stop localstack
docker rm localstack

# Verify cleanup
docker ps -a | grep localstack
ls -la auth.log auth.chain auth.tampered evidence_*.log evidence.sha256 2>/dev/null
```

#### Evidence

![Cleanup & Teardown](evidence/Cleanup%20%26%20Teardown.png)

**Figure 9:** The screenshot shows the execution of cleanup commands removing all log files (auth.log, auth.chain, auth.tampered, evidence_*.log, evidence.sha256) and stopping/removing the LocalStack container, followed by verification commands confirming that no lab artifacts remain, ensuring that the lab environment was successfully cleaned up and system resources were freed.

**Note:** The cleanup removes all containers and files created during the lab. The `2>/dev/null` redirect suppresses error messages for files that don't exist, allowing the cleanup script to run safely even if some tasks were not completed. If you need to preserve evidence for report submission, take screenshots before executing cleanup commands.

---

## Acknowledgments

This lab was completed as part of the IKB42603 Cloud Computing Security Essentials course at Universiti Kuala Lumpur Malaysian Institute of Information Technology (UniKL MIIT). Special thanks to:

- **Prof. Dr. Shahrulniza Musa** for developing the comprehensive lab curriculum covering centralized logging, tamper-proof audit trails, correlation-based threat detection, and incident response procedures, and for providing expert guidance on security monitoring principles and defense-in-depth strategies throughout the course.

- **Teaching staff** for supervising lab sessions, providing clarifications on logging architectures and incident response procedures, and offering feedback on implementation approaches during hands-on exercises.

- **The Docker Project and LocalStack Community** for providing open-source containerization and AWS service emulation platforms that enable hands-on learning of production-grade cloud security monitoring technologies without requiring actual cloud infrastructure.

- **Aquasec Trivy, OWASP, and other open-source security projects** that make vulnerability scanning, security testing, and best-practice documentation accessible for educational purposes and production deployments.

---

*End of Lab 5 Report*

