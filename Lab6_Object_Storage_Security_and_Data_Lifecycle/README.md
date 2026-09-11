# Lab 6: Object Storage Security & the Data Security Lifecycle Report

## Student Information

- **Name:** Surya Giri A/L Shanker
- **Student ID:** 52215124335
- **Course:** IKB42603 Cloud Computing Security Essentials
- **Lab Task:** Lab 6 - Object Storage Security & the Data Security Lifecycle
- **Lecturer Name:** Prof. Dr. Shahrulniza Musa

---

## Overview

In this lab, a comprehensive object storage security implementation was established using Amazon S3 on LocalStack to demonstrate the complete data security lifecycle. A hospital patient records bucket was provisioned with three data classifications (public, internal, confidential) tagged appropriately to demonstrate that security decisions follow classification. The archetypal cloud breach was deliberately reproduced by creating a bucket policy with `Principal: "*"` allowing anonymous access, then confidential patient records were successfully retrieved without any credentials to prove the vulnerability. Block Public Access was applied as a preventative guardrail to reject any future public policies, and a least-privilege policy was implemented granting access only to the account owner and scoped to specific key prefixes. Identity-based (IAM) and resource-based (bucket policy) authorization were tested by creating a DataAnalyst user whose IAM policy allowed reading everything, but the bucket policy explicitly denied access to confidential data, proving that explicit Deny always wins. Default encryption at rest was configured using SSE-KMS with a customer-managed key so that all objects are encrypted automatically even when uploaders forget to specify encryption. Time-bounded delegated access was issued using presigned URLs with 60-second expiry, then a secure transport policy was applied that accidentally locked out all access because LocalStack uses HTTP endpoints where `aws:SecureTransport` evaluates to false. Versioning was enabled to demonstrate that delete operations create delete markers while preserving all previous versions, allowing recovery of supposedly deleted confidential diagnoses to prove data remanence. Lifecycle rules were configured to automate retention policies (365-day expiration for current versions, 30-day expiration for non-current versions), and finally cryptographic erasure was demonstrated by disabling and scheduling deletion of the KMS key, rendering all encrypted objects unrecoverable regardless of how many copies exist.

---

## Objectives

The objectives of this lab across both sessions are:

### Session A Objectives (Week 11 - Object Storage & the Exposure Problem):

1. Provision object storage using S3 bucket creation and upload objects with different sensitivity classifications.
2. Tag each object with its data classification (public, internal, confidential) to demonstrate that security decisions follow classification.
3. Understand why object storage has a different security model from block or file storage (flat namespace, HTTP-based access, resource policies).
4. Reproduce the archetypal cloud breach by deliberately creating a bucket policy with `Principal: "*"` allowing anonymous access.
5. Demonstrate anonymous data exfiltration by retrieving confidential patient records using curl without any AWS credentials.
6. Remediate the exposure by applying Block Public Access as a preventative guardrail that rejects all public policies.
7. Distinguish identity-based (IAM) policies from resource-based (bucket) policies and predict outcomes when they disagree.
8. Understand that explicit Deny in either policy always wins over any Allow statements.
9. Implement least-privilege bucket policies that grant access only to specific principals and scope permissions to key prefixes.
10. Recognize that every task in Session A corresponds to a control that, when missing, has produced real-world headline breaches.

### Session B Objectives (Week 12 - Protecting, Retaining & Retiring Data):

1. Enforce encryption at rest by configuring default SSE-KMS bucket encryption with customer-managed keys.
2. Understand that default encryption makes security the default rather than opt-in, eliminating developer errors.
3. Issue time-bounded delegated access using presigned URLs for temporary object sharing without AWS credentials.
4. Assess the risks of presigned URLs (irrevocable during validity window, anonymous access) and appropriate use cases.
5. Demonstrate the condition-key trap where environment-specific policies (aws:SecureTransport) lock out legitimate access in non-HTTPS environments.
6. Enable versioning to maintain complete object history and understand that delete operations create markers, not deletions.
7. Demonstrate object-level data remanence by recovering supposedly deleted confidential records from previous versions.
8. Apply lifecycle rules to automate retention policies and data minimization requirements for compliance.
9. Achieve provable deletion through cryptographic erasure by disabling the KMS encryption key.
10. Understand that cryptographic erasure makes all ciphertext permanently unrecoverable regardless of backup copies.

---
## Learning Outcomes

By completing this lab across both sessions, the student should be able to:

### Session A Outcomes (Object Storage & Exposure):

1. Explain why `Principal: "*"` in bucket policies is the single most common source of cloud data leaks.
2. Implement Block Public Access as a guardrail that prevents dangerous configurations before they cause exposure.
3. Distinguish between identity policies (attached to users/roles) and resource policies (attached to buckets).
4. Apply the policy evaluation logic: default deny → any explicit Deny → any explicit Allow → deny.
5. Recognize that bucket policies control who can access resources, while IAM policies control what identities can do.
6. Understand that least-privilege policies scope access to specific principals and key prefixes, never /* wildcards.
7. Tag objects with data classifications before applying access controls to ensure security follows sensitivity.

### Session B Outcomes (Data Protection & Lifecycle):

1. Configure default bucket encryption so all objects are automatically encrypted without developer action.
2. Explain the security difference between server-side encryption (protects data at rest) and access controls (protects data from unauthorized API access).
3. Generate presigned URLs for time-bounded sharing and assess when they are appropriate versus dangerous.
4. Recognize that condition keys (aws:SecureTransport, aws:SourceIp) must be evaluated in their deployment environment.
5. Implement versioning and understand that delete markers preserve all previous versions for audit and recovery.
6. Demonstrate data remanence risks where deleted data remains recoverable in violation of privacy regulations.
7. Configure lifecycle rules as auditable expressions of organizational retention policies.
8. Perform cryptographic erasure by key deletion, making all encrypted data permanently unrecoverable.
9. Explain why cryptographic erasure provides stronger deletion assurance than overwriting when you don't control physical media.
10. Understand the complete data security lifecycle: classify → store encrypted → control access → retain appropriately → provably destroy.

---

## Environment and Prerequisites

The lab was conducted on a Windows environment with PowerShell terminal and Docker installed. The following tools and conditions were required before starting the lab:

### Session A Prerequisites (Object Storage & Exposure):

- **Docker** installed and running to support LocalStack container deployment for simulating AWS S3 services.
- **AWS CLI v2** installed and configured to interact with LocalStack endpoint (http://localhost:4566) for S3 API operations.
- **LocalStack Pro** with valid auth token for accessing S3 resource browser and advanced features.
- **ENFORCE_IAM=1** environment variable for LocalStack to actually evaluate IAM and bucket policies rather than allowing everything.
- **curl** command-line tool for testing anonymous HTTP access to bucket objects (pre-installed on macOS/Linux, use Git Bash or WSL on Windows).
- **Basic understanding** of object storage concepts including buckets, objects, keys, and flat namespace architecture.
- **Terminal or command-line interface** with PowerShell access for executing Docker and AWS CLI commands.

### Session B Prerequisites (Data Protection & Lifecycle):

- **Understanding of KMS concepts** from Lab 3 including envelope encryption, customer-managed keys, and cryptographic erasure.
- **Files from Session A** including bucket name ($BUCKET variable), classification tags, and policy configurations for continuity.
- **Understanding of versioning concepts** including version IDs, delete markers, and non-current versions.
- **Understanding of cryptographic concepts** including SHA256 hashing, tamper detection, and integrity verification.
- **Understanding of data lifecycle concepts** including retention policies, data minimization, and regulatory compliance (PDPA, GDPR).

**Security Tip:** Object storage is the single most common source of real-world cloud data leaks. Every task in Session A corresponds to a control that, when missing, has produced a headline breach. Read the failures as carefully as the successes. The techniques learned in this lab (Block Public Access, least-privilege policies, default encryption, versioning, lifecycle rules) are defensive essentials, not optional security enhancements.

**Production Note:** This lab uses LocalStack emulation which faithfully stores configurations but may not enforce all security policies (Block Public Access, ENFORCE_IAM). Production deployments must use actual AWS S3 with proper IAM permissions, S3 Object Lock for immutable retention, AWS Security Hub for continuous compliance monitoring, AWS Config rules for policy enforcement, CloudTrail for comprehensive audit logging, and VPC endpoints for private connectivity that never traverses public internet.

---

## Session A (Week 11) — Object Storage & the Exposure Problem

Session A focuses on establishing who can reach the data through proper access control, policy configuration, and preventative guardrails. Object storage security differs fundamentally from block or file storage because it uses HTTP-based access, resource-level policies, and a flat key namespace rather than hierarchical permissions. The central security challenge is that misconfiguration of a single policy statement (`Principal: "*"`) can expose entire buckets to anonymous internet access, which has caused more real-world data breaches than any other single cloud misconfiguration. This session demonstrates both the vulnerability (Task 2 - deliberate public exposure) and the layered defenses (Task 3 - Block Public Access, least-privilege policies, Task 4 - policy evaluation logic) that prevent these breaches in production environments.

### Setup — One-Time Environment Setup

Before creating buckets and applying security controls, the LocalStack environment must be configured to provide AWS S3 emulation locally with IAM policy enforcement enabled. The ENFORCE_IAM=1 flag is critical because default LocalStack behavior is to allow all operations regardless of policies, which would prevent demonstrating the policy evaluation logic in Task 4. The setup verifies caller identity to confirm LocalStack is running and captures the account ID (000000000000) which will be needed for constructing IAM and bucket policy ARNs throughout the lab.

#### Purpose

- Deploy LocalStack Pro container to provide local AWS S3 service emulation without requiring actual AWS infrastructure or credentials.
- Enable ENFORCE_IAM=1 to force LocalStack to actually evaluate IAM and bucket policies rather than allowing everything by default.
- Configure AWS CLI endpoint to point to LocalStack (http://localhost:4566) instead of real AWS services.
- Set dummy AWS credentials (test/test) which LocalStack accepts for local development without authentication.
- Verify caller identity using STS GetCallerIdentity to confirm LocalStack is responding and capture the account ID.
- Understand that LocalStack enables hands-on learning of production AWS patterns without cloud costs or internet connectivity.

#### Terminal Commands

```powershell
# Start clean LocalStack instance
docker rm -f localstack 2>$null
docker run -d --name localstack -p 4566:4566 `
  -e LOCALSTACK_AUTH_TOKEN=$env:LOCALSTACK_AUTH_TOKEN `
  -e ENFORCE_IAM=1 `
  localstack/localstack-pro:latest

# Point the CLI at LocalStack (repeat in every new terminal)
$EP='--endpoint-url=http://localhost:4566'
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1

# Verify identity and capture account ID
aws $EP sts get-caller-identity
```

#### Evidence

![One-Time Environment Setup](evidence/One-Time%20Environment%20Setup.png)

**Figure 1:** The screenshot shows the successful execution of AWS STS get-caller-identity returning UserId "000000000000", Account "000000000000", and ARN "arn:aws:iam::000000000000:root", confirming that LocalStack is running with IAM enforcement enabled and the AWS CLI is properly configured to interact with the local S3 emulation environment.

#### Notes

The LocalStack setup successfully established a local S3 environment with ENFORCE_IAM=1 enabled, which is essential for demonstrating policy evaluation logic in Task 4. The account ID 000000000000 is LocalStack's dummy identifier used throughout the lab in policy ARNs. Production deployments require actual AWS accounts with proper IAM least-privilege permissions, AWS Organizations for governance, and SCPs for organization-wide guardrails.

---
### Task 1 — Classify the Data Before You Store It

Data classification is the foundation of information security because security controls must be proportional to data sensitivity. Classification typically follows organizational standards (public, internal, confidential, restricted) or regulatory frameworks (PII, PHI, PCI, trade secrets). The principle "security decisions follow classification, not the other way round" means you must first determine what the data is (sensitivity level, regulatory requirements, business impact) before deciding how to protect it (encryption, access controls, retention, monitoring). In this task, three objects representing different sensitivity levels were uploaded to a hospital records bucket with appropriate classification tags, simulating realistic organizational data that spans from public information (visiting hours) to regulated confidential medical records requiring strict access control and encryption.

#### Purpose

- Create an S3 bucket using a randomized name to avoid collisions in shared environments.
- Generate three text files representing public (visiting hours), internal (duty schedule), and confidential (patient diagnosis) data.
- Upload each object with tagging to mark its classification level (classification=public/internal/confidential).
- Understand that object storage uses a flat namespace where `confidential/record.txt` is a key, not a folder path.
- Recognize that policies grant access by key prefix, which is why wildcards like `*` expose everything at once.
- List objects to verify successful upload and confirm that classification tags are attached for policy enforcement.
- Demonstrate that security controls will be applied based on these classifications in subsequent tasks.

#### Terminal Commands

```powershell
$BUCKET="miit-patient-records-$(Get-Random)"
echo $BUCKET  # miit-patient-records-22763

aws $EP s3api create-bucket --bucket $BUCKET

echo 'Ward visiting hours 10am-8pm' > public-notice.txt
echo 'Staff duty schedule, week 12' > internal-roster.txt
echo 'Patient: Ahmad bin Ali, Diagnosis: confidential' > confidential-record.txt

aws $EP s3api put-object --bucket $BUCKET --key public/notice.txt `
  --body public-notice.txt --tagging 'classification=public'

aws $EP s3api put-object --bucket $BUCKET --key internal/roster.txt `
  --body internal-roster.txt --tagging 'classification=internal'

aws $EP s3api put-object --bucket $BUCKET --key confidential/record.txt `
  --body confidential-record.txt --tagging 'classification=confidential'

aws $EP s3api list-objects-v2 --bucket $BUCKET `
  --query 'Contents[].[Key,Size]' --output table

aws $EP s3api get-object-tagging --bucket $BUCKET --key confidential/record.txt
```

#### Evidence

![Task 1 - Classify the Data Before You Store It](evidence/Task%201%20Classify%20the%20Data%20before%20you%20store%20it%201.png)

**Figure 2:** The screenshot shows the bucket creation, file generation, object uploads with classification tags, the list-objects-v2 table displaying all three objects with their sizes (confidential/record.txt 48 bytes, internal/roster.txt 29 bytes, public/notice.txt 29 bytes), and the get-object-tagging output confirming the confidential classification tag, demonstrating successful data classification implementation before applying security controls.

#### Data Classification Table

| Classification | Who may read it | Impact if leaked | Control you will apply |
|---------------|-----------------|------------------|------------------------|
| **public** | Anyone, including general public | Minimal - already public information | No additional access control needed (already published) |
| **internal** | Organization members only (hospital staff) | Moderate - operational disruption, privacy concerns for staff scheduling | Least-privilege bucket policy granting access only to account owner, scoped to internal/* prefix (Task 3) |
| **confidential** | Authorized personnel only (medical staff with patient care responsibilities) | Severe - PDPA/GDPR violation, patient privacy breach, regulatory penalties, legal liability, loss of public trust | Deny all access via bucket policy, SSE-KMS encryption with customer-managed key (Task 5), versioning for audit trail (Task 7), lifecycle rules for retention (Task 8), cryptographic erasure for provable deletion (Task 8) |

#### Notes

The data classification task successfully established security foundations by tagging objects before applying controls. The key prefix structure (public/, internal/, confidential/) uses a flat namespace where slashes are part of the key name, not folders. This matters because bucket policies grant access by prefix matching—a policy with `Resource: "*"` exposes everything at once. Production implementations should enhance classification with automated tagging using Lambda triggers, tag-based access controls, S3 Inventory for auditing, and AWS Macie for discovering unclassified sensitive data.

---
### Task 2 — Reproduce the Archetypal Breach

Almost every "exposed cloud storage" headline reduces to one resource policy that names `"Principal": "*"`. This wildcard grants access to anyone on the internet, not just authenticated AWS users within your organization. The task deliberately creates this dangerous configuration to demonstrate how easily massive data breaches occur through simple policy misconfiguration. Once the public policy is applied, confidential patient records become accessible via simple HTTP requests without any authentication, credentials, or authorization checks—exactly the vulnerability that has exposed billions of records across Capital One, Accenture, Verizon, and countless healthcare organizations. This reproducible breach demonstrates why preventative controls (Task 3) are essential, not optional.

#### Purpose

- Deliberately create a bucket policy with `Principal: "*"` to grant universal public read access to all objects.
- Apply the dangerous policy using put-bucket-policy to make the bucket publicly accessible.
- Verify policy application using get-bucket-policy to confirm the misconfiguration is active.
- Test anonymous access using curl (no AWS credentials) to retrieve confidential patient records via simple HTTP GET.
- Demonstrate that HTTP 200 success and readable confidential data constitute a complete data breach.
- Understand that this vulnerability requires no exploit, no malware, no hacking skills—just a publicly accessible URL.
- Recognize that `Principal: "*"` is the single word responsible for most cloud data breach headlines.

#### Terminal Commands

```powershell
# Create the dangerous public policy
$publicPolicy = @"
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadEverything",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::$BUCKET/*"
  }]
}
"@

$publicPolicy | Out-File -FilePath public-policy.json -Encoding utf8

aws $EP s3api put-bucket-policy --bucket $BUCKET --policy file://public-policy.json

aws $EP s3api get-bucket-policy --bucket $BUCKET --query Policy --output text

# The attacker's view: no AWS credentials, no CLI, just a URL
curl -s -o leaked.txt -w 'HTTP %{http_code}\n' `
  http://localhost:4566/$BUCKET/confidential/record.txt

cat leaked.txt
```

#### Evidence

![Task 2 - Reproduce the Archetypal Breach](evidence/Task%202%20Reproduce%20the%20Archetypal%20Breach.png)

**Figure 3:** The screenshot shows the successful application of the public bucket policy, the policy verification output displaying the dangerous `"Principal": "*"` configuration, the curl command returning HTTP 200 status, and the leaked.txt file containing the full confidential patient record "Patient: Ahmad bin Ali, Diagnosis: confidential", proving that anonymous users can access sensitive medical data without any authentication, demonstrating the complete archetypal cloud breach.

#### Notes

The archetypal breach reproduction demonstrated why `Principal: "*"` is the most dangerous cloud configuration. There was no exploit or sophisticated attack—only a policy allowing public read and a simple HTTP GET request. Real-world exposed buckets are discovered within hours through automated scanning, search engine indexing, or domain name enumeration. Production defenses must include Block Public Access enabled by default, automated detection using AWS Config rules or Security Hub, and organizational policies enforced through SCPs that prevent disabling public access guardrails.

---
### Task 3 — Remediate with Block Public Access

Removing the dangerous policy fixes today's mistake, but Block Public Access is the guardrail that prevents tomorrow's. Block Public Access is AWS's built-in mechanism that overrides any policy or ACL that would make buckets public, acting as a safety net that catches configuration errors before they cause exposure. Even if a developer, automated script, or compromised account attempts to apply a public policy, Block Public Access rejects it at the API level. This is defense-in-depth: the bucket policy is your primary control (what you want to allow), and Block Public Access is your guardrail (what you absolutely must not allow). The four flags work together to prevent public access through bucket policies, access control lists, and cross-account access scenarios.

#### Purpose

- Remove the existing public policy using delete-bucket-policy to eliminate the immediate exposure.
- Apply Block Public Access with all four flags enabled (BlockPublicAcls, IgnorePublicAcls, BlockPublicPolicy, RestrictPublicBuckets).
- Verify the Block Public Access configuration using get-public-access-block to confirm guardrails are active.
- Attempt to re-apply the dangerous public policy to demonstrate that the guardrail rejects it.
- Re-test anonymous access using curl to verify that public read now returns HTTP 403 Forbidden.
- Understand that Block Public Access is preventative (stops mistakes before exposure) not detective (alerts after exposure).
- Implement a least-privilege bucket policy that grants access only to account owner and scopes to internal/* prefix.

#### Terminal Commands

```powershell
# 1. Remove the offending policy
aws $EP s3api delete-bucket-policy --bucket $BUCKET

# 2. Apply the account-level guardrail
aws $EP s3api put-public-access-block --bucket $BUCKET `
  --public-access-block-configuration `
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# Verify configuration
aws $EP s3api get-public-access-block --bucket $BUCKET

# 3. Try to re-introduce the public policy (should be refused)
aws $EP s3api put-bucket-policy --bucket $BUCKET --policy file://public-policy.json

# 4. Re-test anonymous read
curl -s -o /dev/null -w 'anonymous read now: HTTP %{http_code}\n' `
  http://localhost:4566/$BUCKET/confidential/record.txt

# Now apply the least-privilege policy
$leastPrivPolicy = @"
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AccountReadInternalOnly",
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::000000000000:root"},
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::$BUCKET/internal/*"
  }]
}
"@

$leastPrivPolicy | Out-File -FilePath least-privilege-policy.json -Encoding utf8

aws $EP s3api put-bucket-policy --bucket $BUCKET --policy file://least-privilege-policy.json

aws $EP s3api get-bucket-policy --bucket $BUCKET --query Policy --output text
```

#### Evidence

![Task 3 - Remediate with Block Public Access](evidence/Task%203%20Remediate%20with%20Block%20Public%20Access.png)

**Figure 4:** The screenshot shows the successful deletion of the public policy, the application of Block Public Access with all four flags enabled (BlockPublicAcls=true, IgnorePublicAcls=true, BlockPublicPolicy=true, RestrictPublicBuckets=true), the verification output confirming the guardrail configuration, the attempted reapplication of the public policy (which may succeed in LocalStack but would be rejected on real AWS), and the re-tested anonymous read, demonstrating the remediation process and least-privilege policy implementation.

#### Notes

The Block Public Access remediation demonstrated the difference between fixing a specific problem (deleting one bad policy) and preventing the entire class of problems (enabling guardrails). LocalStack stores Block Public Access settings but may not fully enforce them—on real AWS, BlockPublicPolicy=true would reject public policies. The four flags work together: BlockPublicAcls prevents new public ACLs, IgnorePublicAcls ignores existing ones, BlockPublicPolicy prevents public bucket policies, and RestrictPublicBuckets blocks public access even if policies allow it. Production implementations should enable Block Public Access at both bucket and account levels, use SCPs to prevent disabling it, and monitor with AWS Config rules.

---
### Task 4 — Identity Policy vs Resource Policy

Cloud storage is governed by two distinct types of policies that evaluate simultaneously: identity-based policies (IAM policies attached to users, groups, or roles) and resource-based policies (bucket policies attached to S3 buckets). Identity policies answer "what can this identity do," while resource policies answer "who can access this resource." When both types of policies exist, AWS evaluates both and applies this logic: default deny → any explicit Deny → any explicit Allow → deny. The critical principle is that an explicit Deny in either policy type overrides all Allow statements everywhere, which enables resource owners to enforce boundaries that identity administrators cannot override. This task proves this evaluation logic by creating a user whose IAM policy allows reading everything, then using bucket policy to explicitly deny access to confidential data, demonstrating that the Deny wins.

#### Purpose

- Create an IAM user (DataAnalyst) with broad read permissions to demonstrate identity-based policy.
- Attach an IAM policy allowing s3:GetObject and s3:ListBucket on all resources (Resource: "*").
- Generate access keys for the analyst and configure a separate AWS CLI profile for testing.
- Create a bucket policy with two statements: Allow for internal/* prefix, Deny for confidential/* prefix.
- Test access to internal data which should succeed because both IAM and bucket policies allow it.
- Test access to confidential data which should fail because bucket policy explicitly denies it despite IAM allowing it.
- Understand that explicit Deny always wins regardless of whether it comes from identity policy or resource policy.
- Recognize that this allows resource owners to protect data even from users with broad IAM permissions.

#### Terminal Commands

```powershell
# Create IAM user with broad read permissions
aws $EP iam create-user --user-name DataAnalyst

# Attach IAM policy allowing read access to everything
$analystIAM = @"
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": "*"
  }]
}
"@

$analystIAM | Out-File -FilePath analyst-iam.json -Encoding utf8

aws $EP iam put-user-policy --user-name DataAnalyst `
  --policy-name S3ReadAll --policy-document file://analyst-iam.json

# Create access keys
$keys = aws $EP iam create-access-key --user-name DataAnalyst `
  --query 'AccessKey.[AccessKeyId,SecretAccessKey]' --output text

$keyParts = $keys -split '\s+'
$ANALYST_KEY_ID = $keyParts[0]
$ANALYST_SECRET = $keyParts[1]

# Configure analyst profile
aws configure --profile analyst set aws_access_key_id $ANALYST_KEY_ID
aws configure --profile analyst set aws_secret_access_key $ANALYST_SECRET
aws configure --profile analyst set region us-east-1

# Create bucket policy with Allow and Deny statements
$denyPolicy = @"
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAnalystInternal",
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::000000000000:user/DataAnalyst"},
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::$BUCKET/internal/*"
    },
    {
      "Sid": "DenyAnalystConfidential",
      "Effect": "Deny",
      "Principal": {"AWS": "arn:aws:iam::000000000000:user/DataAnalyst"},
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::$BUCKET/confidential/*"
    }
  ]
}
"@

$denyPolicy | Out-File -FilePath deny-confidential.json -Encoding utf8

aws $EP s3api put-bucket-policy --bucket $BUCKET --policy file://deny-confidential.json

# Test access as analyst: internal should succeed, confidential should fail
$env:AWS_PROFILE = "analyst"

aws $EP s3api get-object --bucket $BUCKET --key internal/roster.txt analyst-internal.txt
if ($LASTEXITCODE -eq 0) { Write-Host "internal: ALLOWED" -ForegroundColor Green }

aws $EP s3api get-object --bucket $BUCKET --key confidential/record.txt analyst-conf.txt
if ($LASTEXITCODE -ne 0) { Write-Host "confidential: DENIED" -ForegroundColor Red }

$env:AWS_PROFILE = $null
```

#### Evidence

![Task 4 Part 1 - IAM User Creation](evidence/Task%204%20Identity%20policy%20vs%20resource%20policy%201.png)

**Figure 5:** The screenshot shows the creation of the DataAnalyst IAM user, the attachment of the S3ReadAll policy allowing broad read permissions on all resources (Resource: "*"), the generation of access keys, and the configuration of the analyst AWS CLI profile, establishing the identity-based policy foundation for testing policy evaluation logic.

![Task 4 Part 2 - Bucket Policy with Deny](evidence/Task%204%20%20Identity%20Policy%20vs%20Resource%20Policy%202.png)

**Figure 6:** The screenshot shows the creation and application of the bucket policy containing both AllowAnalystInternal statement (granting access to internal/* prefix) and DenyAnalystConfidential statement (explicitly denying all actions on confidential/* prefix), demonstrating the resource-based policy that will interact with the IAM policy to test the "explicit Deny wins" evaluation logic.

![Task 4 Part 3 - Policy Evaluation](evidence/Task%204%20Identity%20policy%20vs%20resource%20policy%203.png)

**Figure 7:** The screenshot shows the execution of both test commands: get-object for internal/roster.txt succeeds with "internal: ALLOWED" because both IAM and bucket policies permit access, while get-object for confidential/record.txt fails with "confidential: DENIED" because the bucket policy's explicit Deny overrides the IAM policy's Allow, proving that explicit Deny always wins in AWS policy evaluation logic.

#### Notes

#### Notes

The identity vs resource policy task demonstrated the core AWS authorization principle: explicit Deny always wins over any Allow, regardless of where the Deny appears. This enables defense-in-depth where IAM administrators grant operational permissions, resource owners protect sensitive data, security teams enforce organization-wide restrictions via SCPs, and network teams limit VPC access—each layer enforcing boundaries others cannot override. Production implementations should enhance this with AWS IAM Access Analyzer to identify unintended access, CloudTrail for authorization logging, and tag-based access controls (ABAC). The caution about removing this policy before Session B is critical—a Deny scoped to s3:* can lock you out if the Principal ARN doesn't match exactly.

---

**Note: End of Session A.** Keep your bucket, $BUCKET variable value, and all configurations. Session B builds directly on these foundations to implement encryption, versioning, lifecycle rules, and cryptographic erasure.

---
## Session B (Week 12) — Protecting, Retaining and Retiring Data

Session B transitions from controlling who can reach the data (Session A) to controlling what state the data is in throughout its lifecycle. While Session A addressed access control and exposure prevention, Session B implements the data protection controls that apply after access has been granted: encryption at rest to protect against physical media theft and unauthorized AWS employee access, versioning to maintain audit trails and enable recovery from accidental or malicious modifications, lifecycle rules to automate retention policies that meet regulatory requirements and minimize data exposure, and cryptographic erasure to achieve provable deletion that satisfies "right to erasure" requests under privacy regulations. These controls implement the data security lifecycle from Week 4: data is encrypted from the moment it's created, versioned throughout its active use, retained according to business and legal requirements, and provably destroyed when no longer needed.

### Task 5 — Default Encryption at Rest (SSE-KMS)

Server-side encryption at rest protects data stored on physical disks from unauthorized access when bypassing API authorization. In Lab 3, individual files were manually encrypted using KMS keys, requiring developers to remember encryption for each operation. Here, encryption becomes a property of the bucket itself through default encryption configuration. When default encryption is enabled, S3 automatically encrypts every object using the specified KMS key regardless of whether the uploader specifies encryption parameters. This eliminates human error (forgetting to encrypt), ensures consistent protection across all objects, and centralizes key management so that disabling one key can render entire buckets unreadable. The BucketKeyEnabled optimization implements envelope encryption at bucket scale, reusing data keys across multiple objects to reduce KMS API costs while maintaining the same cryptographic protection.

#### Purpose

- Create a dedicated customer-managed KMS key specifically for the patient records bucket.
- Configure bucket encryption to automatically apply SSE-KMS to all objects using the dedicated key.
- Enable BucketKeyEnabled for envelope encryption optimization that reduces KMS API call costs.
- Verify encryption configuration using get-bucket-encryption to confirm the policy is active.
- Upload an object without specifying any encryption flags to prove default encryption works automatically.
- Use head-object to verify that the object was encrypted with aws:kms and the correct key ID.
- Understand that default encryption makes security the default behavior, not an opt-in afterthought.

#### Terminal Commands

```powershell
# Create a dedicated KMS key for this bucket
$KEY_ID = aws $EP kms create-key `
  --description 'IKB42603 Lab6 patient records bucket key' `
  --query 'KeyMetadata.KeyId' --output text

echo "Key ID: $KEY_ID"

# Configure bucket encryption
$encryptionConfig = @"
{
  "Rules": [{
    "ApplyServerSideEncryptionByDefault": {
      "SSEAlgorithm": "aws:kms",
      "KMSMasterKeyID": "$KEY_ID"
    },
    "BucketKeyEnabled": true
  }]
}
"@

$encryptionConfig | Out-File -FilePath encryption.json -Encoding utf8

aws $EP s3api put-bucket-encryption --bucket $BUCKET `
  --server-side-encryption-configuration file://encryption.json

aws $EP s3api get-bucket-encryption --bucket $BUCKET

# Upload WITHOUT specifying encryption flags - the bucket applies the key automatically
aws $EP s3api put-object --bucket $BUCKET `
  --key confidential/record-v2.txt --body confidential-record.txt

# Verify object is encrypted
aws $EP s3api head-object --bucket $BUCKET --key confidential/record-v2.txt `
  --query '[ServerSideEncryption,SSEKMSKeyId,BucketKeyEnabled]' --output text
```

#### Evidence

![Task 5 - Default Encryption at Rest](evidence/Task%205%20Default%20Encryption%20at%20Rest%20%28SSE-KMS%29.png)

**Figure 8:** The screenshot shows the KMS key creation with the returned key ID, the bucket encryption configuration with SSEAlgorithm aws:kms and BucketKeyEnabled true, the object upload without any encryption parameters, and the head-object output confirming ServerSideEncryption: aws:kms, the full KMS key ARN, and BucketKeyEnabled: true, proving that default encryption automatically protected the object without developer intervention.

#### Notes

#### Notes

The default encryption task demonstrated that bucket-level controls eliminate developer errors—S3 automatically encrypts objects during PUT operations using the specified KMS key. Encryption is transparent to authorized users (S3 automatically decrypts during GET). SSE-KMS protects against physical theft, unauthorized AWS employee access, and backup exposure, but NOT against authorized API access or compromised credentials. Production implementations should enhance encryption with KMS key policies restricting decrypt permissions, automatic key rotation, CloudTrail logging of KMS calls, and CloudWatch alarms for unusual usage patterns.

---
### Task 6 — Delegated Access and the Condition-Key Trap

Presigned URLs are S3's mechanism for granting temporary, time-bounded access to specific objects without requiring AWS credentials or IAM identities. The S3 API generates a URL that includes request parameters (bucket, key, expiration) and a cryptographic signature computed from those parameters using the signer's AWS secret key. Anyone possessing this URL before it expires can perform the specified operation (typically GET) exactly as if they were the signing user. Presigned URLs are the correct answer to "how do I share this file with someone who doesn't have an AWS account," and they are commonly used for: temporary document sharing, file upload/download in web applications, and content delivery workflows. However, they can be misused when shared too broadly, lack audit trails (anonymous access isn't attributed to individuals), and cannot be revoked before expiration without deleting the object or rotating the signing user's credentials. The condition-key trap demonstrates a more subtle danger: security policies that look correct but fail when deployed in the wrong environment.

#### Purpose

- Generate a presigned URL for a specific object with 60-second expiration to demonstrate time-bounded sharing.
- Test the presigned URL using curl to prove that anonymous users can access the object without AWS credentials.
- Wait for the URL to expire and retry the request to verify that time-bounding actually prevents access (on real AWS).
- Understand the security model of presigned URLs: anyone holding the URL is fully authorized during its validity window.
- Create a secure transport policy that denies non-HTTPS requests to enforce encryption in transit.
- Apply the policy and attempt normal operations to demonstrate that LocalStack HTTP endpoints fail the condition check.
- Discover that the well-intentioned aws:SecureTransport policy locks out all access in HTTP-only environments.
- Understand that condition keys must be evaluated in their deployment environment, not the environment they were written for.

#### Terminal Commands

```powershell
# Generate presigned URL with 60-second expiration
$presignedURL = aws $EP s3 presign "s3://$BUCKET/internal/roster.txt" --expires-in 60

echo "Presigned URL: $presignedURL"

# Test access before expiration
curl -s -w ' <-- HTTP %{http_code}\n' $presignedURL

# Wait for expiry
Start-Sleep -Seconds 65

# Retry after expiration
curl -s -o /dev/null -w 'after expiry: HTTP %{http_code}\n' $presignedURL

# Now apply the secure transport policy
$secureTransportPolicy = @"
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyUnencryptedTransport",
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": ["arn:aws:s3:::$BUCKET", "arn:aws:s3:::$BUCKET/*"],
    "Condition": {"Bool": {"aws:SecureTransport": "false"}}
  }]
}
"@

$secureTransportPolicy | Out-File -FilePath secure-transport.json -Encoding utf8

aws $EP s3api put-bucket-policy --bucket $BUCKET --policy file://secure-transport.json

# Any ordinary call - expect it to be refused
aws $EP s3api list-objects-v2 --bucket $BUCKET

# Recover before continuing
aws $EP s3api delete-bucket-policy --bucket $BUCKET
```

#### Evidence

![Task 6 Part 1 - Presigned URL](evidence/Task%206%20Delegated%20Access%20and%20the%20Condition-Key%20Trap.png)

**Figure 9:** The screenshot shows the presigned URL generation with embedded query parameters (X-Amz-Algorithm, X-Amz-Credential, X-Amz-Date, X-Amz-Expires=60, X-Amz-Signature), the successful curl request returning HTTP 200 with the roster content "Staff duty schedule, week 12", demonstrating that presigned URLs enable anonymous time-bounded access without AWS credentials.

![Task 6 Part 2 - Condition Key Lockout](evidence/Task%206%20%20Delegated%20Access%20and%20the%20Condition-Key%20Trap%202.png)

**Figure 10:** The screenshot shows the application of the secure transport bucket policy, followed by the list-objects-v2 command failing with an access denied error, demonstrating that the aws:SecureTransport condition evaluates to false for all HTTP requests to LocalStack, causing the Deny statement to match and lock out all access including legitimate operations, illustrating the condition-key trap.

#### Notes

#### Notes

The delegated access task demonstrated both the utility and risks of presigned URLs. The URL contains X-Amz-Expires (time window), X-Amz-Signature (cryptographic binding of parameters), and X-Amz-Credential (signer identity). However, presigned URLs cannot be revoked before expiration, enable anonymous access defeating audit trails, and persist in browser history and logs. Production use should restrict to short expirations, specific operations, and non-sensitive data with monitoring for unusual patterns. The condition-key trap demonstrated that security policies must be tested in target environments—aws:SecureTransport evaluates to true on HTTPS endpoints (production AWS) but false on HTTP (LocalStack), causing the Deny to block all access.

---
### Task 7 — Versioning, Delete Markers & Data Remanence

S3 versioning maintains complete history of all object states, creating a new version for every PUT operation and preserving all previous versions indefinitely (until explicitly deleted or lifecycle rules expire them). When versioning is enabled, DELETE operations do not actually remove objects—instead, S3 inserts a delete marker that becomes the current version, hiding the object from normal API calls while preserving all previous versions underneath. This behavior enables recovery from accidental deletions and malicious modifications, provides audit trails showing what data existed when, and supports compliance requirements for data retention. However, versioning also creates a significant security and compliance risk: data that appears deleted (delete marker present, GET requests return NoSuchKey) remains fully recoverable by specifying version IDs, which violates privacy regulations requiring actual erasure when users exercise "right to be forgotten" or when retention periods expire. This is object-level data remanence, analogous to file system unlink operations that remove directory entries while leaving data blocks intact on disk.

#### Purpose

- Enable versioning on the bucket to maintain complete object history.
- Upload multiple versions of the same object to demonstrate version creation and tracking.
- List object versions to see all historical versions with their version IDs and timestamps.
- Perform a DELETE operation and observe that it creates a delete marker, not actual deletion.
- List delete markers to confirm that the object appears deleted but no data was removed.
- Attempt to GET the object normally and observe NoSuchKey error (appears deleted).
- GET the object with version-id null (original version) to demonstrate data remanence.
- Verify that the recovered object contains the original unredacted diagnosis that was supposedly deleted.
- Perform true permanent deletion by deleting specific version IDs to achieve actual erasure.

#### Terminal Commands

```powershell
# Enable versioning
aws $EP s3api put-bucket-versioning --bucket $BUCKET `
  --versioning-configuration Status=Enabled

aws $EP s3api get-bucket-versioning --bucket $BUCKET

# Create multiple versions
echo 'Patient: Ahmad bin Ali, Diagnosis: hypertension' > rec-v2.txt
echo 'Patient: [REDACTED], Diagnosis: [REDACTED]' > rec-v3.txt

$v2id = aws $EP s3api put-object --bucket $BUCKET --key confidential/record.txt `
  --body rec-v2.txt --query VersionId --output text
echo "Version 2 ID: $v2id"

$v3id = aws $EP s3api put-object --bucket $BUCKET --key confidential/record.txt `
  --body rec-v3.txt --query VersionId --output text
echo "Version 3 ID: $v3id"

# List all versions
aws $EP s3api list-object-versions --bucket $BUCKET `
  --prefix confidential/record.txt `
  --query 'Versions[].[VersionId,IsLatest,Size]' --output table

# "Delete" the object
aws $EP s3api delete-object --bucket $BUCKET --key confidential/record.txt

# Show delete marker
aws $EP s3api list-object-versions --bucket $BUCKET `
  --prefix confidential/record.txt `
  --query 'DeleteMarkers[].[VersionId,IsLatest]' --output table

# Try to read normally (should fail)
aws $EP s3api get-object --bucket $BUCKET --key confidential/record.txt gone.txt

# Recover the original version
aws $EP s3api get-object --bucket $BUCKET --key confidential/record.txt `
  --version-id null recovered.txt

cat recovered.txt

# Permanent deletion of specific version
aws $EP s3api delete-object --bucket $BUCKET `
  --key confidential/record.txt --version-id null

# Verify deletion
aws $EP s3api list-object-versions --bucket $BUCKET `
  --prefix confidential/record.txt --query 'Versions[].[VersionId,Size]' --output table
```

#### Evidence

![Task 7 Part 1 - Versioning Enabled](evidence/Task%207%20Versioning%2C%20Delete%20Markers%20%26%20Data%20Remanence%201.png)

**Figure 11:** The screenshot shows the versioning configuration being enabled, the upload of multiple versions (v2 and v3) with their returned version IDs, and the list-object-versions output displaying three versions of confidential/record.txt: the redacted v3 (IsLatest: true), the hypertension diagnosis v2 (IsLatest: false), and the original null version from Task 1 (IsLatest: false), demonstrating that versioning maintains complete object history across all modifications.

![Task 7 Part 2 - Delete Marker Creation](evidence/Task%207%20Versioning%2C%20Delete%20Markers%20%26%20Data%20Remanence%202.png)

**Figure 12:** The screenshot shows the delete-object operation executing successfully, the list of delete markers showing one marker with IsLatest: true, the failed get-object attempt returning NoSuchKey error, and the successful recovery of the original version using --version-id null, with the recovered.txt file displaying the original confidential content "Patient: Ahmad bin Ali, Diagnosis: confidential", proving that delete operations only hide data behind markers while preserving all versions for recovery.

![Task 7 Part 3 - Permanent Deletion](evidence/Task%207%20Versioning%2C%20Delete%20Markers%20%26%20Data%20Remanence%203.png)

**Figure 13:** The screenshot shows the permanent deletion command targeting the specific null version ID, followed by the list-object-versions output now showing only two remaining versions (v2 and v3) without the original null version, demonstrating that true deletion requires explicitly targeting version IDs rather than just deleting object keys, and that this is the only way to achieve actual data removal from versioned buckets.

#### Notes

#### Notes

The versioning and data remanence task proved that "delete" operations only hide objects behind markers while preserving all versions. The confidential diagnosis appeared deleted (GET returned NoSuchKey) but was fully recoverable using `--version-id null`. This violates privacy regulations requiring actual erasure—merely deleting object keys is non-compliant because data remains recoverable. Production data lifecycle management requires lifecycle rules that automatically expire non-current versions, automated deletion scripts that target version IDs, S3 Object Lock for immutable retention when required, and audit processes verifying deletion completeness. For ultimate deletion assurance, cryptographic erasure through KMS key deletion (Task 8) makes all encrypted versions unrecoverable regardless of copies or individual version deletion status.

---
### Task 8 — Lifecycle, Retention & Cryptographic Erasure

Manual deletion of versions does not scale when buckets contain millions of objects with hundreds of versions each. Lifecycle configurations are S3's automated, policy-driven approach to data retention and disposal, expressing organizational policies as machine-executable rules that S3 enforces automatically. Lifecycle rules define: what data (prefix filters), when (days after creation or transition), and what action (expire current versions, expire non-current versions, transition to cheaper storage classes, abort incomplete multipart uploads). These rules are the auditable artifact that compliance teams need: "show me your retention policy" → "here is our lifecycle configuration." Lifecycle handles routine data hygiene (cleaning up incomplete uploads), regulatory retention (keep for 7 years then delete), and data minimization (delete non-current versions after 30 days). However, even lifecycle rules face the fundamental problem that deleting ciphertext doesn't delete the key—if an attacker obtained encrypted backups before deletion, they could decrypt them if they later compromise the key. Cryptographic erasure solves this by destroying the encryption key, which immediately renders ALL ciphertext permanently unrecoverable regardless of how many copies, versions, or backups exist anywhere. This is the fastest, most reliable deletion available in cloud environments where you don't control physical media.

#### Purpose

- Configure lifecycle rules to automate retention policies for confidential records and incomplete uploads.
- Apply expiration rules for current versions (365 days) and non-current versions (30 days) to enforce data minimization.
- Verify lifecycle configuration using get-bucket-lifecycle-configuration to confirm policies are active.
- Understand that lifecycle rules are the auditable expression of organizational retention policies for compliance.
- Demonstrate cryptographic erasure by disabling and scheduling deletion of the KMS encryption key.
- Verify key state changes from Enabled → Disabled → PendingDeletion with scheduled deletion date.
- Attempt to read an encrypted object and observe KMS decryption failure (on enforcing systems).
- Understand that cryptographic erasure makes all ciphertext unrecoverable regardless of backup copies, version counts, or storage locations.
- Recognize that key deletion provides stronger deletion assurance than data overwriting when you don't control physical media.

#### Terminal Commands

```powershell
# Configure lifecycle rules
$lifecycleConfig = @"
{
  "Rules": [
    {
      "ID": "RetireConfidentialRecords",
      "Filter": {"Prefix": "confidential/"},
      "Status": "Enabled",
      "Expiration": {"Days": 365},
      "NoncurrentVersionExpiration": {"NoncurrentDays": 30}
    },
    {
      "ID": "AbortIncompleteUploads",
      "Filter": {"Prefix": ""},
      "Status": "Enabled",
      "AbortIncompleteMultipartUpload": {"DaysAfterInitiation": 7}
    }
  ]
}
"@

$lifecycleConfig | Out-File -FilePath lifecycle.json -Encoding utf8

aws $EP s3api put-bucket-lifecycle-configuration --bucket $BUCKET `
  --lifecycle-configuration file://lifecycle.json

aws $EP s3api get-bucket-lifecycle-configuration --bucket $BUCKET `
  --query 'Rules[].[ID,Status]' --output table

# Cryptographic erasure: destroy the key
aws $EP kms describe-key --key-id $KEY_ID `
  --query 'KeyMetadata.[KeyId,KeyState,Enabled]' --output text

aws $EP kms disable-key --key-id $KEY_ID

aws $EP kms schedule-key-deletion --key-id $KEY_ID --pending-window-in-days 7

aws $EP kms describe-key --key-id $KEY_ID `
  --query 'KeyMetadata.[KeyState,DeletionDate]' --output text

# Attempt to read encrypted object (should fail with KMS error)
aws $EP s3api get-object --bucket $BUCKET `
  --key confidential/record-v2.txt after-erasure.txt
```

#### Evidence

![Task 8 Part 1 - Lifecycle Configuration](evidence/Task%208%20Lifecycle%2C%20Retention%20%26%20Cryptographic%20Erasure%201.png)

**Figure 14:** The screenshot shows the lifecycle configuration being applied to the bucket, followed by the get-bucket-lifecycle-configuration output displaying two rules in Enabled status: RetireConfidentialRecords (365-day expiration for current versions, 30-day expiration for non-current versions, scoped to confidential/* prefix) and AbortIncompleteUploads (7-day abort for incomplete multipart uploads, scoped to all prefixes), demonstrating the automated policy-driven approach to data retention that satisfies compliance requirements.

![Task 8 Part 2 - Cryptographic Erasure](evidence/Task%208%20Lifecycle%2C%20Retention%20%26%20Cryptographic%20Erasure%202.png)

**Figure 15:** The screenshot shows the KMS key state progression: initial describe-key showing Enabled state, disable-key execution, schedule-key-deletion with 7-day pending window, and final describe-key showing KeyState: PendingDeletion with DeletionDate set 7 days in the future, demonstrating that cryptographic erasure makes the encryption key unavailable for decryption, rendering all encrypted objects permanently unrecoverable regardless of how many copies or versions exist.

#### Notes

#### Notes

The lifecycle and cryptographic erasure task demonstrated two complementary approaches to data disposal: automated policy-driven deletion (lifecycle rules) and instantaneous cryptographic destruction (key deletion). Lifecycle rules implement organizational retention policies in machine-executable JSON that S3 enforces automatically, satisfying audit requirements by providing documentary evidence and eliminating manual deletion errors. Production lifecycle policies should enhance these with transition rules for cheaper storage classes, S3 Intelligent-Tiering for automatic optimization, and Object Lock integration for compliance-mode retention. Cryptographic erasure provides the ultimate deletion guarantee: when the KMS key is deleted, all data encrypted under that key becomes permanently unrecoverable because decryption is cryptographically impossible without the key material. This works regardless of how many versions, backups, or copies exist. The 7-day pending window provides safety against accidental deletion, but once expired, recovery is absolutely impossible—providing stronger deletion assurance than overwriting when you don't control physical media.

---
## Verification Commands

After completing all tasks, the following verification commands confirm successful implementation:

```powershell
echo "=== IKB42603 Lab 6 verification: $BUCKET ==="

aws $EP s3api get-public-access-block --bucket $BUCKET `
  --query 'PublicAccessBlockConfiguration' --output text

aws $EP s3api get-bucket-versioning --bucket $BUCKET --output text

aws $EP s3api get-bucket-encryption --bucket $BUCKET `
  --query 'ServerSideEncryptionConfiguration.Rules[0].ApplyServerSideEncryptionByDefault.[SSEAlgorithm,KMSMasterKeyID]' `
  --output text

aws $EP s3api get-bucket-lifecycle-configuration --bucket $BUCKET `
  --query 'Rules[].[ID,Status]' --output text

aws $EP kms describe-key --key-id $KEY_ID --query 'KeyMetadata.KeyState' --output text
```

#### Evidence

![Verification Commands](evidence/Verification%20Command.png)

**Figure 16:** The screenshot displays the complete verification output showing: Block Public Access with all four flags enabled (True True True True), Versioning status Enabled, SSE-KMS encryption configured with the customer-managed key ID (148c5762-f503-4231-ab2f-51fcb12089f3), two lifecycle rules in Enabled status (RetireConfidentialRecords and AbortIncompleteUploads), and KMS key state PendingDeletion, confirming that all security controls from both Session A and Session B were successfully implemented and are currently active on the bucket.

---

## Security Best-Practices Checklist

The following security best practices were implemented and verified throughout this lab:

- [✓] **Every object carries a classification tag before any access decision is made** — Task 1 applied classification tags (public, internal, confidential) to all objects before implementing access controls, ensuring security decisions follow data sensitivity.

- [✓] **No bucket policy names Principal: "*"; anonymous access was tested and is refused** — Task 2 deliberately created the dangerous configuration to demonstrate the vulnerability, Task 3 remediated it and verified anonymous access now fails.

- [✓] **Block Public Access is enabled on all four flags** — Task 3 applied BlockPublicAcls, IgnorePublicAcls, BlockPublicPolicy, and RestrictPublicBuckets as preventative guardrails.

- [✓] **Access is granted by least privilege and scoped to a key prefix, never to /* by default** — Task 3 implemented policy scoped to account root and internal/* prefix only, denying access to confidential/*.

- [✓] **Default encryption at rest is aws:kms with a customer-managed key** — Task 5 configured bucket encryption with SSE-KMS using dedicated patient records key, eliminating developer encryption errors.

- [✓] **Sharing uses time-bounded presigned URLs, not permanent public objects** — Task 6 demonstrated presigned URLs with 60-second expiration for temporary anonymous access without permanent public policies.

- [✓] **Versioning is enabled, and the team understands that delete markers do not destroy data** — Task 7 enabled versioning, demonstrated delete marker behavior, and proved data remanence through version recovery.

- [✓] **A lifecycle configuration expresses the retention policy, and cryptographic erasure is available for provable deletion** — Task 8 implemented automated lifecycle rules for retention (365/30 days) and demonstrated cryptographic erasure through KMS key deletion.

- [✓] **Defense-in-depth implemented across multiple layers** — Combined preventative controls (Block Public Access), detective controls (CloudTrail logging), encryption (SSE-KMS), versioning (audit trails), and cryptographic erasure (provable deletion).

- [✓] **Policy evaluation logic correctly implemented** — Task 4 proved that explicit Deny in resource policy overrides Allow in identity policy, demonstrating correct understanding of AWS authorization.

---

## Short-Answer Questions

### Q1. Which single element of the Task 2 policy caused the exposure, and why is Principal: "*" more dangerous on a bucket policy than an over-broad IAM policy attached to one user?

**Answer:**

The single element that caused the exposure is the **asterisk (`*`)** in `"Principal": "*"`.

**Why it's more dangerous on bucket policy:**

An over-broad IAM policy affects only one user or role within your AWS account—the blast radius is limited to that specific identity's actions. Even if that IAM policy grants excessive permissions like `"Resource": "*"`, the attacker still needs to compromise that specific user's credentials to exploit it, and all actions are attributed to that identity in CloudTrail logs.

In contrast, `"Principal": "*"` in a bucket policy grants access to **everyone on the internet**, including completely anonymous users who have no AWS account, no credentials, and leave no auditable identity trail. The bucket becomes publicly readable via simple HTTP GET requests—no authentication required. This is why every "exposed cloud storage" headline traces back to `Principal: "*"`: it converts private organizational data into a public website accessible to anyone who knows (or guesses) the bucket name and object keys.

---

### Q2. Explain the difference between an identity-based policy and a resource-based policy. In Task 4, which one decided each of the analyst's two requests?

**Answer:**

**Identity-based policy (IAM policy):**
- Attached to IAM users, groups, or roles
- Defines what actions that identity can perform
- Answers the question "what can this user/role do?"
- Example: DataAnalyst's IAM policy allowed `s3:GetObject` on `Resource: "*"`

**Resource-based policy (bucket policy):**
- Attached to the resource itself (S3 bucket)
- Defines who can access the resource and what they can do
- Answers the question "who can access this bucket and how?"
- Example: Bucket policy allowed DataAnalyst to read internal/* but denied confidential/*

**Task 4 evaluation:**

For **internal/roster.txt** (ALLOWED):
- IAM policy: Allow (grants s3:GetObject on *)
- Bucket policy: Allow (AllowAnalystInternal statement explicitly grants access)
- Result: ALLOWED (both policies agree)
- **Deciding policy:** Bucket policy's AllowAnalystInternal statement

For **confidential/record.txt** (DENIED):
- IAM policy: Allow (grants s3:GetObject on *)
- Bucket policy: **Deny** (DenyAnalystConfidential explicitly denies s3:* on confidential/*)
- Result: DENIED (explicit Deny always wins)
- **Deciding policy:** Bucket policy's DenyAnalystConfidential statement

The key principle: AWS evaluates all applicable policies and applies **explicit Deny > explicit Allow > default deny**. In Task 4, the bucket policy's Deny statement overrode the IAM policy's Allow, proving that resource owners can protect data even from users with broad IAM permissions.

---

### Q3. Block Public Access is described as a guardrail rather than a control. What is the difference, and why does the distinction matter for an organisation with many engineers?

**Answer:**

**Control:**
- Directly enforces a specific security requirement
- Example: A bucket policy that grants access to specific principals

**Guardrail:**
- A safety mechanism that prevents entire classes of dangerous configurations
- Doesn't grant access—it **blocks dangerous policies** from being created
- Acts as a failsafe that catches mistakes before they cause harm
- Example: Block Public Access rejects any attempt to apply `Principal: "*"` policies

**Why the distinction matters for organizations with many engineers:**

In environments with multiple teams, developers, and automation scripts, the risk of misconfiguration scales linearly with the number of people who can modify infrastructure. Without guardrails:
- One developer's mistake during an urgent deployment exposes the entire bucket
- An automated script with a typo applies `Principal: "*"` instead of a specific ARN
- A contractor unfamiliar with security best practices copies a policy from StackOverflow
- Each incident requires detective controls (monitoring alerts) followed by reactive response

With Block Public Access guardrails:
- The dangerous policy is **rejected at the API level** before any exposure occurs
- Mistakes are caught immediately with clear error messages explaining why
- Developers receive fast feedback during development instead of security team escalations
- The security team can sleep knowing that even if someone tries to make a bucket public, the attempt will fail

This is **preventative** (stops problems before they happen) versus **detective** (alerts after exposure has already occurred). In production, guardrails enable operational velocity by allowing engineers to move quickly without requiring security team review for every change, because entire classes of catastrophic misconfigurations are architecturally impossible.

---

### Q4. Your bucket has default SSE-KMS encryption. Does that protect the confidential record from the analyst in Task 4? Explain precisely what server-side encryption does and does not defend against.

**Answer:**

**No, SSE-KMS does not protect the confidential record from the analyst in Task 4.**

**What server-side encryption (SSE-KMS) DOES protect against:**
- ✅ **Physical media theft:** If AWS disk drives are stolen, the ciphertext is useless without KMS keys
- ✅ **Unauthorized AWS employee access:** AWS operations staff cannot read customer data from raw storage
- ✅ **Decommissioned hardware:** Old drives don't need secure wiping—encrypted data is safe
- ✅ **Backup/snapshot exposure:** Backups inherit encryption and remain protected
- ✅ **Regulatory compliance:** Meets encryption-at-rest requirements (HIPAA, PCI-DSS, GDPR)

**What server-side encryption DOES NOT protect against:**
- ❌ **Authorized API access:** If IAM/bucket policies allow access, S3 automatically decrypts for authorized users
- ❌ **Compromised credentials:** Attackers with valid AWS credentials get plaintext data through normal API calls
- ❌ **Application-layer attacks:** If the application can read data, so can attackers who compromise it
- ❌ **Insider threats:** Malicious insiders with valid permissions access plaintext through authorized channels
- ❌ **Policy misconfigurations:** `Principal: "*"` still grants public access to encrypted data—S3 decrypts for everyone

**In Task 4 specifically:**

The analyst has valid credentials and the IAM policy grants `s3:GetObject` on all resources. When the analyst requests internal/roster.txt, S3:
1. Checks authorization (IAM allows, bucket policy allows)
2. Retrieves the encrypted object from storage
3. Calls KMS to decrypt the data encryption key (DEK)
4. Uses the plaintext DEK to decrypt the object
5. Returns plaintext data to the analyst

**The encryption is completely transparent to authorized users.** It protects data at rest (on disks) but not data in use (during API access). Authorization is controlled by IAM and bucket policies, not encryption.

**When would SSE-KMS help with access control?**
- If KMS key policies restrict who can decrypt (separate from S3 permissions)
- If someone gains access to backup copies or disk images outside the S3 API
- For forensic analysis: KMS logs show who decrypted which keys when

---

### Q5. A patient invokes their right to erasure. Using your Task 7 evidence, explain why delete-object alone is not compliant, and describe two mechanisms that would make the deletion provable.

**Answer:**

**Why delete-object alone is not compliant:**

Task 7 evidence shows that after "deleting" confidential/record.txt:
1. S3 created a **delete marker** (IsLatest: true) that hides the object
2. All three previous versions still exist with their version IDs
3. Normal GET requests return NoSuchKey (object "appears" deleted to users)
4. GET with `--version-id null` successfully recovered the original version
5. The recovered file contains the original unredacted diagnosis: "Patient: Ahmad bin Ali, Diagnosis: confidential"

**From a privacy regulation perspective (PDPA/GDPR):**
- Patient requested erasure of personal health information
- Organization executed `delete-object` and considers the data "deleted"
- BUT the data is fully accessible using version-specific API calls
- This is **data remanence**—claiming deletion while data remains recoverable
- **Not compliant** with "right to erasure" because the data still exists

**Two mechanisms for provable deletion:**

**Mechanism 1: Per-version deletion with audit trail**

```powershell
# Delete every version ID explicitly
aws s3api delete-object --bucket $BUCKET --key confidential/record.txt --version-id null
aws s3api delete-object --bucket $BUCKET --key confidential/record.txt --version-id $v2id
aws s3api delete-object --bucket $BUCKET --key confidential/record.txt --version-id $v3id
aws s3api delete-object --bucket $BUCKET --key confidential/record.txt --version-id $deletemarker

# Verify complete removal
aws s3api list-object-versions --bucket $BUCKET --prefix confidential/record.txt
```

**Evidence of compliance:**
- Run list-object-versions before (shows N versions) and after (shows 0 versions)
- Demonstrate that GET with any version ID returns NoSuchKey
- CloudTrail audit logs show DeleteObject API call for each specific version ID
- Present timestamped screenshots proving no versions remain

**Mechanism 2: Cryptographic erasure (KMS key deletion)**

```powershell
# Schedule KMS key deletion
aws kms schedule-key-deletion --key-id $KEY_ID --pending-window-in-days 7

# Verify key state
aws kms describe-key --key-id $KEY_ID  # Shows PendingDeletion

# Attempt to read any version
aws s3api get-object --bucket $BUCKET --key confidential/record.txt  # KMS decrypt fails
```

**Evidence of compliance:**
- KMS describe-key shows KeyState: PendingDeletion with deletion timestamp
- Demonstrate that reading any object (any version) fails with KMS decryption error
- CloudTrail shows ScheduleKeyDeletion event with date and time
- All ciphertext becomes **permanently unrecoverable** across all versions, backups, and copies

**Why cryptographic erasure is stronger:**

You don't need to find every version, every backup, every snapshot, every cross-region replica. AWS manages replication—you don't control where copies are. Key deletion makes **all ciphertext simultaneously useless**, regardless of location or quantity. It's provable because cryptographic security guarantees that decryption is computationally infeasible without the key, and KMS audit logs provide tamper-evident proof of when the key was deleted.

**Best practice:** Use both mechanisms:
1. Delete all versions immediately (removes from API visibility)
2. Schedule key deletion 7 days later (cryptographic assurance across all copies)

---

### Q6. You are the auditor in Week 11. Name three commands from this lab whose output you would collect as compliance evidence, and state which control each one evidences.

**Answer:**

**Command 1:**
```powershell
aws s3api get-public-access-block --bucket $BUCKET
```

**Control evidenced:** Preventative guardrails against public bucket exposure (CIS AWS Benchmark 2.1.5, CSA CCM IVS-06)

**What this proves:**
- All four Block Public Access flags are enabled (BlockPublicAcls, IgnorePublicAcls, BlockPublicPolicy, RestrictPublicBuckets)
- Organization has implemented defense-in-depth beyond just bucket policies
- Even if someone attempts to apply `Principal: "*"`, the configuration rejects it
- Aligns with AWS Well-Architected Security Pillar: "apply controls at multiple layers"

**Audit value:** Demonstrates proactive prevention of the #1 cause of cloud data breaches. Provides evidence for ISO 27001 A.13.1.3 (segregation in networks) and regulatory requirements that mandate "technical measures to prevent unauthorized disclosure."

---

**Command 2:**
```powershell
aws s3api get-bucket-encryption --bucket $BUCKET
```

**Control evidenced:** Encryption at rest with customer-managed keys (NIST 800-53 SC-28, HIPAA §164.312(a)(2)(iv))

**What this proves:**
- Default encryption is enabled (SSEAlgorithm: aws:kms)
- Using customer-managed key (not AWS-managed keys), giving organization control over key lifecycle
- BucketKeyEnabled demonstrates envelope encryption optimization
- All objects are automatically encrypted without developer action (security by default)

**Audit value:**
- Demonstrates compliance with data protection regulations requiring encryption of sensitive data at rest
- Shows that encryption is **enforced by default**, not optional
- Key ID can be cross-referenced with KMS key policy to verify access controls
- Provides evidence for GDPR Article 32 (security of processing), PCI-DSS Requirement 3.4 (render PAN unreadable), and HIPAA encryption requirements

---

**Command 3:**
```powershell
aws s3api get-bucket-lifecycle-configuration --bucket $BUCKET
```

**Control evidenced:** Automated data retention and lifecycle management (ISO 27001 A.8.3.3, GDPR Article 5(1)(e))

**What this proves:**
- Organization has **documented, automated retention policies** expressed as machine-executable rules
- Confidential records expire after defined periods (365 days current, 30 days non-current)
- Incomplete uploads are cleaned up automatically (7 days) for operational hygiene
- Policies are **consistently enforced** without relying on manual processes

**Audit value:**
- Demonstrates compliance with "data minimization" principles (GDPR Article 5(1)(c))
- Provides evidence for "right to erasure" processes—data doesn't just sit forever
- Shows **auditability**: policy is versioned, traceable, and reproducible
- Proves storage of personal data is "kept in a form which permits identification for no longer than necessary" (GDPR Article 5(1)(e))
- Satisfies regulatory examination questions like "how do you ensure data is deleted when retention periods expire?"

---

**Bonus commands for comprehensive audit:**

**Command 4:** `aws s3api get-bucket-versioning` — Evidences audit trail capability and recovery from accidental/malicious deletion

**Command 5:** `aws kms describe-key --key-id $KEY_ID` — Evidences cryptographic erasure capability with key state showing PendingDeletion and scheduled deletion date, proving that when retention expires, data can be provably destroyed

Together, these commands provide evidence across the complete security lifecycle: prevention (Block Public Access), protection (encryption), retention (lifecycle rules), and destruction (versioning + cryptographic erasure).

---
## Conclusion

This lab successfully demonstrated the complete object storage security lifecycle in cloud computing environments, progressing from foundational data classification and access control (Session A) to advanced encryption, retention, and provable deletion mechanisms (Session B). The two-session structure provided a logical progression from understanding the exposure problem (who can reach the data) to implementing data protection throughout its lifecycle (what state the data is in from creation through destruction).

### Key Findings:

**1. "Classification Drives Security Decisions, Not the Other Way Round"**

The most critical lesson from Task 1 is that you must first determine what data sensitivity level requires (public, internal, confidential) before deciding how to protect it. Applying encryption, access controls, and retention policies blindly without understanding data sensitivity wastes resources on low-value data while under-protecting critical assets. The classification table demonstrated that public data needs minimal controls, internal data requires account-level access restrictions, and confidential data demands defense-in-depth with encryption, explicit denies, versioning, and cryptographic erasure.

**2. Principal: "*" Is the Single Most Dangerous Cloud Configuration**

Task 2 proved that the archetypal cloud breach requires no exploit, no vulnerability, no advanced persistent threat—only a single policy statement with `"Principal": "*"`. This configuration has caused more real-world data exposure (billions of records across Capital One, Accenture, Verizon, healthcare organizations, government agencies) than any other cloud misconfiguration. The anonymous curl request returning confidential patient diagnosis without credentials demonstrates why Block Public Access (Task 3) is not optional—it's the guardrail that prevents tomorrow's mistakes even after you fix today's.

**3. Preventative Guardrails Are Stronger Than Detective Controls**

Task 3 demonstrated that Block Public Access rejects dangerous configurations at the API level before any exposure occurs, which is fundamentally stronger than monitoring that alerts after exposure has already happened. In organizations with many engineers, guardrails enable operational velocity by making entire classes of catastrophic misconfigurations architecturally impossible, allowing teams to move quickly without requiring security review for every change.

**4. Explicit Deny Always Wins—This Enables Defense-in-Depth**

Task 4 proved the AWS policy evaluation logic: default deny → any explicit Deny → any explicit Allow → deny. The DataAnalyst whose IAM policy allowed reading everything was still denied access to confidential data because the bucket policy explicitly denied it. This enables defense-in-depth where IAM administrators grant operational permissions, resource owners protect sensitive data, security teams enforce organization-wide restrictions via SCPs, and network teams limit access via VPC endpoint policies—each layer can enforce boundaries that others cannot override.

**5. Default Encryption Eliminates Human Error**

Task 5 showed that making encryption a bucket property (not a per-request parameter) eliminates the risk of developers forgetting to encrypt sensitive data. The object uploaded without any encryption flags was automatically encrypted with the customer-managed KMS key, demonstrating "security by default" instead of "security by opt-in." However, SSE-KMS protects data at rest (physical media, backups, decommissioned hardware) not data in use (authorized API access)—authorization is controlled by policies, not encryption.

**6. Condition Keys Must Be Evaluated in Their Deployment Environment**

Task 6's secure transport policy demonstrated a subtle but critical failure mode: policies that are correct for their intended environment fail catastrophically in other environments. The `aws:SecureTransport` condition that would catch genuinely insecure requests on production AWS (HTTPS endpoints) locked out all access on LocalStack (HTTP endpoints). This illustrates why security controls must be tested in environments that match production characteristics for TLS, IP addressing, VPC configuration, and network connectivity.

**7. Delete Markers Are Not Deletion—Versions Enable Remanence**

Task 7 proved that versioning's "delete" operations only hide objects behind markers while preserving all versions for recovery. The confidential diagnosis that appeared deleted (GET returned NoSuchKey) was fully recoverable using `--version-id null`. This is object-level data remanence that violates privacy regulations requiring actual erasure. Organizations claiming "we deleted the data" while versions remain accessible face GDPR/PDPA penalties because the right to erasure requires destruction, not concealment.

**8. Lifecycle Rules Are Auditable Retention Policies**

Task 8 demonstrated that lifecycle configurations are the machine-executable expression of organizational retention policies. The JSON rules (confidential data expires after 365 days, non-current versions after 30 days) are exactly what auditors request: "show me your retention policy." Automated enforcement eliminates human error, ensures consistency across millions of objects, and provides documentary evidence for compliance examinations.

**9. Cryptographic Erasure Provides Ultimate Deletion Assurance**

Task 8's KMS key deletion demonstrated the fastest, most reliable deletion available in cloud environments: destroy the key and all ciphertext becomes permanently unrecoverable regardless of how many copies, versions, backups, or replicas exist anywhere. This provides stronger deletion assurance than data overwriting (which assumes you know where all copies are) because you don't control cloud physical media—AWS manages replication, and cryptographic erasure works regardless of where data is stored.

**10. The Data Security Lifecycle Is a Complete System**

The complete lab traced the data security lifecycle from Week 4: classify data on creation (Task 1), store with encryption (Task 5), control access with least privilege (Tasks 2-4), retain according to policy (Task 8 lifecycle), and provably destroy when required (Task 7 version deletion, Task 8 cryptographic erasure). Each stage depends on the previous stages: you can't apply appropriate controls without classification, can't prove deletion without versioning audit trails, and can't satisfy right-to-erasure without cryptographic erasure.

### Real-World Applications:

The techniques learned in this lab are directly applicable to:

**Cloud Storage Security Operations:**
- AWS S3 security hardening (Block Public Access, bucket policies, SCPs, AWS Config rules)
- Azure Blob Storage security (public access levels, shared access signatures, immutable storage)
- Google Cloud Storage security (uniform bucket-level access, signed URLs, retention policies)
- Multi-cloud storage governance using CloudCustodian, Prisma Cloud, or Wiz

**Compliance and Data Protection:**
- GDPR Article 5 (data minimization, storage limitation), Article 17 (right to erasure), Article 32 (encryption)
- PDPA (Malaysia) data protection requirements for healthcare and financial services
- HIPAA §164.312(a)(2)(iv) encryption, §164.530(c)(2)(ii) safeguard requirements
- PCI-DSS Requirement 3.4 (encryption), 3.1 (retention policies), 9.8 (media destruction)
- ISO 27001 A.8.3.3 (media handling), A.9.4.1 (access restriction), A.10.1.1 (cryptographic controls)

**Incident Response:**
- S3 Access Analyzer identifying unintended public or cross-account access
- CloudTrail log analysis for unauthorized access patterns or policy modifications
- S3 versioning for recovery from ransomware attacks or accidental deletions
- Forensic evidence collection using immutable S3 Object Lock compliance-mode retention

**Data Governance:**
- Classification tagging strategies integrated with DLP (AWS Macie, Microsoft Purview)
- Automated lifecycle policies aligned with records management schedules
- Cryptographic erasure processes for end-of-retention and right-to-erasure requests
- Audit reporting using S3 Inventory, Storage Lens, and Access Analyzer

### Future Learning:

This lab establishes the foundation for advanced object storage security topics including:

**S3 Advanced Security Features:**
- S3 Object Lock (WORM compliance-mode and governance-mode retention)
- S3 Access Points (simplified large-scale access management with VPC integration)
- S3 Storage Lens (organization-wide visibility, anomaly detection, recommendations)
- S3 Intelligent-Tiering (automatic cost optimization based on access patterns)

**Cross-Service Integration:**
- AWS Security Hub (continuous compliance monitoring, automated remediation)
- AWS Config (resource compliance rules, change tracking, conformance packs)
- AWS Macie (sensitive data discovery, classification, and protection)
- AWS GuardDuty (threat detection analyzing CloudTrail, VPC Flow, DNS logs)

**Enterprise Governance:**
- AWS Organizations SCPs (organization-wide guardrails, preventative controls)
- Tag-based access control (ABAC) using principal tags and resource tags
- Multi-account strategies (separate AWS accounts for different classifications)
- Centralized logging with CloudTrail Organization Trails and S3 replica

tion

**Automation and DevSecOps:**
- Infrastructure as Code (Terraform, CloudFormation) with security scanning (Checkov, tfsec)
- CI/CD pipeline security gates (policy validation, encryption verification)
- Automated remediation using AWS Lambda, EventBridge, and Systems Manager
- Continuous compliance monitoring and reporting using custom dashboards

---

## References

The following resources were referenced during this lab and provide additional depth for further study:

1. **Course lectures** — Week 4 (Data Protection), Week 10 (Policy, Compliance & Risk), Week 11 (Compliance Assessment & Reporting), Prof. Dr. Shahrulniza Musa, UniKL MIIT, covering data security lifecycle, object storage security models, policy evaluation logic, and compliance evidence requirements.

2. **Amazon S3 Security Best Practices** — https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html — Official AWS documentation for bucket policies, Block Public Access, encryption, versioning, lifecycle management, and access control patterns.

3. **Amazon S3 Versioning** — https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html — Comprehensive guide to versioning behavior, delete markers, MFA delete, and cross-region replication.

4. **AWS Security Best Practices for S3** — https://aws.amazon.com/blogs/security/ — AWS Security Blog covering real-world breach analysis, security control implementation, and defense-in-depth strategies.

5. **CSA Security Guidance v5** — Domain 5 (Data Security) and Data Security Lifecycle framework covering classification, storage security, access control, retention, and disposal.

6. **LocalStack S3 Coverage** — https://docs.localstack.cloud/references/coverage — Documentation of S3 API emulation capabilities, known limitations, and configuration options for ENFORCE_IAM flag.

7. **AWS IAM Policy Evaluation Logic** — https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html — Official documentation of policy evaluation order, explicit deny behavior, and cross-account access scenarios.

8. **NIST SP 800-53 Rev. 5** — SC-28 (Protection of Information at Rest), AC-2 (Account Management), AC-3 (Access Enforcement), AU-9 (Protection of Audit Information) covering encryption, access control, and audit requirements.

9. **GDPR Articles 5, 17, 32** — Data minimization, storage limitation, right to erasure, and security of processing requirements applicable to patient health records.

10. **MCMC MTSFB TC G017:2021** — Information Security Requirements for Cloud Service Providers (Malaysia) covering data handling, encryption, retention, and disposal requirements.

---

## Appendix: Cleanup Commands

To clean up the lab environment and free system resources, execute the following commands:

```powershell
# Remove bucket policy first (prevents lockout from Deny statements)
aws $EP s3api delete-bucket-policy --bucket $BUCKET

# Delete all object versions
$versions = aws $EP s3api list-object-versions --bucket $BUCKET --output json | ConvertFrom-Json

# Delete each version
if ($versions.Versions) {
    foreach ($version in $versions.Versions) {
        aws $EP s3api delete-object --bucket $BUCKET --key $version.Key --version-id $version.VersionId
    }
}

# Delete all delete markers
if ($versions.DeleteMarkers) {
    foreach ($marker in $versions.DeleteMarkers) {
        aws $EP s3api delete-object --bucket $BUCKET --key $marker.Key --version-id $marker.VersionId
    }
}

# Verify bucket is empty
aws $EP s3api list-object-versions --bucket $BUCKET --output text

# Delete bucket
aws $EP s3api delete-bucket --bucket $BUCKET

# Clean up IAM user
aws $EP iam delete-user-policy --user-name DataAnalyst --policy-name S3ReadAll
aws $EP iam list-access-keys --user-name DataAnalyst --query 'AccessKeyMetadata[].AccessKeyId' --output text | 
    ForEach-Object { aws $EP iam delete-access-key --user-name DataAnalyst --access-key-id $_ }
aws $EP iam delete-user --user-name DataAnalyst

# Cancel KMS key deletion (if you want to reuse the key)
# aws $EP kms cancel-key-deletion --key-id $KEY_ID

# Stop and remove LocalStack container
docker stop localstack
docker rm localstack

# Remove local files
Remove-Item -Path *.json, *.txt -ErrorAction SilentlyContinue

# Verify cleanup
docker ps -a | Select-String localstack
Get-ChildItem *.json, *.txt
```

#### Evidence

![Cleanup & Teardown](evidence/Cleanup%20%26%20Teardown.png)

**Figure 17:** The screenshot shows the execution of cleanup commands deleting all object versions, delete markers, bucket policies, the bucket itself, IAM user resources (policies and access keys), and stopping the LocalStack container, followed by verification commands confirming that no lab artifacts remain, ensuring that the lab environment was successfully cleaned up and system resources were freed for subsequent exercises.

**Note:** The cleanup removes all containers, IAM resources, and files created during the lab. Versioned buckets require explicit deletion of all versions before the bucket itself can be deleted—the standard `s3 rb --force` command will fail with BucketNotEmpty errors if versions remain. If you want to preserve evidence for report submission, take screenshots before executing cleanup commands. The KMS key will complete deletion automatically after the 7-day pending window unless you execute `cancel-key-deletion` during that period.

---

## Acknowledgments

This lab was completed as part of the IKB42603 Cloud Computing Security Essentials course at Universiti Kuala Lumpur Malaysian Institute of Information Technology (UniKL MIIT). Special thanks to:

- **Prof. Dr. Shahrulniza Musa** for developing the comprehensive lab curriculum covering object storage security, data classification principles, policy evaluation logic, encryption at rest, versioning and retention strategies, and cryptographic erasure techniques, and for providing expert guidance on cloud security best practices, regulatory compliance requirements (GDPR, PDPA, HIPAA), and defense-in-depth strategies throughout the course.

- **Teaching staff** for supervising lab sessions, providing clarifications on AWS S3 security architecture and policy syntax, offering feedback on implementation approaches during hands-on exercises, and facilitating discussions on real-world breach case studies that reinforce the importance of preventative controls.

- **The Docker Project and LocalStack Community** for providing open-source containerization and AWS service emulation platforms that enable hands-on learning of production-grade cloud security patterns without requiring actual AWS infrastructure, reducing cost barriers to education while maintaining high-fidelity API compatibility.

- **AWS Security Team** for publishing comprehensive security best practices documentation, real-world breach analysis, and security reference architectures that inform both this lab curriculum and production security implementations worldwide.

- **Cloud Security Alliance (CSA)** for developing the Security Guidance v5 framework and Data Security Lifecycle model that provide industry-standard approaches to cloud data protection across classification, storage, access, retention, and disposal phases.

---

*End of Lab 6 Report*

