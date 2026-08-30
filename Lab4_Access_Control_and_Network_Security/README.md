# IKB42603 Cloud Computing Security Essentials
## Lab 4: Access Control & Network Security
**Student Name:** Surya  
**Lab Title:** Authentication vs Authorization, Network Segmentation and Host Hardening  
**Date:** August 30, 2026  
**Platform:** Docker & Kubernetes

---

## Table of Contents
1. [Lab Overview](#lab-overview)
2. [Lab Learning Outcomes](#lab-learning-outcomes)
3. [Technical Prerequisites](#technical-prerequisites)
4. [Session A: Authentication & Authorization](#session-a-authentication--authorization)
   - [Task 1: Authentication - Password-Protected Service](#task-1-authentication---password-protected-service)
   - [Task 2: Add a Second Factor (MFA/TOTP)](#task-2-add-a-second-factor-mfatotp)
   - [Task 3: Authorization - RBAC Roles](#task-3-authorization---rbac-roles)
5. [Session B: Network Security & Hardening](#session-b-network-security--hardening)
   - [Task 4: Network Segmentation (Three-Tier)](#task-4-network-segmentation-three-tier)
   - [Task 5: Firewall Rules (Default-Deny)](#task-5-firewall-rules-default-deny)
   - [Task 6: Container/Host Hardening](#task-6-containerhost-hardening)
6. [Verification Commands](#verification-commands)
7. [Short-Answer Questions](#short-answer-questions)
8. [Security Best-Practices Checklist](#security-best-practices-checklist)
9. [Cleanup & Teardown](#cleanup--teardown)
10. [Conclusion](#conclusion)

---

## Lab Overview

This lab focuses on implementing critical security controls in cloud environments:
- **Session A (Week 7):** Controls **WHO** gets in (Authentication & Authorization)
- **Session B (Week 8):** Controls **WHAT** they can reach and reduces **WHAT** an intruder could exploit (Network Security & Hardening)

**Security Principle:** *Identity is the perimeter.* Every control asks: "Are you who you claim?" and "Are you allowed to do this?"

---

## Lab Learning Outcomes

At the end of this lab, I was able to:
1. ✅ Distinguish and implement **authentication** (who you are) and **authorization** (what you may do)
2. ✅ Add a second factor with a **TOTP (MFA)** code and verify it
3. ✅ Configure **network access control and segmentation** so services reach only what they must
4. ✅ **Harden a container image**: non-root, minimal, dropped capabilities, read-only filesystem
5. ✅ Scan an image for vulnerabilities and apply the principle of **least privilege** across compute, network and storage

---

## Technical Prerequisites

- ✅ Laptop with Docker installed
- ✅ Terminal access
- ✅ `kind` and `kubectl` installed
- ✅ `oathtool` for TOTP (MFA)
- ✅ Trivy container scanner
- ✅ Internet connection for initial image downloads

---

## Session A: Authentication & Authorization

Session A focuses on controlling **WHO** gets access to systems and **WHAT** they are allowed to do.

---

### Task 1: Authentication - Password-Protected Service

**Objective:** Run a web service behind HTTP Basic authentication where only requests with valid credentials get access.

#### Step 1.1: Create Password File

```bash
docker run --rm httpd:alpine htpasswd -nbB student 'P@ssw0rd!' > htpasswd.txt
```

**Purpose:** Generate a bcrypt-hashed password file for the user `student` with password `P@ssw0rd!`

#### Step 1.2: Create Nginx Configuration

```bash
cat > default.conf <<'EOF'
server { listen 80;
 location / { auth_basic "Restricted";
 auth_basic_user_file /etc/nginx/.htpasswd;
 return 200 'Authenticated OK\n'; } }
EOF
```

**Purpose:** Configure nginx to require HTTP Basic authentication for all requests.

#### Step 1.3: Start the Authentication Service

```bash
docker run --rm -d --name authsvc -p 8080:80 \
 -v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf \
 -v $(pwd)/htpasswd.txt:/etc/nginx/.htpasswd nginx
```

**Purpose:** Start nginx container with authentication enabled on port 8080.

#### Step 1.4: Test Without Credentials (Should Fail)

```bash
curl -s -o /dev/null -w 'no-creds: %{http_code}\n' http://localhost:8080
```

**Expected Output:**
```
no-creds: 401
```

**✅ Result:** Access denied - received HTTP 401 Unauthorized

#### Step 1.5: Test With Valid Credentials (Should Succeed)

```bash
curl -s -u student:'P@ssw0rd!' http://localhost:8080
```

**Expected Output:**
```
Authenticated OK
```

**✅ Result:** Access granted - received HTTP 200 with "Authenticated OK" message

#### Evidence - Task 1

![Task 1 Authentication - Part 1](evidence/Task%201%20Authentication%201.png)
*Figure 1.1: Creating password file and nginx configuration*

![Task 1 Authentication - Part 2](evidence/Task%201%20Authentication%202.png)
*Figure 1.2: Testing authentication - 401 without credentials, 200 with valid credentials*

#### Key Learning - Task 1

- **Authentication** verifies identity: "Are you who you claim to be?"
- HTTP Basic authentication requires valid credentials (username + password)
- Unauthenticated requests are rejected with HTTP 401
- Authenticated requests proceed and receive HTTP 200

---

### Task 2: Add a Second Factor (MFA/TOTP)

**Objective:** Implement Multi-Factor Authentication using Time-based One-Time Password (TOTP), the same mechanism used by authenticator apps like Google Authenticator.

#### Step 2.1: Generate Shared Secret

```bash
SECRET=$(head -c20 /dev/urandom | base32)
echo "Enrol this secret in an authenticator app: $SECRET"
```

**Purpose:** Create a base32-encoded random secret that serves as the shared key between server and client.

#### Step 2.2: Generate Current TOTP Code

```bash
oathtool --totp -b "$SECRET"
```

**Purpose:** Generate the current 6-digit time-based code (changes every 30 seconds).

#### Step 2.3: Validate User-Entered Code

```bash
read -p 'Enter the 6-digit code: ' CODE
[ "$CODE" = "$(oathtool --totp -b "$SECRET")" ] && echo 'MFA OK' || echo 'MFA FAILED'
```

**Purpose:** Compare user input with the expected TOTP code to validate the second factor.

#### Evidence - Task 2

![Task 2 MFA/TOTP](evidence/Task%202.png)
*Figure 2.1: MFA implementation - Secret generation, TOTP code (045367), and successful validation "MFA OK"*

**✅ Result:** MFA successfully validated with code **045367** - Output shows "MFA OK"

#### Key Learning - Task 2

- **MFA combines factors from different classes:**
  - Something you know (password)
  - Something you have (TOTP device/app)
- **TOTP codes are time-synchronized:** valid for 30-second windows
- **MFA defeats the majority of credential attacks** - the cheapest big security win
- Even if an attacker steals your password, they cannot authenticate without the second factor

---

### Task 3: Authorization - RBAC Roles

**Objective:** Demonstrate the difference between authentication and authorization. Authentication proves identity; authorization decides permissions using Role-Based Access Control (RBAC).

#### Step 3.1: Create Kubernetes Cluster

```bash
kind create cluster --name ccse-lab4
kubectl create namespace app
kubectl create serviceaccount dev -n app
```

**Purpose:** Set up a Kubernetes environment with a namespace and service account for testing RBAC.

#### Step 3.2: Create Developer Role (Read-Only Pods)

```bash
kubectl create role dev-role -n app --verb=get,list --resource=pods
kubectl create rolebinding dev-rb -n app --role=dev-role --serviceaccount=app:dev
```

**Purpose:** Create a role that allows ONLY reading pods (get, list operations). No create, update, or delete permissions.

#### Step 3.3: Test Authorization - What Can the Developer Do?

```bash
SA=system:serviceaccount:app:dev
kubectl auth can-i list pods -n app --as=$SA        # Should return: yes
kubectl auth can-i create deploy -n app --as=$SA    # Should return: no
kubectl auth can-i delete pods -n app --as=$SA      # Should return: no
```

**Purpose:** Verify that the developer role follows the principle of **least privilege** - only permitted actions are allowed.

#### Evidence - Task 3

![Task 3 RBAC Authorization](evidence/Task%203.png)
*Figure 3.1: Kubernetes cluster creation, RBAC role configuration, and authorization testing*

**✅ Results:**
- `kubectl auth can-i list pods -n app --as=$SA` → **yes** ✅
- `kubectl auth can-i create deploy -n app --as=$SA` → **no** ❌
- `kubectl auth can-i delete pods -n app --as=$SA` → **no** ❌

#### Key Learning - Task 3

- **Authentication vs Authorization:**
  - **Authentication (Task 1):** Proves WHO you are
  - **Authorization (Task 3):** Decides WHAT you can do
- **RBAC enforces least privilege:** Users get only the permissions they need, nothing more
- The developer identity is authenticated to the cluster but authorized for only limited read operations
- This prevents unauthorized actions even from legitimate users

---

### Session A Summary

**End of Session A** - Authentication service stopped:
```bash
docker stop authsvc
```

**Completed Controls:**
- ✅ Authentication with password protection (401/200 results)
- ✅ Multi-Factor Authentication with TOTP (MFA OK)
- ✅ Authorization with RBAC (yes/no/no results)

---

## Session B: Network Security & Hardening

Session B focuses on controlling **WHAT** authenticated users can reach and reducing **WHAT** an intruder could exploit through network segmentation and container hardening.

---

### Task 4: Network Segmentation (Three-Tier)

**Objective:** Implement defense in depth by separating frontend, backend, and database into isolated Docker networks. The frontend cannot reach the database directly, limiting lateral movement.

#### Step 4.1: Create Isolated Networks

```bash
docker network create frontend-net
docker network create backend-net
```

**Purpose:** Create two separate network segments for isolation.

#### Step 4.2: Deploy Three-Tier Architecture

```bash
# Database - only on backend network
docker run -d --name db --network backend-net redis:alpine

# Application - connected to BOTH networks (bridge tier)
docker run -d --name app --network backend-net nginx
docker network connect frontend-net app

# Web - only on frontend network (internet-facing)
docker run -d --name web --network frontend-net nginx
```

**Purpose:** Create a segmented architecture where:
- **db** (database) → backend-net only
- **app** (application) → backend-net + frontend-net (bridge)
- **web** (frontend) → frontend-net only

#### Step 4.3: Test Segmentation - Web → Database (Should FAIL)

```bash
docker exec web sh -c 'apk add -q curl; curl -s -m 3 db:6379 || echo BLOCKED'
```

**Expected Output:**
```
BLOCKED
```

**✅ Result:** Web tier CANNOT reach database directly - segmentation working

#### Step 4.4: Test Connectivity - App → Database (Should WORK)

```bash
docker exec app sh -c 'apk add -q curl; nc -z -w3 db 6379 && echo REACHABLE'
```

**Expected Output:**
```
REACHABLE
```

**✅ Result:** Application tier CAN reach database - expected connectivity preserved

#### Evidence - Task 4

![Task 4 Network Segmentation - Part 1](evidence/Task%204%20Network%20segmentation%201.png)
*Figure 4.1: Creating networks and deploying three-tier architecture*

![Task 4 Network Segmentation - Part 2](evidence/Task%204%20Network%20segmentation%202.png)
*Figure 4.2: Testing network segmentation - web→db BLOCKED, app→db REACHABLE*

#### Key Learning - Task 4

- **Network segmentation implements defense in depth**
- The database is unreachable from the internet-facing tier
- **Security benefit:** An attacker who compromises the web tier still cannot talk directly to the data
- **Segmentation contains lateral movement** - limits blast radius of a breach
- This mirrors cloud security architectures with public/private subnets

---

### Task 5: Firewall Rules (Default-Deny)

**Objective:** Apply host-level firewall rules using the **default-deny** principle - nothing is allowed unless explicitly permitted. This mirrors cloud security group configurations.

#### Step 5.1: Configure iptables with Default-Deny + Allow HTTPS

```bash
docker run --rm --cap-add=NET_ADMIN alpine sh -c '\
 apk add -q iptables; \
 iptables -P INPUT DROP; \
 iptables -A INPUT -p tcp --dport 443 -j ACCEPT; \
 iptables -A INPUT -i lo -j ACCEPT; \
 iptables -L INPUT -n'
```

**Configuration Breakdown:**
- `iptables -P INPUT DROP` → **Default policy: DROP** (deny all by default)
- `iptables -A INPUT -p tcp --dport 443 -j ACCEPT` → Allow ONLY port 443 (HTTPS)
- `iptables -A INPUT -i lo -j ACCEPT` → Allow loopback interface (localhost)
- `iptables -L INPUT -n` → List the configured rules

#### Evidence - Task 5

![Task 5 Firewall Rules](evidence/Task%205.png)
*Figure 5.1: iptables default-deny configuration with explicit ACCEPT rules for port 443 and loopback*

**✅ Result:** Firewall configured with:
- Chain INPUT (policy **DROP**)
- Target ACCEPT for tcp dpt:443
- Target ACCEPT for all on loopback interface

#### Key Learning - Task 5

- **Default-deny firewall policy:** Nothing is allowed unless you explicitly permit it
- This is the **security-group model** used in cloud environments (AWS Security Groups, Azure NSGs)
- **Least privilege for the network:** Only necessary ports are opened
- Reduces attack surface by blocking all unexpected traffic
- In production, you would allow only required ports (22 for SSH, 443 for HTTPS, etc.)

---

### Task 6: Container/Host Hardening

**Objective:** Reduce the attack surface by building a minimal, non-root, capability-dropped, read-only container and scanning it for vulnerabilities.

#### Step 6.1: Run a Hardened Container

```bash
docker run -d --name hardened \
 --user 1000:1000 \                  # non-root user
 --read-only \                        # read-only root filesystem
 --cap-drop=ALL \                     # drop all Linux capabilities
 --security-opt no-new-privileges \   # prevent privilege escalation
 --tmpfs /tmp \                       # writable temp directory
 nginxinc/nginx-unprivileged
```

**Hardening Measures Applied:**
1. **Non-root user (1000:1000)** - Prevents root-level compromises
2. **Read-only filesystem** - Prevents malware from modifying system files
3. **All capabilities dropped** - Removes dangerous Linux kernel capabilities
4. **No new privileges** - Prevents privilege escalation attacks
5. **Tmpfs for /tmp** - Provides writable space for temporary files only

#### Step 6.2: Verify Hardening Configuration

```bash
docker inspect hardened --format 'User={{.Config.User}} ReadOnly={{.HostConfig.ReadonlyRootfs}}'
```

**Expected Output:**
```
User=1000:1000 ReadOnly=true
```

**✅ Result:** Container verified as non-root with read-only filesystem

#### Step 6.3: Scan Image for Vulnerabilities

```bash
docker run --rm aquasec/trivy image --severity HIGH,CRITICAL nginx:alpine | head -20
```

**Purpose:** Use Trivy to scan the nginx:alpine image for known HIGH and CRITICAL severity vulnerabilities.

#### Evidence - Task 6

![Task 6 Container Hardening - Part 1](evidence/Task%206.png)
*Figure 6.1: Running hardened container with security configurations*

![Task 6 Container Hardening - Part 2](evidence/Task%206%20container%202.png)
*Figure 6.2: Verifying hardening - User=1000:1000, ReadOnly=true*

![Task 6 Container Hardening - Part 3](evidence/Task%206%20container%203.png)
*Figure 6.3: Trivy vulnerability scan results for nginx:alpine*

**Scan Results Summary:**
- **Target:** nginx:alpine (alpine 3.24.1)
- **Total Vulnerabilities:** 2 (HIGH: 2, CRITICAL: 0)
- **Status:** Relatively secure base image with minimal vulnerabilities

#### Key Learning - Task 6

**Three Hardening Measures and the Attacks They Blunt:**

1. **Non-root User (--user 1000:1000)**
   - **Attack Blunted:** Root-level container escape and privilege escalation
   - Even if an attacker compromises the container, they have limited user privileges

2. **Read-only Filesystem (--read-only)**
   - **Attack Blunted:** Malware installation, backdoor deployment, system file modification
   - Attackers cannot persist malicious code or modify binaries

3. **Dropped Capabilities (--cap-drop=ALL)**
   - **Attack Blunted:** Kernel-level exploits, network manipulation, system call abuse
   - Removes dangerous Linux capabilities that could be exploited

**Additional Security Measures:**
4. **No New Privileges** - Prevents setuid binary exploits
5. **Minimal Base Image** - Reduces attack surface by limiting installed packages
6. **Vulnerability Scanning** - Identifies known CVEs before deployment

---

## Verification Commands

After completing all tasks, the following verification commands confirm successful implementation:

### Verify RBAC Configuration

```bash
kubectl get rolebinding dev-rb -n app -o yaml
```

### Verify Container Hardening

```bash
docker inspect hardened --format '{{json .HostConfig.CapDrop}}'
```

#### Evidence - Verification

![Verification Commands](evidence/Verification%20Command.png)
*Figure 7.1: Verification of RBAC role binding and container capability drops*

**✅ Results:**
- Role binding `dev-rb` exists in namespace `app`
- Resource version confirmed: "616"
- Role reference: `dev-role`
- Subject: ServiceAccount `dev` in namespace `app`
- Capabilities dropped: `["ALL"]` confirmed

---

## Short-Answer Questions

### Q1. Explain the difference between authentication and authorization using Tasks 1 and 3.

**Answer:**

**Authentication (Task 1)** answers the question: *"WHO are you?"*
- In Task 1, we implemented HTTP Basic authentication where users must prove their identity with credentials (username: `student`, password: `P@ssw0rd!`)
- Without valid credentials, the system rejects access with HTTP 401 (Unauthorized)
- With valid credentials, the system recognizes the user and allows access (HTTP 200)
- Authentication is about **identity verification**

**Authorization (Task 3)** answers the question: *"WHAT are you allowed to do?"*
- In Task 3, we implemented RBAC where the authenticated `dev` service account has limited permissions
- The system allows the developer to LIST pods (permitted action → yes)
- The system denies the developer from CREATE deployments or DELETE pods (unauthorized actions → no)
- Authorization is about **permission enforcement**

**Key Difference:**
- Authentication happens **first** - you must prove who you are
- Authorization happens **second** - the system decides what you can do based on your identity
- You can be authenticated but still unauthorized for certain actions

---

### Q2. Why is MFA so effective, and which attacks does it defeat?

**Answer:**

**Why MFA is Effective:**

MFA (Multi-Factor Authentication) combines factors from **different security classes**:
- **Something you know:** Password or PIN
- **Something you have:** TOTP device, authenticator app, hardware token
- (Optionally) **Something you are:** Biometrics

**Attacks MFA Defeats:**

1. **Credential Theft / Password Breaches**
   - Stolen passwords are useless without the second factor
   - Database breaches that expose passwords don't compromise accounts

2. **Phishing Attacks**
   - Even if a user enters their password on a fake site, attackers can't access the TOTP device
   - Time-limited codes expire quickly (30-second windows)

3. **Brute Force Attacks**
   - Guessing the password isn't enough - attackers would also need the TOTP code
   - TOTP codes change every 30 seconds, making brute force impractical

4. **Credential Stuffing**
   - Reused passwords from other breaches won't work without the second factor
   - Each account requires its own unique TOTP secret

5. **Keylogger Attacks**
   - Even if malware captures the password, it can't capture the dynamically generated TOTP codes

**Security Impact:**
MFA is called **"the cheapest big security win"** because it defeats the majority of credential-based attacks with relatively simple implementation. According to industry research, MFA blocks over 99% of automated attacks.

---

### Q3. How does network segmentation limit the damage of a compromised web server?

**Answer:**

**Network Segmentation Defense in Depth:**

In Task 4, we implemented a three-tier architecture with isolated networks:

**Architecture:**
- **Frontend Network:** Web server (internet-facing)
- **Backend Network:** Database server (internal only)
- **Bridge:** Application server (both networks)

**How It Limits Damage:**

1. **Prevents Direct Database Access**
   - The web server cannot communicate directly with the database (we verified: BLOCKED)
   - An attacker who compromises the web server cannot directly query or dump the database
   - The attacker cannot steal sensitive data directly

2. **Contains Lateral Movement**
   - Segmentation creates **security boundaries** within the infrastructure
   - Compromise of one tier doesn't automatically mean compromise of other tiers
   - The attacker must breach multiple security layers to move deeper

3. **Reduces Blast Radius**
   - The impact of a breach is limited to the compromised network segment
   - Critical data assets (database) remain protected behind additional network barriers
   - Attack surface is minimized

4. **Forces Traffic Through Controlled Paths**
   - All database access must go through the application tier
   - The application tier can implement additional security controls (authentication, input validation, logging)
   - Provides a **choke point** for monitoring and access control

**Real-World Analogy:**
This mirrors cloud architectures with:
- **Public subnets** for internet-facing resources
- **Private subnets** for databases and sensitive services
- **Application tier** in between with strict security group rules

**Security Principle:** Defense in depth - multiple layers of security controls so a single point of failure doesn't compromise the entire system.

---

### Q4. What does a default-deny firewall policy achieve, and how does it relate to cloud security groups?

**Answer:**

**Default-Deny Firewall Policy:**

A **default-deny** policy means:
- **Default rule:** DENY/DROP all traffic
- **Exception rules:** Explicitly ALLOW only necessary traffic
- Security posture: *"Nothing is permitted unless specifically authorized"*

**What It Achieves:**

1. **Implements Least Privilege for Network Access**
   - Only required ports and protocols are open
   - All unexpected traffic is automatically blocked
   - Reduces attack surface dramatically

2. **Prevents Unknown/Unexpected Connections**
   - New vulnerabilities or services can't be exploited if ports aren't open
   - Zero-day attacks targeting unexpected ports are blocked by default
   - Misconfigured services aren't accidentally exposed

3. **Explicit Security Posture**
   - Security is the default state, not an afterthought
   - Every allowed connection is documented and intentional
   - Easier to audit: "What is allowed?" vs. "What is blocked?"

4. **Fails Secure**
   - If rules are misconfigured, the system defaults to blocking
   - Errors don't accidentally open access

**Relationship to Cloud Security Groups:**

Cloud security groups (AWS, Azure, GCP) implement the **exact same default-deny model**:

| Local Firewall (iptables) | Cloud Security Group |
|---------------------------|---------------------|
| `iptables -P INPUT DROP` | Default: Deny all inbound |
| `iptables -A INPUT -p tcp --dport 443 -j ACCEPT` | Inbound rule: Allow TCP 443 |
| `iptables -A INPUT -i lo -j ACCEPT` | Allow within security group |

**Cloud Security Group Example:**
```
Default: Deny all inbound traffic
Rule 1: Allow TCP port 443 from 0.0.0.0/0 (HTTPS)
Rule 2: Allow TCP port 22 from 10.0.0.0/8 (SSH from corporate network)
Result: Only HTTPS and SSH from specific sources are allowed
```

**Security Best Practice:**
- Start with deny-all
- Add only necessary allow rules
- Document each exception
- Review and remove unused rules regularly

This approach is foundational to **Zero Trust** security models: "Never trust, always verify."

---

### Q5. List the hardening measures you applied and the attack surface each one removes.

**Answer:**

**Container Hardening Measures and Attack Surface Reduction:**

| # | Hardening Measure | Attack Surface Removed | Security Benefit |
|---|------------------|----------------------|-----------------|
| 1 | **Non-root User** (`--user 1000:1000`) | • Root-level container escapes<br>• Privilege escalation exploits<br>• System-wide modifications | Even if compromised, attacker has limited user privileges; cannot modify system files or access root-only resources |
| 2 | **Read-only Filesystem** (`--read-only`) | • Malware installation<br>• Backdoor persistence<br>• Binary modification<br>• Log tampering | Attackers cannot write malicious code to disk; prevents persistent threats; limits what attackers can do even with access |
| 3 | **Drop All Capabilities** (`--cap-drop=ALL`) | • Kernel exploitation<br>• Network manipulation<br>• System call abuse<br>• Advanced container escapes | Removes dangerous Linux capabilities (CAP_NET_RAW, CAP_SYS_ADMIN, etc.); prevents low-level system manipulation |
| 4 | **No New Privileges** (`--security-opt no-new-privileges`) | • Setuid binary exploits<br>• Privilege escalation via file permissions<br>• Sudo exploitation | Prevents processes from gaining more privileges than the parent; blocks common privilege escalation techniques |
| 5 | **Minimal Base Image** (nginx-unprivileged) | • Unnecessary packages/tools<br>• Additional vulnerabilities<br>• Attack utilities (shells, compilers) | Fewer installed packages = fewer vulnerabilities; removes tools attackers could use; reduces CVE exposure |
| 6 | **Tmpfs for /tmp** (`--tmpfs /tmp`) | • Persistent malicious files<br>• Storage-based attacks | Temporary files are stored in memory only; cleared on container restart; provides necessary writable space without persistence |

**Combined Security Impact:**

These measures implement **defense in depth** through **least privilege** principles:

1. **Reduces the initial attack surface** - Fewer vulnerabilities to exploit
2. **Limits what an attacker can do** - Even successful exploits have limited impact
3. **Prevents lateral movement** - Difficult to move from container to host
4. **Prevents persistence** - Attackers can't maintain long-term access
5. **Facilitates detection** - Unusual behavior is more noticeable with restricted capabilities

**Verification Results:**
- Container runs as UID 1000 (non-root) ✅
- Root filesystem is read-only ✅
- All capabilities dropped ✅
- Trivy scan: Only 2 HIGH vulnerabilities (0 CRITICAL) ✅

**Security Maturity:** This configuration represents **production-grade** container security suitable for sensitive workloads.

---

## Security Best-Practices Checklist

✅ **Service requires authentication** (unauthenticated requests rejected)
- Implemented HTTP Basic authentication
- Unauthenticated requests return 401
- Verified with curl tests

✅ **MFA / second factor implemented and validated**
- TOTP-based MFA configured
- Secret generated and tested
- Validation successful (MFA OK)

✅ **Authorization enforced by RBAC** (least privilege; unauthorized actions denied)
- Kubernetes RBAC roles created
- Developer can list pods (authorized)
- Developer cannot create/delete (unauthorized)

✅ **Network segmented** so the data tier is unreachable from the front tier
- Three-tier architecture implemented
- Web → Database: BLOCKED ✅
- App → Database: REACHABLE ✅

✅ **Default-deny firewall** with explicit allow rules
- iptables configured with DROP policy
- Only port 443 and loopback explicitly allowed
- All other traffic blocked by default

✅ **Container hardened:** non-root, minimal, capabilities dropped, read-only; image scanned
- Non-root user (1000:1000) ✅
- Read-only filesystem ✅
- All capabilities dropped ✅
- No new privileges ✅
- Trivy vulnerability scan completed ✅

---

## Cleanup & Teardown

After completing the lab and capturing all evidence, clean up resources:

```bash
# Remove Docker containers
docker rm -f authsvc db app web hardened 2>/dev/null

# Remove Docker networks
docker network rm frontend-net backend-net 2>/dev/null

# Delete Kubernetes cluster
kind delete cluster --name ccse-lab4
```

#### Evidence - Cleanup

![Cleanup & Teardown](evidence/Cleanup%20&%20Teardown.png)
*Figure 8.1: Resource cleanup - containers, networks, and Kubernetes cluster removed*

**✅ Cleanup Completed:**
- All containers removed: authsvc, db, app, web, hardened
- Networks removed: frontend-net, backend-net
- Kubernetes cluster deleted: ccse-lab4
- System returned to clean state

---

## Conclusion

### Lab Summary

This lab successfully demonstrated critical cloud security controls across two key domains:

**Authentication & Authorization (Session A):**
- Implemented password-based authentication with proper rejection of unauthorized access
- Added multi-factor authentication (MFA) using TOTP for enhanced security
- Configured RBAC to enforce least-privilege authorization policies

**Network Security & Hardening (Session B):**
- Deployed network segmentation to isolate sensitive data from internet-facing tiers
- Configured default-deny firewall rules following the security-group model
- Hardened containers using multiple security measures and validated with vulnerability scanning

### Key Security Principles Applied

1. **Identity is the Perimeter** - Every control asks: "Who are you?" and "What are you allowed to do?"
2. **Defense in Depth** - Multiple layers of security controls
3. **Least Privilege** - Grant only the minimum necessary permissions
4. **Default-Deny** - Block everything except explicitly allowed traffic
5. **Reduce Attack Surface** - Minimize vulnerabilities through hardening

### Skills Developed

- ✅ Implementing authentication and authorization controls
- ✅ Configuring multi-factor authentication (MFA/TOTP)
- ✅ Designing segmented network architectures
- ✅ Configuring host-based firewalls with default-deny policies
- ✅ Hardening container configurations for production workloads
- ✅ Scanning images for vulnerabilities using Trivy
- ✅ Applying least privilege across compute, network, and storage

### Real-World Applications

These security controls directly map to production cloud environments:
- AWS Security Groups, Network ACLs, and VPCs
- Azure Network Security Groups and Virtual Networks
- Kubernetes RBAC and Network Policies
- Container security in ECS, EKS, AKS, GKE
- Zero Trust security architectures

### Course Learning Outcome Achievement

**CLO2 - Construct secure cloud operations that safeguard data integrity:** ✅ **ACHIEVED**

This lab successfully demonstrated the construction of secure cloud operations through:
- Identity and access management (authentication + authorization)
- Network security controls (segmentation + firewall)
- Secure compute practices (container hardening + vulnerability management)

---

## References

- Course lectures — Week 5 (Access Control), Week 9 (Network Security patterns)
- Docker security — [docs.docker.com/engine/security](https://docs.docker.com/engine/security)
- CIS Docker / Kubernetes Benchmarks — [www.cisecurity.org](https://www.cisecurity.org)
- CSA Security Guidance v5 — Infrastructure & Networking; IAM
- OWASP Container Security Top 10
- NIST SP 800-190: Application Container Security Guide

---

**Lab Completed By:** Surya  
**Date:** August 30, 2026  
**Course:** IKB42603 Cloud Computing Security Essentials  
**Institution:** UniKL MIIT  
**Instructor:** Prof. Dr. Shahrulniza Musa

---

*This report demonstrates comprehensive understanding and practical implementation of access control and network security principles in cloud computing environments.*

