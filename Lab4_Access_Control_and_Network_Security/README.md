# Lab 4: Access Control & Network Security Report

## Student Information

- **Name:** Surya Giri A/L Shanker
- **Student ID:** 52215124335
- **Course:** IKB42603 Cloud Computing Security Essentials
- **Lab Task:** Lab 4 - Access Control & Network Security
- **Lecturer Name:** Prof. Dr. Shahrulniza Musa

---

## Overview

This report documents the implementation and verification of access control and network security controls for cloud computing environments, demonstrating how authentication, authorization, network segmentation, firewall rules, and container hardening work together as a defense-in-depth strategy. The lab was conducted in two sessions over two weeks. Session A focused on controlling WHO gets in by implementing HTTP Basic authentication for password-protected services, adding multi-factor authentication (MFA) using time-based one-time passwords (TOTP) for second-factor verification, and configuring Kubernetes Role-Based Access Control (RBAC) to enforce authorization policies that determine what authenticated users are allowed to do. Session B focused on controlling WHAT users can reach and reducing WHAT attackers could exploit by implementing three-tier network segmentation to isolate sensitive services, configuring default-deny firewall rules that permit only necessary traffic, and hardening container deployments with non-root users, read-only filesystems, dropped capabilities, and vulnerability scanning. The purpose of this comprehensive lab was to develop practical understanding that identity is the security perimeter—every control ultimately asks "are you who you claim to be?" and "are you allowed to do this?"—and that defense-in-depth requires multiple layers of security controls across access control, network security, and host hardening. Both sessions were executed using command-line tools (Docker for containers and authentication services, kubectl and kind for Kubernetes RBAC, iptables for firewall rules, Trivy for vulnerability scanning) in a Windows PowerShell environment, and each task was systematically documented with terminal commands and screenshots as evidence of successful implementation.

**Security Principle:** *Identity is the perimeter.* Almost every control in this lab ultimately asks the same two questions: are you who you claim, and are you allowed to do this?

---

## Objectives

The objectives of this lab across both sessions are:

**Session A Objectives (Week 7 - Authentication & Authorization):**

- Implement HTTP Basic authentication to protect a web service with username and password credentials.
- Demonstrate that unauthenticated requests are rejected with HTTP 401 Unauthorized responses.
- Verify that authenticated requests with valid credentials receive HTTP 200 OK responses.
- Generate a time-based one-time password (TOTP) shared secret for multi-factor authentication.
- Implement MFA validation by comparing user-entered codes with expected TOTP values.
- Understand that MFA combines something you know (password) with something you have (TOTP device).
- Create a Kubernetes cluster with namespaces and service accounts for RBAC testing.
- Configure role-based access control (RBAC) with limited permissions (read-only pods).
- Test authorization enforcement using kubectl auth can-i commands to verify allowed and denied operations.
- Distinguish between authentication (proving identity) and authorization (enforcing permissions).

**Session B Objectives (Week 8 - Network Security & Hardening):**

- Create isolated Docker networks to implement three-tier architecture segmentation.
- Deploy frontend, backend, and database tiers on separate networks with controlled connectivity.
- Demonstrate that network segmentation prevents direct communication between isolated tiers.
- Verify that segmentation contains lateral movement by blocking web-to-database connections.
- Configure host-level firewall rules using iptables with default-deny policy.
- Implement explicit allow rules for required ports (HTTPS) while blocking all other traffic.
- Understand the security-group model where nothing is allowed unless explicitly permitted.
- Deploy hardened containers with non-root users, read-only filesystems, and dropped capabilities.
- Implement security options including no-new-privileges and tmpfs for temporary storage.
- Scan container images for vulnerabilities using Trivy to identify HIGH and CRITICAL severity issues.
- Verify hardening measures by inspecting container configuration and security settings.

**Common Objectives:**

- Document all procedures clearly using terminal commands and screenshots as evidence of implementation.
- Understand that identity is the security perimeter and every control validates identity and permissions.
- Recognize that defense-in-depth requires multiple layers of security controls.
- Apply the principle of least privilege across access control, network segmentation, and container security.
- Understand how access control mechanisms integrate with network security and host hardening.

---

## Learning Outcomes

By completing this lab across both sessions, the student should be able to:

**Session A Outcomes (Authentication & Authorization):**

- Explain the difference between authentication (verifying identity) and authorization (enforcing permissions).
- Implement HTTP Basic authentication using password files and web server configuration.
- Use oathtool or authenticator apps to generate and validate time-based one-time passwords (TOTP).
- Understand why MFA is considered "the cheapest big security win" by defeating credential-based attacks.
- Configure Kubernetes RBAC roles and role bindings to enforce least-privilege access control.
- Use kubectl auth can-i commands to test authorization policies and verify permission enforcement.
- Recognize that authentication proves WHO you are while authorization determines WHAT you can do.
- Understand that even authenticated users should have limited permissions based on their role.

**Session B Outcomes (Network Security & Hardening):**

- Design and implement three-tier network architectures with isolated network segments.
- Understand how network segmentation limits blast radius and contains lateral movement during breaches.
- Configure default-deny firewall policies with explicit allow rules for required traffic.
- Recognize the relationship between host firewalls and cloud security groups.
- Implement container hardening measures including non-root users, read-only filesystems, and capability drops.
- Scan container images for vulnerabilities and interpret severity levels (HIGH, CRITICAL).
- Explain how each hardening measure reduces specific attack surfaces and mitigates threats.
- Apply defense-in-depth principles by layering multiple security controls.

**Common Outcomes:**

- Recognize that security controls work together to provide comprehensive protection.
- Understand that compromising authentication doesn't automatically grant full system access if authorization is properly configured.
- Apply the principle of least privilege across identity, network, and compute layers.
- Document technical security procedures clearly using terminal commands and visual evidence.
- Understand how access control and network security complement encryption and integrity controls from previous labs.

---

## Environment and Prerequisites

The lab was conducted on a Windows environment with PowerShell terminal and Docker installed. The following tools and conditions were required before starting the lab:

**Session A Prerequisites (Authentication & Authorization):**

- Docker installed and running to support containerized authentication services and web servers.
- kubectl and kind installed for creating local Kubernetes clusters and configuring RBAC.
- oathtool installed for generating and validating TOTP codes (or any authenticator app).
- Basic understanding of authentication concepts including passwords, usernames, and session management.
- Basic understanding of authorization concepts including roles, permissions, and access control policies.
- Terminal or command-line interface with PowerShell access for executing Docker and kubectl commands.
- Network connectivity for pulling Docker images (nginx, httpd) and Kubernetes node images.

**Session B Prerequisites (Network Security & Hardening):**

- Docker networking features available for creating custom bridge networks and network isolation.
- iptables or equivalent firewall tools available in Docker containers for firewall rule configuration.
- Trivy container scanner installed for vulnerability assessment of container images.
- Understanding of network concepts including network isolation, subnets, and connectivity rules.
- Understanding of container security concepts including users, capabilities, and filesystem permissions.
- Files and configurations from Session A for continuity and verification tasks (optional).

**Common Prerequisites:**

- Administrative privileges on the local system for managing Docker containers and networks.
- Sufficient system resources (CPU, memory, disk) to run multiple Docker containers simultaneously.
- Internet connectivity for downloading Docker images, Kubernetes components, and security tools.
- Basic familiarity with command-line operations including Docker commands, kubectl commands, and shell scripting.
- Understanding of file permissions and the importance of protecting credentials and configuration files.

**Security Note:** This lab uses local development tools (kind clusters, self-signed certificates, basic authentication) that are appropriate for learning and testing but would require additional hardening for production deployments. Production systems should use managed Kubernetes services (EKS, AKS, GKE) with proper IAM integration, certificate authorities for TLS, enterprise authentication systems (LDAP, SAML, OIDC), network policies, pod security policies, comprehensive audit logging, and regular security assessments.

---

## Session A (Week 7) — Authentication & Authorization

Session A focuses on controlling **WHO** gets access to systems and **WHAT** they are allowed to do once authenticated. Authentication verifies identity using credentials (passwords, tokens, biometrics), while authorization enforces permissions based on roles and policies. These two concepts work together to provide secure access control—authentication without authorization would allow any authenticated user to perform any action, while authorization without proper authentication would be meaningless. Understanding this distinction is critical for implementing defense-in-depth security strategies in cloud environments.

### Task 1 — Authentication: Password-Protected Service

HTTP Basic authentication is a simple authentication mechanism built into the HTTP protocol where clients send username and password credentials with each request. The credentials are Base64-encoded (not encrypted) in the HTTP Authorization header, which is why HTTPS/TLS is essential when using Basic authentication in production to prevent credential theft through eavesdropping. In this task, nginx web server was configured to require valid credentials before allowing access to protected resources, demonstrating that authentication controls WHO can access services by rejecting unauthenticated requests and accepting authenticated requests.

#### Purpose

- Create an htpasswd file containing hashed passwords for user authentication.
- Configure nginx to require HTTP Basic authentication for all requests to protected resources.
- Deploy an authentication-protected web service in a Docker container.
- Demonstrate that requests without credentials are rejected with HTTP 401 Unauthorized.
- Verify that requests with valid credentials receive HTTP 200 OK responses.
- Understand the key-distribution problem: how do users securely receive and store passwords?

#### Terminal Commands

```bash
# Create a password file (user: student)
docker run --rm httpd:alpine htpasswd -nbB student 'P@ssw0rd!' > htpasswd.txt

# Serve a page that requires authentication
cat > default.conf <<'EOF'
server { listen 80;
 location / { auth_basic "Restricted";
 auth_basic_user_file /etc/nginx/.htpasswd;
 return 200 'Authenticated OK\n'; } }
EOF

docker run --rm -d --name authsvc -p 8080:80 \
 -v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf \
 -v $(pwd)/htpasswd.txt:/etc/nginx/.htpasswd nginx

curl -s -o /dev/null -w 'no-creds: %{http_code}\n' http://localhost:8080
curl -s -u student:'P@ssw0rd!' http://localhost:8080
```

#### Explanation of the Commands

- `docker run --rm httpd:alpine htpasswd -nbB student 'P@ssw0rd!'` uses Apache's htpasswd utility in the httpd Alpine container to generate a bcrypt-hashed password entry for username "student" with password "P@ssw0rd!"—the `-n` flag outputs to stdout instead of a file, `-b` accepts the password from the command line, and `-B` uses bcrypt hashing for stronger security than MD5.
- `> htpasswd.txt` redirects the htpasswd output to a file that nginx will use for authentication, storing the username and bcrypt hash (the password itself is never stored in plaintext).
- The `cat > default.conf <<'EOF'` heredoc creates an nginx configuration file that defines a server listening on port 80 with Basic authentication enabled—the `auth_basic "Restricted"` directive enables authentication with a realm message, and `auth_basic_user_file /etc/nginx/.htpasswd` specifies where nginx should find the password file.
- `return 200 'Authenticated OK\n'` is a simple response that nginx sends when authentication succeeds, demonstrating that the protected resource was accessed successfully without setting up a full web application.
- `docker run --rm -d --name authsvc -p 8080:80` starts an nginx container in detached mode, mapping host port 8080 to container port 80, with `--rm` ensuring automatic cleanup when stopped.
- `-v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf` mounts the nginx configuration file into the container where nginx reads additional configuration—this volume mount allows us to customize nginx behavior without rebuilding the image.
- `-v $(pwd)/htpasswd.txt:/etc/nginx/.htpasswd` mounts the password file into the container at the path specified in the nginx configuration for authentication validation.
- `curl -s -o /dev/null -w 'no-creds: %{http_code}\n' http://localhost:8080` makes an HTTP request without credentials, with `-s` for silent mode, `-o /dev/null` discarding response body, and `-w` printing only the HTTP status code—this should return 401 Unauthorized demonstrating authentication enforcement.
- `curl -s -u student:'P@ssw0rd!' http://localhost:8080` makes an HTTP request with Basic authentication credentials using the `-u` flag, which should return 200 OK and display "Authenticated OK" proving that valid credentials grant access.

#### Evidence

![Task 1 Authentication - Setup](evidence/Task%201%20Authentication%201.png)

The screenshot shows the creation of the htpasswd file and nginx configuration, followed by the Docker container startup, confirming that the authentication service was deployed successfully.

![Task 1 Authentication - Testing](evidence/Task%201%20Authentication%202.png)

The screenshot demonstrates the authentication testing with curl showing "no-creds: 401" for the unauthenticated request and "Authenticated OK" for the authenticated request, proving that authentication enforcement works correctly.

#### Notes

The authentication task successfully demonstrated that HTTP Basic authentication provides identity verification by rejecting unauthorized access. However, this also reveals challenges: passwords must be transmitted with every request (requiring HTTPS to prevent interception), users must remember complex passwords, and passwords can be stolen through phishing or keyloggers. These limitations motivate the need for multi-factor authentication (Task 2) which adds a second verification factor that attackers cannot easily steal or replicate.

---

### Task 2 — Add a Second Factor (MFA / TOTP)

Multi-factor authentication (MFA) significantly strengthens security by requiring two or more independent authentication factors from different categories: something you know (password), something you have (phone, hardware token), or something you are (biometric). Time-based One-Time Password (TOTP) is a common MFA implementation that generates 6-digit codes using a shared secret and the current time, with codes changing every 30 seconds. TOTP is standardized in RFC 6238 and is used by Google Authenticator, Microsoft Authenticator, and many other authenticator apps. The security comes from the fact that even if an attacker steals your password, they still need access to your physical device that generates the time-based codes, making credential theft attacks much harder to execute successfully.

#### Purpose

- Generate a cryptographically random shared secret for TOTP authentication.
- Use oathtool to generate time-based one-time passwords from the shared secret.
- Implement MFA validation by comparing user-entered codes with expected values.
- Understand that TOTP codes are time-synchronized and expire every 30 seconds.
- Demonstrate that MFA defeats credential theft attacks even when passwords are compromised.

#### Terminal Commands

```bash
# Create a shared secret (base32) and generate the current 6-digit code
SECRET=$(head -c20 /dev/urandom | base32)
echo "Enrol this secret in an authenticator app: $SECRET"
oathtool --totp -b "$SECRET"

# Validate a code the user types (compare to the expected value)
read -p 'Enter the 6-digit code: ' CODE
[ "$CODE" = "$(oathtool --totp -b "$SECRET")" ] && echo 'MFA OK' || echo 'MFA FAILED'
```

#### Explanation of the Commands

- `SECRET=$(head -c20 /dev/urandom | base32)` generates a 20-byte cryptographically random secret by reading from `/dev/urandom` (Linux's cryptographic random number generator) and encoding it in base32 format, which is the standard encoding for TOTP shared secrets that can be safely typed or transmitted as text.
- `echo "Enrol this secret in an authenticator app: $SECRET"` displays the shared secret that would be entered into an authenticator app (Google Authenticator, Authy, Microsoft Authenticator) or encoded as a QR code for mobile device scanning—this secret must be kept confidential because anyone who possesses it can generate valid TOTP codes.
- `oathtool --totp -b "$SECRET"` generates the current 6-digit TOTP code using the shared secret, with `--totp` specifying time-based OTP mode and `-b` indicating the secret is base32-encoded—this command computes the same code that an authenticator app would display.
- The TOTP algorithm uses HMAC-SHA1 with the shared secret and the current Unix timestamp divided by 30 seconds, ensuring that both the server and the client device generate the same code when their clocks are synchronized.
- `read -p 'Enter the 6-digit code: ' CODE` prompts the user to enter the 6-digit code they see in their authenticator app or generated with oathtool, simulating the user authentication flow where they type their second factor.
- `[ "$CODE" = "$(oathtool --totp -b "$SECRET")" ]` compares the user-entered code with the freshly computed expected code, implementing TOTP validation—the comparison must happen within the same 30-second time window or validation fails.
- `&& echo 'MFA OK' || echo 'MFA FAILED'` outputs the result of the validation, with "MFA OK" indicating successful second-factor verification and "MFA FAILED" indicating an incorrect code (wrong secret, clock skew, or expired code).

#### Evidence

![Task 2 MFA/TOTP](evidence/Task%202.png)

The screenshot demonstrates the generation of the TOTP shared secret, the current 6-digit code (045367), user input of the code, and successful validation with "MFA OK" output, proving that time-based multi-factor authentication works correctly.

#### Notes

The MFA task demonstrated why TOTP is considered "the cheapest big security win" in cybersecurity—it defeats the vast majority of credential-based attacks (phishing, password reuse, brute force, keyloggers) with relatively simple implementation. According to industry research, MFA blocks over 99% of automated credential stuffing attacks because attackers cannot generate valid TOTP codes without physical access to the victim's device. Production implementations should consider TOTP code validation windows (allowing previous/next time periods for clock skew), rate limiting to prevent code guessing, backup codes for device loss, and user education about keeping the shared secret secure.

---

### Task 3 — Authorization: RBAC Roles

Role-Based Access Control (RBAC) is an authorization model where permissions are assigned to roles rather than individual users, and users are assigned to roles based on their job function. Kubernetes RBAC uses four API resources: Role (defines permissions in a namespace), ClusterRole (defines cluster-wide permissions), RoleBinding (grants Role permissions to users/service accounts in a namespace), and ClusterRoleBinding (grants ClusterRole permissions cluster-wide). RBAC implements the principle of least privilege by granting users only the minimum permissions necessary to perform their job functions. In this task, a developer role was created with read-only access to pods, demonstrating that even authenticated users should have limited permissions based on their responsibilities.

#### Purpose

- Create a Kubernetes cluster using kind for RBAC testing and configuration.
- Create a namespace and service account to represent a developer user identity.
- Define a Role with limited permissions (get, list pods only—no create, update, delete).
- Bind the Role to the service account using a RoleBinding to grant permissions.
- Test authorization using kubectl auth can-i commands to verify allowed and denied operations.
- Understand that authentication proves identity while authorization enforces permissions.

#### Terminal Commands

```bash
kind create cluster --name ccse-lab4
kubectl create namespace app
kubectl create serviceaccount dev -n app

# Developer may only read pods
kubectl create role dev-role -n app --verb=get,list --resource=pods
kubectl create rolebinding dev-rb -n app --role=dev-role --serviceaccount=app:dev

SA=system:serviceaccount:app:dev
kubectl auth can-i list pods -n app --as=$SA      # yes
kubectl auth can-i create deploy -n app --as=$SA  # no
kubectl auth can-i delete pods -n app --as=$SA    # no
```

#### Explanation of the Commands

- `kind create cluster --name ccse-lab4` creates a local Kubernetes cluster using kind (Kubernetes in Docker), which runs Kubernetes components in Docker containers for development and testing purposes—this provides a full-featured Kubernetes API without requiring cloud resources or complex setup.
- `kubectl create namespace app` creates a new namespace called "app" which provides logical isolation and scope for resources—namespaces allow teams to share a cluster while maintaining separate environments and access controls.
- `kubectl create serviceaccount dev -n app` creates a ServiceAccount named "dev" in the app namespace, which represents a developer identity that applications or users can authenticate as—service accounts are Kubernetes-native identity objects used for RBAC authorization.
- `kubectl create role dev-role -n app --verb=get,list --resource=pods` creates a Role that defines a set of permissions: the verbs `get` and `list` allow reading pod information, but notably excludes `create`, `update`, `delete`, `patch` and other write operations—this implements least privilege by granting only read access.
- `kubectl create rolebinding dev-rb -n app --role=dev-role --serviceaccount=app:dev` creates a RoleBinding that connects the Role to the ServiceAccount, effectively granting the "dev" service account the permissions defined in "dev-role"—without this binding, the role would exist but grant no actual permissions.
- `SA=system:serviceaccount:app:dev` stores the full ServiceAccount identity string in a shell variable for easier reuse in authorization tests—the format `system:serviceaccount:<namespace>:<name>` is how Kubernetes internally identifies service accounts.
- `kubectl auth can-i list pods -n app --as=$SA` tests whether the dev service account has permission to list pods in the app namespace using the `--as` flag for impersonation—this should return "yes" because listing pods is explicitly allowed by the dev-role.
- `kubectl auth can-i create deploy -n app --as=$SA` tests whether the dev service account can create deployments—this should return "no" because the role only grants permissions for pods, not deployments, demonstrating resource-level authorization.
- `kubectl auth can-i delete pods -n app --as=$SA` tests whether the dev service account can delete pods—this should return "no" because the role only allows `get` and `list` verbs, not `delete`, demonstrating operation-level authorization enforcement.

#### Evidence

![Task 3 RBAC Authorization](evidence/Task%203.png)

The screenshot shows the Kubernetes cluster creation, namespace and service account setup, role and rolebinding configuration, and authorization testing with three can-i commands returning "yes" for list pods and "no" for create deployments and delete pods, confirming that RBAC authorization enforcement works correctly.

#### Notes

The RBAC task demonstrated the critical distinction between authentication and authorization: the dev service account is authenticated to the Kubernetes API (proven by successful kubectl commands), but authorization policies limit what actions it can perform. This implements defense-in-depth where compromising one account doesn't automatically grant full cluster access. Production Kubernetes clusters should use RBAC extensively with roles for different job functions (developers, operators, auditors), namespace-level isolation for teams and applications, ClusterRoles for cluster-wide resources, regular permission audits, and integration with enterprise identity providers (LDAP, Active Directory, OIDC) rather than static service accounts.

**End of Session A** - Authentication service stopped:
```bash
docker stop authsvc
```

Session A successfully demonstrated controlling WHO gets access (authentication with passwords and MFA) and WHAT they can do (authorization with RBAC). Session B will build on these access controls by implementing network security and host hardening to control WHAT resources can be reached and WHAT capabilities attackers could exploit.

---

## Session B (Week 8) — Network Security & Hardening

Session B transitions from access control (WHO and WHAT permissions) to network security and host hardening (WHAT resources can be reached and WHAT capabilities exist). While Session A controlled access through authentication and authorization, Session B implements defense-in-depth by adding network-level isolation to limit lateral movement after authentication and host-level hardening to reduce the attack surface available to authenticated users or compromised services. The principle is that even if an attacker breaches authentication controls, network segmentation prevents them from reaching sensitive resources, and container hardening limits what they can do with compromised containers.

### Task 4 — Network Segmentation (Three-Tier)

Network segmentation is a security architecture pattern that divides a network into isolated segments with controlled connectivity between them. Three-tier architecture is a common segmentation pattern with distinct frontend (web/presentation), backend (application logic), and database (data storage) tiers, where each tier can only communicate with its adjacent tier—frontend cannot directly reach database, implementing the principle of least privilege at the network level. This segmentation provides defense-in-depth because an attacker who compromises the internet-facing web tier still cannot directly access the database tier, forcing them to breach multiple security boundaries and providing opportunities for detection and response.

#### Purpose

- Create isolated Docker networks to implement three-tier segmentation architecture.
- Deploy database tier on backend network only (no internet exposure).
- Deploy application tier connected to both frontend and backend networks (bridge).
- Deploy web tier on frontend network only (internet-facing).
- Demonstrate that web tier cannot reach database tier directly (segmentation working).
- Verify that application tier can reach database tier (expected connectivity preserved).
- Understand that segmentation contains lateral movement and limits blast radius.

#### Terminal Commands

```bash
# Create two segmented networks
docker network create frontend-net
docker network create backend-net

# DB only on backend-net; app on both; web only on frontend-net
docker run -d --name db --network backend-net redis:alpine
docker run -d --name app --network backend-net nginx
docker network connect frontend-net app
docker run -d --name web --network frontend-net nginx

# web -> db should FAIL (not on the same network)
docker exec web sh -c 'apk add -q curl; curl -s -m 3 db:6379 || echo BLOCKED'

# app -> db should WORK (shared backend-net)
docker exec app sh -c 'apk add -q curl; nc -z -w3 db 6379 && echo REACHABLE'
```

#### Explanation of the Commands

- `docker network create frontend-net` and `docker network create backend-net` create two isolated bridge networks with their own IP address ranges and network isolation—containers on different networks cannot communicate unless explicitly connected to multiple networks.
- `docker run -d --name db --network backend-net redis:alpine` deploys a Redis database container connected only to the backend network, ensuring the database is not exposed to the frontend network where internet-facing services reside—this implements the security principle that sensitive data stores should not be directly reachable from the internet.
- `docker run -d --name app --network backend-net nginx` deploys an application container initially connected to the backend network where it can reach the database.
- `docker network connect frontend-net app` connects the application container to the frontend network as well, making it a bridge between the two isolated networks—this allows the app to receive requests from the web tier and query the database tier while preventing direct web-to-database connectivity.
- `docker run -d --name web --network frontend-net nginx` deploys a web server container connected only to the frontend network, simulating an internet-facing service—this container can reach the application tier but cannot reach the database tier directly due to network isolation.
- `docker exec web sh -c 'apk add -q curl; curl -s -m 3 db:6379 || echo BLOCKED'` tests network segmentation by attempting to connect from the web container to the database—the command installs curl (Alpine doesn't include it by default), attempts to connect to the db hostname on port 6379 with a 3-second timeout, and echoes "BLOCKED" if the connection fails (which it should due to network isolation).
- `docker exec app sh -c 'apk add -q curl; nc -z -w3 db 6379 && echo REACHABLE'` verifies that expected connectivity is preserved by testing the application-to-database connection—nc (netcat) with `-z` performs a port scan, `-w3` sets a 3-second timeout, and "REACHABLE" is printed if the connection succeeds, proving that the app tier can still access the database tier as required.

#### Evidence

![Task 4 Network Segmentation - Setup](evidence/Task%204%20Network%20segmentation%201.png)

The screenshot shows the creation of two isolated networks (frontend-net and backend-net) and the deployment of three containers with proper network assignments, confirming that the three-tier architecture was established successfully.

![Task 4 Network Segmentation - Testing](evidence/Task%204%20Network%20segmentation%202.png)

The screenshot demonstrates the network segmentation testing with the web-to-database test showing "BLOCKED" and the app-to-database test showing "REACHABLE", proving that network isolation works correctly while preserving required connectivity.

#### Notes

The network segmentation task successfully demonstrated defense-in-depth through network isolation. An attacker who compromises the web tier cannot directly query or dump the database, significantly reducing the impact of a breach. This architecture mirrors cloud security patterns with public subnets (web tier), private subnets (database tier), and application load balancers or API gateways (app tier) connecting them. Production implementations should enhance this pattern with Network Policies in Kubernetes, security groups in cloud providers, microsegmentation with service meshes, and zero-trust networking that requires authentication for every network connection rather than relying solely on network boundaries.

---

### Task 5 — Firewall Rules (Default-Deny)

Host-based firewalls provide network-level access control at the operating system or container level, complementing network segmentation by enforcing rules about what traffic is allowed to reach services. The default-deny firewall policy is a fundamental security principle where all traffic is blocked by default, and only explicitly necessary traffic is allowed through specific rules—this is the opposite of default-allow (permit everything except explicitly blocked traffic) which is inherently less secure. This approach mirrors cloud security groups (AWS, Azure, GCP) where inbound traffic is denied by default and teams must create explicit allow rules for required services, implementing the principle of least privilege at the network layer.

#### Purpose

- Understand the default-deny firewall policy (deny all, then explicitly allow required traffic).
- Configure iptables firewall rules inside a container to demonstrate host-level filtering.
- Set default INPUT policy to DROP to block all incoming traffic by default.
- Create explicit ACCEPT rules for required services (HTTPS port 443).
- Allow loopback interface traffic for localhost communication.
- Understand the relationship between host firewalls and cloud security groups.

#### Terminal Commands

```bash
# Inside a throwaway container with iptables, model default-deny + allow 443
docker run --rm --cap-add=NET_ADMIN alpine sh -c '\
 apk add -q iptables; \
 iptables -P INPUT DROP; \
 iptables -A INPUT -p tcp --dport 443 -j ACCEPT; \
 iptables -A INPUT -i lo -j ACCEPT; \
 iptables -L INPUT -n'
```

#### Explanation of the Commands

- `docker run --rm --cap-add=NET_ADMIN alpine sh -c` launches a temporary Alpine Linux container with the NET_ADMIN capability, which is required to modify network configuration including firewall rules—without this capability, iptables commands would fail with permission errors.
- `apk add -q iptables` installs the iptables package in Alpine Linux (minimal container images don't include firewall tools by default) with `-q` for quiet output to reduce terminal noise.
- `iptables -P INPUT DROP` sets the default policy for the INPUT chain to DROP, meaning any incoming packet that doesn't match an explicit ACCEPT rule will be dropped—this implements the default-deny principle where security is the default state.
- `iptables -A INPUT -p tcp --dport 443 -j ACCEPT` appends a rule to the INPUT chain that accepts TCP traffic destined for port 443 (HTTPS)—the `-A` flag appends to the chain, `-p tcp` specifies the protocol, `--dport 443` matches the destination port, and `-j ACCEPT` specifies the action to take for matching packets.
- `iptables -A INPUT -i lo -j ACCEPT` accepts all traffic on the loopback interface (`lo`), which is essential for localhost communication between processes on the same host—without this rule, local services couldn't communicate with each other even when the security policy doesn't require network-level isolation for localhost.
- `iptables -L INPUT -n` lists the rules in the INPUT chain in numeric format (IP addresses and port numbers instead of hostnames and service names), displaying the configured firewall policy to verify that the rules were applied correctly.

#### Evidence

![Task 5 Firewall Rules](evidence/Task%205.png)

The screenshot shows the iptables configuration with Chain INPUT policy set to DROP, followed by two ACCEPT rules for tcp dpt:443 and all traffic on the loopback interface, confirming that the default-deny firewall policy was successfully configured.

#### Notes

The firewall rules task demonstrated the security-group model where nothing is allowed unless explicitly permitted. This default-deny approach provides strong security because forgotten services or misconfigurations automatically result in denied access rather than unintended exposure. In production environments, firewall rules should be carefully designed to allow only necessary traffic (SSH from bastion hosts, HTTPS from load balancers, database ports from application subnets), regularly audited to remove obsolete rules, version controlled for tracking changes, and automated through infrastructure-as-code to prevent manual configuration errors. Cloud security groups implement this same model with additional features like stateful firewalling (allowing return traffic for established connections automatically), integration with service discovery (allowing traffic from security groups rather than IP addresses), and central management through cloud control planes.

---

### Task 6 — Container / Host Hardening

Container hardening is the practice of reducing the attack surface of container deployments by applying security controls that limit what attackers can do even if they gain code execution inside a container. The principle is defense-in-depth: encryption and authentication protect data and access, network segmentation limits lateral movement, and host hardening limits the capabilities available inside compromised systems. Hardening measures include running as non-root users (limiting privilege escalation), using read-only filesystems (preventing malware persistence), dropping Linux capabilities (removing dangerous kernel features), preventing privilege escalation (blocking setuid exploits), and using minimal base images (reducing vulnerability surface). These controls implement the principle of least privilege at the compute layer.

#### Purpose

- Deploy a container with multiple hardening measures applied simultaneously.
- Configure non-root user (UID/GID 1000) to prevent root-level compromises.
- Enable read-only filesystem to prevent malware installation and file modifications.
- Drop all Linux capabilities to remove dangerous kernel-level permissions.
- Enable no-new-privileges security option to prevent privilege escalation attacks.
- Use tmpfs for temporary storage that exists only in memory (no persistent writes).
- Verify hardening configuration using docker inspect commands.
- Scan container image for vulnerabilities using Trivy to identify security issues.
- Understand which attacks each hardening measure prevents or mitigates.

#### Terminal Commands

```bash
# A hardened run of a service
docker run -d --name hardened \
 --user 1000:1000 \           # non-root
 --read-only \                # read-only root filesystem
 --cap-drop=ALL \             # drop all Linux capabilities
 --security-opt no-new-privileges \
 --tmpfs /tmp \
 nginxinc/nginx-unprivileged

docker inspect hardened --format 'User={{.Config.User}} ReadOnly={{.HostConfig.ReadonlyRootfs}}'

# Scan an image for known vulnerabilities
docker run --rm aquasec/trivy image --severity HIGH,CRITICAL nginx:alpine | head -20
```

#### Explanation of the Commands

- `docker run -d --name hardened` starts a new container in detached mode with the name "hardened" for easy reference and management.
- `--user 1000:1000` specifies that the container should run as UID 1000 and GID 1000 (non-root) instead of the default root user (UID 0)—this means that even if an attacker achieves code execution inside the container, they have limited user-level privileges rather than root privileges that could be used for privilege escalation or container escape attacks.
- `--read-only` mounts the container's root filesystem as read-only, preventing any writes to the filesystem except for explicitly mounted volumes or tmpfs—this prevents attackers from installing backdoors, modifying binaries, tampering with logs, or persisting malware across container restarts.
- `--cap-drop=ALL` drops all Linux capabilities from the container process, removing dangerous kernel-level permissions like CAP_NET_RAW (crafting raw packets), CAP_SYS_ADMIN (mounting filesystems, loading kernel modules), and CAP_NET_BIND_SERVICE (binding privileged ports <1024)—this implements least privilege by removing capabilities the application doesn't need.
- `--security-opt no-new-privileges` prevents processes inside the container from gaining additional privileges through setuid binaries, filesystem capabilities, or other mechanisms—this blocks common privilege escalation techniques where attackers find vulnerable setuid binaries (like sudo with known exploits) to gain root access.
- `--tmpfs /tmp` mounts a temporary filesystem in memory at /tmp, providing writable space for temporary files that applications need while ensuring those files don't persist to disk (they're lost when the container stops)—this balances the need for some writable space with the security of read-only filesystems.
- `nginxinc/nginx-unprivileged` uses a hardened nginx image designed to run as non-root, avoiding the common security antipattern of running web servers as root and then dropping privileges—using purpose-built unprivileged images is more secure than trying to de-privilege root-designed images.
- `docker inspect hardened --format 'User={{.Config.User}} ReadOnly={{.HostConfig.ReadonlyRootfs}}'` verifies the hardening configuration by extracting the User and ReadonlyRootfs settings from the container's configuration, confirming that the security settings were applied correctly.
- `docker run --rm aquasec/trivy image --severity HIGH,CRITICAL nginx:alpine` runs Trivy vulnerability scanner to analyze the nginx:alpine image and report all known vulnerabilities with HIGH or CRITICAL severity—Trivy checks the image's packages against vulnerability databases (CVE databases) to identify security issues that should be patched by updating to newer image versions.
- `| head -20` limits the output to the first 20 lines to show a summary of findings without overwhelming the terminal with hundreds of lines of detailed CVE information.

#### Evidence

![Task 6 Container Hardening - Deployment](evidence/Task%206.png)

The screenshot shows the Docker run command deploying the hardened container with all security options configured, confirming that the hardened container was started successfully.

![Task 6 Container Hardening - Verification](evidence/Task%206%20container%202.png)

The screenshot demonstrates the docker inspect command output showing "User=1000:1000 ReadOnly=true", confirming that the non-root user and read-only filesystem hardening measures were successfully applied.

![Task 6 Container Hardening - Vulnerability Scan](evidence/Task%206%20container%203.png)

The screenshot displays the Trivy vulnerability scan results showing the target nginx:alpine (alpine 3.24.1) with a total of 2 vulnerabilities (HIGH: 2, CRITICAL: 0), demonstrating that vulnerability scanning identified security issues that should be addressed by updating to patched versions.

#### Notes

The container hardening task demonstrated how multiple security controls work together to implement defense-in-depth at the compute layer. Each hardening measure addresses specific attack scenarios: non-root users prevent many container escape exploits that require root, read-only filesystems prevent malware persistence and backdoor installation, dropped capabilities prevent kernel-level exploits and privilege escalation, no-new-privileges prevents setuid attacks, and minimal base images reduce the number of installed packages that could contain vulnerabilities. Production container security should also include: using distroless or minimal base images (Alpine, scratch), scanning images in CI/CD pipelines before deployment, implementing Pod Security Standards in Kubernetes, using runtime security tools (Falco, Sysdig) to detect anomalous behavior, enabling AppArmor or SELinux mandatory access control, setting resource limits to prevent resource exhaustion attacks, and implementing security patch management processes to update images regularly.

---

## Verification Commands

After completing all tasks, the following verification commands confirm successful implementation:

```bash
# Verify RBAC Configuration
kubectl get rolebinding dev-rb -n app -o yaml

# Verify Container Hardening
docker inspect hardened --format '{{json .HostConfig.CapDrop}}'
```

#### Evidence

![Verification Commands](evidence/Verification%20Command.png)

The screenshot displays the verification commands output showing the RBAC role binding configuration with resource version "616", role reference "dev-role", and subject ServiceAccount "dev" in namespace "app", followed by the container capabilities output showing `["ALL"]` confirming all capabilities were dropped, proving that both RBAC and container hardening configurations were successfully applied.

---

## Security Best-Practices Checklist

The following security best practices were implemented and verified throughout this lab:

- [✓] **Service requires authentication** (unauthenticated requests rejected) — Task 1 implemented HTTP Basic authentication with 401 responses for requests without credentials and 200 responses for authenticated requests.

- [✓] **MFA / second factor implemented and validated** — Task 2 generated TOTP shared secrets, calculated time-based codes, and validated user input with "MFA OK" confirmation.

- [✓] **Authorization enforced by RBAC** (least privilege; unauthorized actions denied) — Task 3 configured Kubernetes roles with limited permissions showing "yes" for allowed operations and "no" for denied operations.

- [✓] **Network segmented** so the data tier is unreachable from the front tier — Task 4 demonstrated three-tier architecture with web→database BLOCKED and app→database REACHABLE.

- [✓] **Default-deny firewall** with explicit allow rules — Task 5 configured iptables with DROP policy and explicit ACCEPT rules for required ports.

- [✓] **Container hardened:** non-root, minimal, capabilities dropped, read-only; image scanned — Task 6 deployed containers with User=1000:1000, ReadOnly=true, capabilities dropped, and Trivy scan completed.

- [✓] **Defense-in-depth implemented** — Multiple layers of security controls across access control, network segmentation, and host hardening.

- [✓] **Least privilege applied** — Permissions, network access, and container capabilities limited to minimum necessary.

- [✓] **Before-and-after testing methodology** — Demonstrated authentication enforcement (401/200), authorization validation (yes/no), network isolation (BLOCKED/REACHABLE), and hardening verification.

---

## Short-Answer Questions

### Q1. Explain the difference between authentication and authorization using Tasks 1 and 3.

**Authentication (Task 1)** answers the question: *"WHO are you?"* In Task 1, we implemented HTTP Basic authentication where users must prove their identity with credentials (username: `student`, password: `P@ssw0rd!`). Without valid credentials, the system rejects access with HTTP 401 (Unauthorized). With valid credentials, the system recognizes the user and allows access (HTTP 200). Authentication is about **identity verification**.

**Authorization (Task 3)** answers the question: *"WHAT are you allowed to do?"* In Task 3, we implemented RBAC where the authenticated `dev` service account has limited permissions. The system allows the developer to LIST pods (permitted action → yes). The system denies the developer from CREATE deployments or DELETE pods (unauthorized actions → no). Authorization is about **permission enforcement**.

**Key Difference:** Authentication happens **first**—you must prove who you are. Authorization happens **second**—the system decides what you can do based on your identity. You can be authenticated but still unauthorized for certain actions.

---

### Q2. Why is MFA so effective, and which attacks does it defeat?

MFA (Multi-Factor Authentication) is effective because it combines factors from **different security classes**: something you know (password) and something you have (TOTP device). This means that attackers must compromise multiple independent factors to gain access, which is significantly harder than stealing a single password.

**Attacks MFA Defeats:**

1. **Credential Theft / Password Breaches** - Stolen passwords are useless without the second factor.
2. **Phishing Attacks** - Even if users enter passwords on fake sites, attackers can't access the TOTP device.
3. **Brute Force Attacks** - Guessing the password isn't enough without TOTP codes that change every 30 seconds.
4. **Credential Stuffing** - Reused passwords from other breaches won't work without the second factor.
5. **Keylogger Attacks** - Malware that captures passwords can't capture dynamically generated TOTP codes.

According to industry research, MFA blocks over 99% of automated credential-based attacks, making it "the cheapest big security win" in cybersecurity.

---

### Q3. How does network segmentation limit the damage of a compromised web server?

Network segmentation creates **security boundaries** within infrastructure by isolating services into separate network segments with controlled connectivity. In Task 4, we implemented three-tier architecture with isolated networks:

**How It Limits Damage:**

1. **Prevents Direct Database Access** - The compromised web server cannot communicate directly with the database (we verified: BLOCKED). An attacker cannot directly query or dump sensitive data.

2. **Contains Lateral Movement** - Segmentation creates barriers that attackers must breach to move deeper into the infrastructure. Compromising one tier doesn't automatically compromise other tiers.

3. **Reduces Blast Radius** - The impact of a breach is limited to the compromised network segment. Critical data assets remain protected behind additional network barriers.

4. **Forces Traffic Through Controlled Paths** - All database access must go through the application tier, which can implement additional security controls (authentication, input validation, logging) and provides a choke point for monitoring.

This architecture mirrors cloud deployments with public subnets (internet-facing) and private subnets (databases), implementing defense-in-depth where multiple security layers must be breached to reach sensitive resources.

---

### Q4. What does a default-deny firewall policy achieve, and how does it relate to cloud security groups?

A **default-deny** firewall policy means all traffic is denied by default, and only explicitly allowed traffic is permitted. The security posture is: *"Nothing is permitted unless specifically authorized."*

**What It Achieves:**

1. **Implements Least Privilege for Network Access** - Only required ports and protocols are open. All unexpected traffic is automatically blocked.

2. **Prevents Unknown/Unexpected Connections** - New vulnerabilities or misconfigured services can't be exploited if their ports aren't open. Zero-day attacks targeting unexpected ports are blocked by default.

3. **Explicit Security Posture** - Every allowed connection is documented and intentional. Security is the default state, not an afterthought.

4. **Fails Secure** - If rules are misconfigured, the system defaults to blocking. Errors don't accidentally open access.

**Relationship to Cloud Security Groups:**

Cloud security groups (AWS, Azure, GCP) implement the exact same default-deny model:

- Default: Deny all inbound traffic
- Explicit rules: Allow specific ports from specific sources
- Example: Allow TCP 443 from 0.0.0.0/0 (HTTPS), Allow TCP 22 from 10.0.0.0/8 (SSH from corporate network)

Both local firewalls (iptables) and cloud security groups follow this principle: start with deny-all, add only necessary allow rules, document each exception, and regularly review to remove unused rules.

---

### Q5. List the hardening measures you applied and the attack surface each one removes.

**Container Hardening Measures:**

| # | Hardening Measure | Attack Surface Removed | Security Benefit |
|---|------------------|----------------------|-----------------|
| 1 | **Non-root User** (`--user 1000:1000`) | Root-level container escapes, privilege escalation exploits, system-wide modifications | Even if compromised, attacker has limited user privileges; cannot modify system files or access root-only resources |
| 2 | **Read-only Filesystem** (`--read-only`) | Malware installation, backdoor persistence, binary modification, log tampering | Attackers cannot write malicious code to disk; prevents persistent threats; limits post-exploitation activities |
| 3 | **Drop All Capabilities** (`--cap-drop=ALL`) | Kernel exploitation, network manipulation, system call abuse, advanced container escapes | Removes dangerous Linux capabilities (CAP_NET_RAW, CAP_SYS_ADMIN); prevents low-level system manipulation |
| 4 | **No New Privileges** (`--security-opt no-new-privileges`) | Setuid binary exploits, privilege escalation via file permissions, sudo exploitation | Prevents processes from gaining more privileges than parent; blocks common privilege escalation techniques |
| 5 | **Minimal Base Image** (nginx-unprivileged) | Unnecessary packages/tools, additional vulnerabilities, attack utilities | Fewer installed packages = fewer vulnerabilities; removes tools attackers could use; reduces CVE exposure |
| 6 | **Tmpfs for /tmp** (`--tmpfs /tmp`) | Persistent malicious files, storage-based attacks | Temporary files stored in memory only; cleared on restart; provides necessary writable space without persistence |

**Combined Security Impact:** These measures implement defense-in-depth through least privilege—they reduce the initial attack surface, limit what attackers can do even with code execution, prevent lateral movement from container to host, prevent persistence mechanisms, and facilitate detection of unusual behavior.

---

## Conclusion

This lab successfully demonstrated the complete implementation of access control and network security controls in cloud computing environments, progressing from foundational authentication mechanisms to enterprise-scale security architectures. The two-session structure provided a logical progression from understanding identity and permission controls (authentication with passwords and MFA, authorization with RBAC) to implementing network-level and host-level defense-in-depth strategies (network segmentation, firewall rules, container hardening).

### Key Findings:

**1. Identity Is the Security Perimeter:**

The most critical lesson from this lab is that modern security architectures center around identity rather than network perimeter. Every control implemented—from HTTP Basic authentication to RBAC policies to network segmentation—ultimately asks two fundamental questions: "Are you who you claim to be?" and "Are you allowed to do this?" This identity-centric approach recognizes that traditional perimeter security (firewalls at the network edge) is insufficient in cloud environments where resources are distributed, users are remote, and services communicate across network boundaries.

**2. Authentication Alone Is Insufficient:**

Task 1 demonstrated that authentication proves WHO you are, but without authorization (Task 3), every authenticated user would have full system access. The combination of authentication (proving identity) and authorization (enforcing permissions) implements the principle of least privilege where users get only the access they need for their job function. This prevents insider threats, limits the impact of compromised credentials, and provides audit trails for compliance.

**3. Network Segmentation Provides Defense-in-Depth:**

Task 4 showed that even with proper authentication and authorization, network-level isolation provides an additional security layer. An attacker who compromises authentication controls still cannot reach the database tier due to network segmentation. This defense-in-depth strategy ensures that no single point of failure compromises the entire system, and each security layer must be independently breached.

**4. Default-Deny Is the Foundation of Secure Systems:**

Task 5 demonstrated the default-deny principle where security is the default state and access must be explicitly granted. This approach appears in firewalls (deny all, then allow specific ports), RBAC (no permissions by default, then grant specific roles), and cloud security groups (block all inbound, then allow specific sources). Default-deny ensures that misconfigurations, forgotten services, and new vulnerabilities automatically result in denied access rather than unintended exposure.

**5. Container Hardening Limits Post-Exploitation Impact:**

Task 6 showed that even if attackers bypass authentication, authorization, and network controls, container hardening limits what they can do with compromised systems. Non-root users prevent privilege escalation, read-only filesystems prevent malware persistence, dropped capabilities prevent kernel exploits, and vulnerability scanning identifies weaknesses before deployment. This demonstrates that security must be implemented at every layer—access control, network, and compute.

**6. MFA Is Essential for Modern Security:**

Task 2 demonstrated why MFA is considered "the cheapest big security win"—it defeats over 99% of credential-based attacks with relatively simple implementation. In an era where password breaches are common and phishing is sophisticated, the second factor (something you have) provides critical protection that passwords alone (something you know) cannot provide.

### Real-World Applications:

The techniques learned in this lab are directly applicable to:

**Cloud Access Management:**
- AWS IAM with MFA enforcement for privileged accounts
- Azure AD conditional access policies requiring MFA for sensitive resources
- Google Cloud Identity with TOTP-based verification

**Kubernetes Security:**
- RBAC policies for multi-tenant clusters
- Network Policies for pod-to-pod communication control
- Pod Security Standards for container hardening requirements
- Service meshes (Istio, Linkerd) for mTLS and authorization

**Enterprise Authentication:**
- SSO (Single Sign-On) with SAML/OIDC and MFA
- Zero-trust architectures requiring authentication for every request
- Privileged Access Management (PAM) systems

**Cloud Network Security:**
- VPC segmentation with public/private subnets
- Security groups with default-deny inbound rules
- Network ACLs for subnet-level filtering
- Service endpoint restrictions

**Container Security:**
- CI/CD pipeline integration with vulnerability scanning
- Runtime security monitoring (Falco, Sysdig)
- Admission controllers enforcing security policies
- Distroless and minimal base images

### Future Learning:

This lab establishes the foundation for advanced security topics including:

**Advanced Authentication:**
- WebAuthn/FIDO2 for passwordless authentication
- Biometric authentication integration
- Certificate-based mutual TLS (mTLS)
- Risk-based adaptive authentication

**Zero-Trust Architecture:**
- Identity-aware proxies (IAP)
- Service-to-service authentication
- Continuous verification and trust evaluation
- Micro-segmentation beyond network boundaries

**Advanced Kubernetes Security:**
- OPA (Open Policy Agent) for fine-grained policy enforcement
- Service mesh security (Istio authorization policies)
- Pod security admission controllers
- Secret management with external vaults

**Advanced Container Security:**
- Runtime behavior monitoring and anomaly detection
- Supply chain security (image signing, SBOM)
- Confidential containers with encrypted execution
- Rootless containers and user namespaces

---

## References

The following resources were referenced during this lab and provide additional depth for further study:

1. **Course lecture** — Week 5 (Access Control) and Week 9 (Network Security patterns), Prof. Dr. Shahrulniza Musa, UniKL MIIT, covering authentication mechanisms, authorization models, and defense-in-depth strategies.

2. **Docker security documentation** — [https://docs.docker.com/engine/security](https://docs.docker.com/engine/security) — Comprehensive guide to container security, including user namespaces, capabilities, and security options.

3. **Kubernetes RBAC documentation** — [https://kubernetes.io/docs/reference/access-authn-authz/rbac](https://kubernetes.io/docs/reference/access-authn-authz/rbac) — Official documentation for role-based access control in Kubernetes.

4. **RFC 6238** — *TOTP: Time-Based One-Time Password Algorithm* — Specification for TOTP authentication used in Task 2.

5. **NIST SP 800-63B** — *Digital Identity Guidelines: Authentication and Lifecycle Management* — Federal guidelines on authentication strength and multi-factor authentication.

6. **CIS Docker Benchmark** — [https://www.cisecurity.org](https://www.cisecurity.org) — Industry best practices for Docker container hardening.

7. **CIS Kubernetes Benchmark** — Security configuration benchmarks for Kubernetes deployments.

8. **OWASP Top 10** — Understanding of authentication and authorization vulnerabilities including broken access control.

9. **Zero Trust Architecture (NIST SP 800-207)** — Framework for identity-centric security models.

10. **Cloud Security Alliance (CSA) Security Guidance v5** — *Domain 1: Cloud Computing Concepts and Architectures* — Industry best practices for cloud access control and network security.

---

## Appendix: Cleanup Commands

To clean up the lab environment and free system resources, execute the following commands:

```bash
# Stop and remove Docker containers
docker rm -f authsvc db app web hardened 2>/dev/null

# Remove Docker networks
docker network rm frontend-net backend-net 2>/dev/null

# Delete Kubernetes cluster
kind delete cluster --name ccse-lab4

# Verify cleanup
docker ps -a
docker network ls
kind get clusters
```

#### Evidence

![Cleanup & Teardown](evidence/Cleanup%20&%20Teardown.png)

The screenshot shows the execution of cleanup commands removing all containers (authsvc, db, app, web, hardened), networks (frontend-net, backend-net), and the Kubernetes cluster (ccse-lab4), confirming that the lab environment was successfully cleaned up and system resources were freed.

**Note:** The cleanup removes all containers, networks, and clusters created during the lab. The `2>/dev/null` redirects suppress error messages for resources that don't exist, allowing the cleanup script to run safely even if some tasks were not completed. If you need to preserve evidence for report submission, take screenshots before executing cleanup commands.

---

## Acknowledgments

This lab was completed as part of the IKB42603 Cloud Computing Security Essentials course at Universiti Kuala Lumpur Malaysian Institute of Information Technology (UniKL MIIT). Special thanks to:

- **Prof. Dr. Shahrulniza Musa** for developing the comprehensive lab curriculum covering authentication, authorization, network security, and container hardening techniques, and for providing expert guidance on cloud security principles and defense-in-depth strategies throughout the course.

- **Teaching staff** for supervising lab sessions, providing clarifications on security procedures, and offering feedback on implementation approaches during hands-on exercises.

- **The Docker Project** and **Kubernetes Community** for providing open-source containerization and orchestration platforms that enable hands-on learning of production-grade security technologies.

- **Trivy**, **OWASP**, and other open-source security projects that make vulnerability scanning and security assessment accessible for educational purposes and production deployments.

---

## End of Report

**Lab Status:** All tasks completed with evidence  
**Evidence Files:** Screenshots documenting all six tasks across both sessions  
**Verification:** All commands executed successfully with expected outputs  
**Learning Outcomes:** Achieved comprehensive understanding of authentication, authorization, MFA, network segmentation, firewall configuration, and container hardening

---

**Submitted by:** Surya Giri A/L Shanker  
**Student ID:** 52215124335  
**Course:** IKB42603 Cloud Computing Security Essentials  
**Institution:** Universiti Kuala Lumpur Malaysian Institute of Information Technology (UniKL MIIT)  
**Lab:** Lab 4 - Access Control & Network Security  
**Sessions:** Week 7 (Authentication & Authorization) and Week 8 (Network Security & Hardening)  
**Date:** August 30, 2026  
**Professor:** Prof. Dr. Shahrulniza Musa

---

**Security Statement:** All security implementations performed in this lab used local development environments (kind clusters, Docker containers, self-signed certificates) and test credentials appropriate for educational purposes. No actual sensitive data was used, and no production systems were accessed. The authentication credentials and containers created during this lab have been properly cleaned up and removed after completion. This report demonstrates understanding of security principles and practical skills that can be applied to real-world cloud security implementations with appropriate production-grade controls, enterprise authentication systems, comprehensive audit logging, and regular security assessments.

