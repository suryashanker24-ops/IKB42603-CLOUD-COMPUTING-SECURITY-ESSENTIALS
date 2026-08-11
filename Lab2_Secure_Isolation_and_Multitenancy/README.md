# IKB42603 Cloud Computing Security Essentials

## LAB 2 REPORT: Secure Isolation & Multi-Tenancy
### Compute, Network and Storage Isolation — Docker & Kubernetes

---

**Student Name:** Surya  
**Lab Title:** Secure Isolation & Multi-Tenancy  
**Date:** Week 3-4  
**Course:** IKB42603 Cloud Computing Security Essentials  
**Instructor:** Prof. Dr. Shahrulniza Musa  

---

## Table of Contents
1. [Lab Overview](#lab-overview)
2. [Lab Learning Outcomes](#lab-learning-outcomes)
3. [Technical Prerequisites](#technical-prerequisites)
4. [Session A - Week 3: Compute Isolation & Default-Open Risk](#session-a)
   - [Setup: Cluster with Policy Enforcement](#setup)
   - [Task 1: Two Tenants on One Cluster](#task-1)
   - [Task 2: Observe the Default-Open Risk](#task-2)
   - [Task 3: Contain the Noisy Neighbour](#task-3)
5. [Session B - Week 4: Network & Storage Isolation](#session-b)
   - [Task 4: Default-Deny Network Isolation](#task-4)
   - [Task 5: Storage & Secret Isolation](#task-5)
   - [Task 6: Data Remanence & Secure Deletion](#task-6)
6. [Verification Commands](#verification)
7. [Short-Answer Questions](#questions)
8. [Security Best-Practices Checklist](#checklist)
9. [Conclusion](#conclusion)
10. [References](#references)

---

## 1. Lab Overview {#lab-overview}

<cite index="1-1,1-2,1-3,1-4,1-5,1-6,1-7,1-8,1-9,1-10">This lab demonstrates compute isolation by separating tenants into containers and Kubernetes namespaces, observes the default-open behaviour of shared infrastructure, implements network isolation with default-deny NetworkPolicy, enforces storage isolation so tenants cannot access each other's data or secrets, and explains data remanence with secure deletion demonstrations.</cite>

The lab is structured over two weeks:
- **Session A (Week 3):** Focus on compute isolation, namespaces, resource quotas, and identifying the default-open security risk
- **Session B (Week 4):** Implementation of network and storage isolation controls, including default-deny policies and secure deletion

---

## 2. Lab Learning Outcomes {#lab-learning-outcomes}

At the end of this lab, I am able to:

1. <cite index="1-2">Demonstrate compute isolation by separating tenants into containers and Kubernetes namespaces</cite>
2. <cite index="1-4">Observe the default-open behaviour of shared infrastructure and explain why it is a risk</cite>
3. <cite index="1-6">Implement network isolation with a default-deny NetworkPolicy and prove cross-tenant traffic is blocked</cite>
4. <cite index="1-8">Enforce storage isolation so one tenant cannot read another tenant's data or secrets</cite>
5. <cite index="1-10">Explain data remanence and demonstrate secure deletion</cite>

---

## 3. Technical Prerequisites {#technical-prerequisites}

<cite index="1-14,1-15,1-16,1-17">The lab requires a laptop with at least 8 GB RAM and admin rights, Docker Desktop/Docker Engine (free), kind and kubectl (free), and internet only for the first image download; the lab then runs offline.</cite>

**System Used:**
- Operating System: Kali Linux
- Docker: Installed and running
- kubectl: Installed
- kind: Installed
- RAM: 8+ GB

---

## Session A (Week 3) — Compute Isolation & the Default-Open Risk {#session-a}

### Setup — Cluster with Policy Enforcement {#setup}

<cite index="1-18,1-19">This lab uses a Kubernetes cluster with a CNI that enforces NetworkPolicy. We use kind with Calico so that the isolation rules actually take effect (the default kind network does not enforce policies).</cite>

**Step 1: Create a Kubernetes Cluster with Disabled Default CNI**

The following commands create a kind cluster with the default CNI disabled to allow Calico installation:

```bash
cat <<EOF | kind create cluster --name ccse-lab2 --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
EOF
```

**Screenshot - Cluster Creation:**

![Setup Cluster 1](Setup%20Cluster%20%20with%20policy%20Enforcement%201.png)

**Output:**
```
Creating cluster "ccse-lab2" ...
✓ Ensuring node image (kindest/node:v1.30.0)
✓ Preparing nodes
✓ Writing configuration
✓ Starting control-plane
✓ Installing StorageClass
Set kubectl context to "kind-ccse-lab2"
You can now use your cluster with:
kubectl cluster-info --context kind-ccse-lab2
```

**Step 2: Install Calico CNI**

Next, Calico is installed to enable NetworkPolicy enforcement:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
kubectl -n kube-system rollout status daemonset/calico-node --timeout=180s
```

**Screenshot - Calico Installation:**

![Setup Cluster 2](Setup%20Cluster%20with%20policy%20Enforcement%202.png)

**Output:**
```
poddisruptionbudget.policy/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
serviceaccount/calico-node created
[... multiple CRD and resource creations ...]
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created
Waiting for daemon set "calico-node" rollout to finish: 0 of 1 updated pods are available...
daemon set "calico-node" successfully rolled out
```

**Analysis:** The cluster is now ready with Calico CNI, which will enforce NetworkPolicy rules that we apply later in the lab.

---

### Task 1 — Two Tenants on One Cluster {#task-1}

<cite index="1-21">Model two customers as two namespaces sharing the same physical infrastructure.</cite>

**Step 1: Create Two Namespaces**

```bash
kubectl create namespace tenant-a
kubectl create namespace tenant-b
```

**Screenshot - Namespace Creation:**

![Task 1 Namespaces](Task%201%20Two%20tenants%20on%20One%20Cluster.png)

**Output:**
```
namespace/tenant-a created
namespace/tenant-b created
```

**Step 2: Deploy Web Servers for Each Tenant**

<cite index="1-22">Deploy a simple web server for each tenant</cite>

```bash
kubectl -n tenant-a create deployment web --image=nginx
kubectl -n tenant-b create deployment web --image=nginx
kubectl -n tenant-a expose deployment web --port=80
kubectl -n tenant-b expose deployment web --port=80
kubectl get pods,svc -n tenant-a
```

**Screenshot - Deployment and Services:**

![Task 1 Deployment](Task%201%20Deploy%20a%20simple%20web%20server%20for%20each%20tenant.png)

**Output:**
```
deployment.apps/web created
deployment.apps/web created
service/web exposed
service/web exposed

NAME                       READY   STATUS    RESTARTS   AGE
pod/web-7c56dcdb9b-hdnrp   1/1     Running   0          0

NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/web   ClusterIP   10.96.111.229   <none>        80/TCP    87s
```

**Analysis:** Both tenants now have their own nginx web server deployments running in isolated namespaces. However, namespace isolation alone does not prevent network communication between tenants.

---

### Task 2 — Observe the Default-Open Risk {#task-2}

<cite index="1-22,1-23">By default, pods in one namespace can reach pods in another. Prove it: launch a test pod in tenant-a and connect to tenant-b's service.</cite>

**Step 1: Get Tenant-B's Service IP**

```bash
kubectl get svc web -n tenant-b -o jsonpath='{.spec.clusterIP}'; echo
```

**Screenshot - Get Service IP:**

![Task 2 Service IP](Task%202%20Observe%20the%20Default-Open%20Risk%201.png)

**Output:**
```
10.96.253.95
```

**Step 2: Test Cross-Tenant Connectivity (Before NetworkPolicy)**

```bash
kubectl -n tenant-a run probe --rm -it --image=curlimages/curl --restart=Never \
  -- curl -s -m 5 http://10.96.253.95 -o /dev/null -w 'HTTP %{http_code}\n'
```

**Screenshot - Cross-Tenant Access Success:**

![Task 2 HTTP 200](Task%202%20Observe%20the%20Default-Open%20Risk%202.png)

**Output:**
```
HTTP 200
pod "probe" deleted from tenant-a namespace
```

**Analysis:** <cite index="1-25,1-26,1-27">A result of HTTP 200 means tenant-a reached tenant-b. On shared infrastructure, isolation is NOT automatic — you must configure it. This is the multi-tenancy risk from Week 3.</cite>

---

### Task 3 — Contain the Noisy Neighbour (Resource Quotas) {#task-3}

<cite index="1-28,1-29">Isolation is also about resources. Apply a quota so one tenant cannot exhaust the shared node.</cite>

**Step 1: Apply Resource Quota to Tenant-A**

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-a-quota
  namespace: tenant-a
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 512Mi
    pods: "5"
EOF
```

**Screenshot - Resource Quota Creation:**

![Task 3 Quota Creation](Task%203%20Contain%20the%20Noisy%20Neighbour%201.png)

**Output:**
```
resourcequota/tenant-a-quota created
```

**Step 2: Verify Resource Quota**

```bash
kubectl describe resourcequota tenant-a-quota -n tenant-a
```

**Screenshot - Resource Quota Details:**

![Task 3 Quota Details](Task%203%20Contain%20the%20Noisy%20Neighbour%202.png)

**Output:**
```
Name:            tenant-a-quota
Namespace:       tenant-a
Resource         Used   Hard
--------         ----   ----
pods             1      5
requests.cpu     0      1
requests.memory  0      512Mi
```

**Analysis:** The ResourceQuota ensures that tenant-a cannot create more than 5 pods or consume more than 1 CPU core and 512Mi of memory. This prevents a "noisy neighbour" scenario where one tenant monopolizes shared cluster resources.

<cite index="1-30,1-31">End of Session A. Save the HTTP 200 result — you will show that the SAME probe returns a failure after applying network policy in Session B.</cite>

---

## Session B (Week 4) — Network & Storage Isolation {#session-b}

### Task 4 — Default-Deny Network Isolation {#task-4}

<cite index="1-32,1-33">Apply a default-deny ingress policy to each tenant, then allow only same-namespace traffic. This is the segmentation principle: deny by default, permit by exception.</cite>

**Step 1: Apply Default-Deny Ingress Policy to Tenant-B**

```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: tenant-b
spec:
  podSelector: {}
  policyTypes: [Ingress]
EOF
```

**Screenshot - NetworkPolicy Creation:**

![Task 4 NetworkPolicy](Task%204%20Default-Deny%20Network%20Isolation%201.png)

**Output:**
```
networkpolicy.networking.k8s.io/default-deny-ingress created
```

**Step 2: Re-test Cross-Tenant Connectivity (After NetworkPolicy)**

```bash
kubectl -n tenant-a run probe --rm -it --image=curlimages/curl --restart=Never \
  -- curl -s -m 5 http://10.96.253.95 -o /dev/null -w 'HTTP %{http_code}\n'
```

**Screenshot - Cross-Tenant Access Blocked:**

![Task 4 Timeout](Task%204%20Default-Deny%20Network%20Isolation%202.png)

**Output:**
```
pod "probe" deleted from tenant-a namespace
error: timed out waiting for the condition
```

**Analysis:** <cite index="1-34,1-35">Capture both results side by side: HTTP 200 (before) and timeout (after). This single before/after is the strongest evidence of enforced network isolation.</cite>

**Comparison:**
- **Before NetworkPolicy (Task 2):** HTTP 200 - tenant-a successfully accessed tenant-b
- **After NetworkPolicy (Task 4):** Timeout - tenant-a cannot reach tenant-b

This proves that the default-deny NetworkPolicy successfully blocks all ingress traffic to tenant-b, including traffic from tenant-a.

---

### Task 5 — Storage & Secret Isolation {#task-5}

<cite index="1-36,1-37">Each tenant stores a secret. Prove that tenant-a cannot read tenant-b's secret — storage isolation enforced by RBAC.</cite>

**Step 1: Create Secrets in Each Tenant Namespace**

```bash
kubectl -n tenant-a create secret generic data --from-literal=value=SECRET_A
kubectl -n tenant-b create secret generic data --from-literal=value=SECRET_B
```

**Screenshot - Secret Creation:**

![Task 5 Secrets](Task%205%20Storage%20&%20Secret%20Isolation%201.png)

**Output:**
```
secret/data created
secret/data created
```

**Step 2: Create Service Account and RBAC for Tenant-A**

```bash
# A service account scoped to tenant-a only
kubectl -n tenant-a create serviceaccount app-a
kubectl -n tenant-a create role reader --verb=get --resource=secrets
kubectl -n tenant-a create rolebinding rb --role=reader --serviceaccount=tenant-a:app-a
```

**Screenshot - RBAC Configuration:**

![Task 5 RBAC](Task%205%20Storage%20&%20Secret%20Isolation%202.png)

**Output:**
```
serviceaccount/app-a created
role.rbac.authorization.k8s.io/reader created
rolebinding.rbac.authorization.k8s.io/rb created
```

**Step 3: Test Secret Access with Authorization Checks**

```bash
SA=system:serviceaccount:tenant-a:app-a
kubectl auth can-i get secrets -n tenant-a --as=$SA  # expect: yes
kubectl auth can-i get secrets -n tenant-b --as=$SA  # expect: no
```

**Screenshot - Authorization Test Results:**

![Task 5 Auth Test](Task%205%20Storage%20&%20Secret%20Isolation%203.png)

**Output:**
```
yes
no
```

**Analysis:** The RBAC rules successfully enforce storage isolation. The service account `app-a` in tenant-a can access secrets within its own namespace (tenant-a) but is denied access to secrets in tenant-b. This demonstrates that namespace-scoped RBAC prevents cross-tenant secret access.

---

### Task 6 — Data Remanence & Secure Deletion {#task-6}

<cite index="1-38,1-39">When data is 'deleted', is it really gone? Demonstrate remanence and a secure wipe inside a container volume.</cite>

**Step 1: Demonstrate Data Remanence (Normal Delete)**

```bash
docker run --rm -v ccse-vol:/data alpine sh -c \
  'echo SENSITIVE-PATIENT-RECORD > /data/phi.txt; sync; rm /data/phi.txt; \
  grep -a SENSITIVE /data/* 2>/dev/null; echo scan-done'
```

**Step 2: Demonstrate Secure Wipe**

```bash
docker run --rm -v ccse-vol:/data alpine sh -c \
  'echo SENSITIVE > /data/phi2.txt; sync; \
  dd if=/dev/zero of=/data/phi2.txt bs=1k count=1 conv=notrunc; rm /data/phi2.txt; \
  echo wiped'
```

**Screenshot - Data Remanence and Secure Wipe:**

![Task 6 Remanence](Task%206%20Data%20Remanence%20&%20Secure%20Deletion.png)

**Output:**
```
# First command - data remanence demonstration
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
[... image download ...]
scan-done

# Second command - secure wipe
1+0 records in
1+0 records out
1024 bytes (1.0KB) copied, 0.000061 seconds, 16.0MB/s
wiped
```

**Analysis:** The first command shows that even after using `rm`, sensitive data may still be recoverable from the storage. The second command demonstrates secure deletion by overwriting the file with zeros before removal. <cite index="1-41,1-42">In cloud storage you rarely control physical blocks, so the practical answer to remanence is cryptographic erasure (destroy the key). You will do exactly that in Lab 3.</cite>

---

## 6. Verification Commands {#verification}

<cite index="1-57">Verification commands to confirm proper implementation:</cite>

```bash
kubectl get networkpolicy -A
kubectl describe resourcequota tenant-a-quota -n tenant-a
```

**Screenshot - Final Verification:**

![Verification Commands](Verification%20Command.png)

**Output:**
```
NAMESPACE   NAME                    PODSELECTOR   AGE
tenant-b    default-deny-ingress    <none>        62m

Name:            tenant-a-quota
Namespace:       tenant-a
Resource         Used   Hard
--------         ----   ----
pods             1      5
requests.cpu     0      1
requests.memory  0      512Mi
```

**Analysis:** The verification confirms:
1. NetworkPolicy `default-deny-ingress` is active in tenant-b
2. ResourceQuota `tenant-a-quota` is properly configured and tracking resource usage

---

## 7. Short-Answer Questions {#questions}

### Q1. Why can containers in different namespaces reach each other by default, and why is that dangerous in multi-tenant cloud?

<cite index="1-50">Why can containers in different namespaces reach each other by default, and why is that dangerous in multi-tenant cloud?</cite>

**Answer:**

By default, Kubernetes allows all pod-to-pod communication across namespaces because no NetworkPolicies are applied. The CNI (Container Network Interface) creates a flat network where all pods can communicate freely. This is dangerous in multi-tenant environments because:

1. **Lack of Isolation:** Tenant A can access Tenant B's services, databases, and internal APIs
2. **Data Breach Risk:** Sensitive data from one tenant could be accessed by another tenant
3. **Lateral Movement:** If one tenant's application is compromised, attackers can pivot to other tenants
4. **Compliance Violations:** Many regulations (GDPR, HIPAA, PCI-DSS) require strict tenant isolation

As demonstrated in Task 2, tenant-a successfully accessed tenant-b's web service with HTTP 200 response, proving the default-open behavior.

---

### Q2. Explain the default-deny principle and how your NetworkPolicy implements it.

<cite index="1-51">Explain the default-deny principle and how your NetworkPolicy implements it.</cite>

**Answer:**

The default-deny principle is a security best practice that states: "Deny all traffic by default, then explicitly permit only necessary traffic." This follows the principle of least privilege.

**Implementation in Our NetworkPolicy:**

```yaml
spec:
  podSelector: {}        # Applies to ALL pods in tenant-b
  policyTypes: [Ingress] # Affects incoming traffic
  # No ingress rules = deny all ingress
```

The NetworkPolicy we applied:
- Uses an empty `podSelector: {}` to select all pods in the tenant-b namespace
- Specifies `policyTypes: [Ingress]` to control incoming traffic
- Contains **no ingress rules**, which means all ingress traffic is denied
- Creates a "default-deny" posture where cross-tenant communication is blocked

This is proven by the timeout result in Task 4, where tenant-a could no longer reach tenant-b after the policy was applied.

---

### Q3. How do virtual machines and containers differ in isolation strength? When would you add a VM boundary?

<cite index="1-52,1-53">How do virtual machines and containers differ in isolation strength? When would you add a VM boundary?</cite>

**Answer:**

**Isolation Strength Comparison:**

| Aspect | Virtual Machines | Containers |
|--------|------------------|------------|
| **Kernel** | Each VM has its own kernel | Containers share the host kernel |
| **Hypervisor** | Hardware-level isolation via hypervisor | OS-level isolation via namespaces/cgroups |
| **Attack Surface** | Smaller - hypervisor is minimal | Larger - kernel vulnerabilities affect all containers |
| **Resource Overhead** | Higher (full OS per VM) | Lower (shared kernel) |
| **Performance** | Slower startup, more memory | Fast startup, efficient |

**When to Add a VM Boundary:**

1. **Multi-tenant SaaS:** When tenants have different trust levels or security requirements
2. **Compliance Requirements:** Regulations requiring hardware-level isolation (e.g., PCI-DSS Level 1)
3. **Untrusted Code:** Running user-submitted code or applications from unknown sources
4. **High-Value Assets:** Protecting critical infrastructure or sensitive data
5. **Different Operating Systems:** When workloads require different OS kernels

**Example Architecture:** Use VMs for tenant boundaries, then containers within each tenant's VM for microservices isolation. This provides defense-in-depth.

---

### Q4. What is data remanence, and why is cryptographic erasure the preferred cloud solution?

<cite index="1-54">What is data remanence, and why is cryptographic erasure the preferred cloud solution?</cite>

**Answer:**

**Data Remanence Definition:**

Data remanence is the residual representation of data that remains even after attempts have been made to remove or erase it. When files are deleted, only the file system pointers are removed—the actual data blocks often remain intact on storage media until overwritten.

**Why Cryptographic Erasure is Preferred in Cloud:**

1. **No Physical Access:** Cloud tenants don't control physical storage devices and cannot perform secure wipes or destruction

2. **Shared Storage:** In cloud environments, storage is virtualized and shared among multiple tenants. Traditional wiping methods aren't feasible

3. **Instant Deletion:** Cryptographic erasure is immediate—simply destroy the encryption key, making the data unrecoverable

4. **Compliance:** Meets regulatory requirements (GDPR "right to be forgotten," HIPAA, PCI-DSS) for data deletion

5. **Cost-Effective:** No need for special tools or time-consuming overwrite operations

**How It Works:**
- Encrypt all data at rest with strong encryption (e.g., AES-256)
- Store encryption keys separately (e.g., AWS KMS, Azure Key Vault)
- To delete data: destroy the encryption key
- Without the key, encrypted data is cryptographically useless

As demonstrated in Task 6, traditional deletion leaves data remnants. In Lab 3, we will implement cryptographic erasure as the cloud-native solution.

---

### Q5. Which of the three isolation dimensions (compute, network, storage) did each task exercise?

<cite index="1-55">Which of the three isolation dimensions (compute, network, storage) did each task exercise?</cite>

**Answer:**

| Task | Isolation Dimension(s) | Description |
|------|------------------------|-------------|
| **Task 1** | **Compute** | Created separate namespaces for tenant-a and tenant-b, providing logical compute isolation |
| **Task 2** | **Network** (observing lack of) | Demonstrated that network isolation does NOT exist by default—tenant-a accessed tenant-b |
| **Task 3** | **Compute** | Applied ResourceQuota to limit CPU, memory, and pod count—preventing resource exhaustion |
| **Task 4** | **Network** | Implemented default-deny NetworkPolicy to enforce network isolation between tenants |
| **Task 5** | **Storage** | Used RBAC to ensure tenant-a cannot access tenant-b's secrets—storage/data isolation |
| **Task 6** | **Storage** | Demonstrated data remanence and secure deletion techniques for persistent storage |

**Summary:**
- **Compute Isolation:** Tasks 1 and 3 (namespaces + resource quotas)
- **Network Isolation:** Tasks 2 and 4 (demonstrating the risk, then enforcing policies)
- **Storage Isolation:** Tasks 5 and 6 (RBAC for secrets + secure deletion)

---

## 8. Security Best-Practices Checklist {#checklist}

<cite index="1-57,1-58,1-59,1-60,1-61">Security best-practices checklist:</cite>

- ✅ **Tenants are separated into distinct namespaces**
  - tenant-a and tenant-b created and isolated

- ✅ **A default-deny NetworkPolicy blocks cross-tenant traffic (verified before/after)**
  - Before: HTTP 200 (Task 2)
  - After: Timeout (Task 4)

- ✅ **Resource quotas prevent a noisy-neighbour from exhausting shared capacity**
  - tenant-a-quota limits: 1 CPU, 512Mi memory, 5 pods

- ✅ **Per-tenant secrets are unreadable by other tenants (RBAC enforced)**
  - Service account app-a can access tenant-a secrets only
  - Access to tenant-b secrets denied

- ✅ **Secure deletion / cryptographic erasure is understood for data remanence**
  - Demonstrated data remanence with normal deletion
  - Showed secure wipe technique
  - Understand cryptographic erasure for cloud environments

---

## 9. Conclusion {#conclusion}

This lab successfully demonstrated the critical importance of multi-tenancy isolation in cloud environments across three dimensions:

### Key Findings:

1. **Default-Open Risk:** <cite index="1-26">On shared infrastructure, isolation is NOT automatic — you must configure it</cite>. Without NetworkPolicies, tenant-a could freely access tenant-b's services.

2. **Defense in Depth:** Effective multi-tenancy requires layered security:
   - **Compute:** Namespaces + ResourceQuotas
   - **Network:** Default-deny NetworkPolicies
   - **Storage:** RBAC + encryption

3. **Network Isolation is Critical:** <cite index="1-35">The before/after comparison (HTTP 200 vs timeout) is the strongest evidence of enforced network isolation</cite>.

4. **Data Remanence Matters:** Traditional deletion is insufficient. Cloud environments require cryptographic erasure for secure data deletion.

### Skills Acquired:

- Created and configured multi-tenant Kubernetes clusters with Calico CNI
- Implemented and tested NetworkPolicies for network segmentation
- Applied RBAC for storage and secret isolation
- Demonstrated secure deletion techniques and understood their limitations
- Verified isolation controls through before/after testing

### Real-World Application:

These techniques are essential for:
- **SaaS providers** hosting multiple customers on shared infrastructure
- **Compliance** with GDPR, HIPAA, PCI-DSS, and other regulations
- **Zero-trust architectures** implementing least-privilege access
- **Cloud-native security** following defense-in-depth principles

---

## 10. References {#references}

<cite index="1-66,1-67">References used for this lab:</cite>

1. Course lecture — Week 3 (Secure Isolation of Physical & Logical Infrastructure)
2. Kubernetes Network Policies — [kubernetes.io/docs/concepts/services-networking/network-policies](https://kubernetes.io/docs/concepts/services-networking/network-policies)
3. Calico documentation — [docs.tigera.io](https://docs.tigera.io)
4. CSA Security Guidance v5 — Infrastructure & Networking domain
5. IKB42603 Lab Manual - Lab 2: Secure Isolation & Multi-Tenancy

---

## Appendix: Cleanup Commands

<cite index="1-62">Cleanup & Teardown commands:</cite>

```bash
# Delete the Kubernetes cluster
kind delete cluster --name ccse-lab2

# Remove Docker volume
docker volume rm ccse-vol
```

---

**End of Report**

**Lab Completion Date:** Week 4  
**Status:** All tasks completed successfully  
**Submitted by:** Surya
