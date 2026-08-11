# Lab 2: Secure Isolation and Multi-Tenancy Report

## Student Information

- **Name:** Surya Giri A/L Shanker
- **Student ID:** 52215124335
- **Course:** IKB42603 Cloud Computing Security Essentials
- **Lab Task:** Lab 2 - Secure Isolation & Multi-Tenancy
- **Lecturer Name:** Prof. Dr. Shahrulniza Musa

---

## Overview

This report documents the implementation and verification of secure isolation and multi-tenancy practices in cloud computing environments using Docker and Kubernetes. The lab was conducted in two sessions over two weeks. Session A focused on establishing compute isolation fundamentals by creating tenant namespaces, deploying containerized workloads, observing the default-open security risk inherent in shared infrastructure, and implementing resource quotas to prevent noisy neighbor problems. Session B extended these security controls by implementing network isolation through default-deny NetworkPolicy enforcement, storage isolation using Kubernetes RBAC for secrets, and demonstrating data remanence risks along with secure deletion techniques. The purpose of this comprehensive lab was to develop practical understanding of the three critical dimensions of multi-tenancy isolation: compute, network, and storage. Effective isolation is essential because in cloud environments, multiple tenants share physical infrastructure, and without proper security controls, one tenant could access, interfere with, or compromise another tenant's resources, leading to data breaches, compliance violations, and service disruptions. Both sessions were executed using command-line tools (kubectl for Kubernetes, docker for containers) against local development environments created with kind (Kubernetes IN Docker) and Calico CNI for NetworkPolicy enforcement, and each task was systematically documented with terminal commands and screenshots as evidence of successful implementation.

---

## Objectives

The objectives of this lab across both sessions are:

**Session A Objectives (Week 3 - Compute Isolation & Default-Open Risk):**

- Install and verify Docker to support container-based infrastructure simulation.
- Create a Kubernetes cluster with Calico CNI to enable NetworkPolicy enforcement that will be tested in Session B.
- Implement namespace-based tenant separation to logically isolate two customers on shared infrastructure.
- Deploy simple web server workloads for each tenant to simulate multi-tenant application environments.
- Observe and document the default-open security risk by proving that cross-tenant network communication is allowed by default.
- Apply resource quotas to contain "noisy neighbor" problems and prevent resource exhaustion attacks.

**Session B Objectives (Week 4 - Network & Storage Isolation):**

- Implement default-deny network isolation by applying NetworkPolicy to block cross-tenant traffic.
- Test and verify network segmentation by re-running the same probe that succeeded in Session A and confirming it now fails.
- Enforce storage isolation by using Kubernetes RBAC to ensure tenant-scoped secrets cannot be accessed cross-tenant.
- Demonstrate data remanence by showing that deleted files may persist in storage.
- Implement secure deletion techniques by overwriting data before removal.
- Understand cryptographic erasure as the cloud-native solution to data remanence.

**Common Objectives:**

- Document all procedures clearly using terminal commands and screenshots as evidence of implementation.
- Understand and apply defense-in-depth security principles across compute, network, and storage layers.
- Practice multi-tenancy isolation in safe local environments before applying to production systems.
- Develop practical skills in securing shared infrastructure for cloud service providers and SaaS platforms.

---

## Learning Outcomes

By completing this lab across both sessions, the student should be able to:

**Session A Outcomes (Compute Isolation & Default-Open Risk):**

- Understand the architecture of multi-tenant Kubernetes environments and the role of namespaces in logical separation.
- Configure Kubernetes clusters with CNI plugins (Calico) that support NetworkPolicy enforcement.
- Recognize the default-open behavior of Kubernetes networking and articulate why this is a critical security risk.
- Implement ResourceQuotas to enforce compute, memory, and pod count limits per tenant namespace.
- Deploy and expose services within Kubernetes namespaces to simulate real-world multi-tenant applications.

**Session B Outcomes (Network & Storage Isolation):**

- Implement default-deny NetworkPolicy rules to enforce network segmentation between tenants.
- Demonstrate before-and-after testing methodology to prove that security controls are functioning correctly.
- Configure Kubernetes RBAC (Roles, RoleBindings, ServiceAccounts) to enforce storage and secret isolation.
- Test authorization boundaries using kubectl auth can-i to verify that access controls work as intended.
- Understand data remanence risks and the limitations of traditional file deletion in cloud environments.
- Recognize cryptographic erasure as the preferred cloud-native approach to secure data deletion.

**Common Outcomes:**

- Recognize that multi-tenancy isolation requires layered security controls across multiple dimensions (compute, network, storage).
- Document technical security procedures clearly using terminal commands and visual evidence.
- Understand the principle of "deny by default, permit by exception" as a fundamental security architecture pattern.
- Apply the principle of least privilege in Kubernetes environments through namespace scoping and RBAC.

---

## Environment and Prerequisites

The lab was conducted on a Kali Linux environment with Docker installed and internet access for downloading container images. The following tools and conditions were required before starting the lab:

**Session A Prerequisites (Compute Isolation):**

- Docker installed and running to support container-based infrastructure.
- kind (Kubernetes IN Docker) tool installed for creating local Kubernetes clusters.
- kubectl command-line tool installed for interacting with Kubernetes clusters.
- Internet connectivity for downloading Calico CNI manifests and container images (nginx, curlimages/curl).
- Basic understanding of container concepts and Kubernetes architecture.

**Session B Prerequisites (Network & Storage Isolation):**

- Existing Kubernetes cluster from Session A (or recreation of the cluster with Calico CNI).
- Understanding of Kubernetes NetworkPolicy concepts and YAML syntax.
- Understanding of Kubernetes RBAC concepts including ServiceAccounts, Roles, and RoleBindings.
- Docker CLI for executing data remanence demonstration commands.

**Common Prerequisites:**

- Sufficient system resources (CPU, memory, disk) to run Kubernetes control plane and worker nodes.
- Administrative privileges on the local system for managing Docker and Kubernetes resources.
- Terminal or command-line interface with PowerShell or bash shell access.

---

## Session A (Week 3) — Compute Isolation & the Default-Open Risk

### Setup — Cluster with Policy Enforcement

Kubernetes by default uses a simple networking model where all pods can communicate with all other pods regardless of namespace boundaries. While this simplifies development, it creates significant security risks in multi-tenant environments where customer isolation is required. To enforce NetworkPolicy rules that will block cross-tenant traffic, a Container Network Interface (CNI) plugin that supports policy enforcement must be installed. The default kind networking does not enforce NetworkPolicy, so this lab uses Calico CNI, which is a production-grade networking solution that provides NetworkPolicy enforcement, network security, and observability. Creating the cluster with the default CNI disabled and then installing Calico ensures that the network isolation controls demonstrated in Session B will actually take effect. This setup step is critical because without proper CNI configuration, NetworkPolicy resources would be accepted by the Kubernetes API but silently ignored, creating a false sense of security.

#### Purpose

- Create a local Kubernetes cluster with default CNI disabled to prepare for Calico installation.
- Install Calico CNI to enable NetworkPolicy enforcement capabilities.
- Verify that the Calico components are running and healthy before proceeding with tenant deployment.
- Establish a production-like environment where network isolation rules will be enforced.

#### Terminal Commands

```bash
# Create a cluster with the default CNI disabled, then install Calico
cat <<EOF | kind create cluster --name ccse-lab2 --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
EOF

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
kubectl -n kube-system rollout status daemonset/calico-node --timeout=180s
```

#### Explanation of the Commands

- `cat <<EOF | kind create cluster --name ccse-lab2 --config=-` creates a new kind cluster named "ccse-lab2" using inline configuration that disables the default CNI (networking.disableDefaultCNI: true) and specifies a pod subnet range (192.168.0.0/16) that is compatible with Calico's default configuration.
- `kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml` downloads and applies the Calico CNI manifest from the official Calico project repository, which deploys all necessary components including calico-node DaemonSet, calico-kube-controllers Deployment, and required CustomResourceDefinitions for NetworkPolicy enforcement.
- `kubectl -n kube-system rollout status daemonset/calico-node --timeout=180s` waits for the Calico node DaemonSet to successfully roll out across all cluster nodes with a 180-second timeout, ensuring that the CNI components are fully operational before proceeding with workload deployment.

#### Evidence

![Setup Cluster Creation](Setup%20Cluster%20%20with%20policy%20Enforcement%201.png)

The screenshot shows the successful creation of the kind cluster with the control plane being prepared, nodes starting, and the kubectl context being set to "kind-ccse-lab2".

![Calico Installation and Verification](Setup%20Cluster%20with%20policy%20Enforcement%202.png)

The screenshot demonstrates the successful application of the Calico manifest with multiple CustomResourceDefinitions, ServiceAccounts, and other resources being created, followed by the successful rollout of the calico-node DaemonSet, confirming that NetworkPolicy enforcement is now enabled.

#### Notes

The Kubernetes cluster was successfully created with Calico CNI installed and verified. Unlike the default kind networking which does not enforce NetworkPolicy, this cluster will actively enforce network segmentation rules when they are applied in Session B. The Calico components running in the kube-system namespace provide the network policy engine that intercepts and filters network traffic based on NetworkPolicy resources. This setup demonstrates an important principle in cloud security: security controls must be enabled at the infrastructure layer before they can be enforced at the policy layer. In production environments, choosing the right CNI plugin is a critical architectural decision that affects security, performance, and operational capabilities.

---

### Step 1: Task 1 - Two Tenants on One Cluster

In cloud computing, multi-tenancy refers to an architecture where multiple customers (tenants) share the same physical infrastructure while maintaining logical isolation from each other. This approach enables cloud providers to achieve economies of scale and resource efficiency, but it also introduces security challenges because tenant isolation must be actively enforced rather than being inherently provided by separate hardware. In Kubernetes, namespaces provide the fundamental mechanism for multi-tenancy by creating logical boundaries within a single cluster. Each namespace acts as a virtual cluster with its own resource quotas, network policies, and access controls. In this task, two separate namespaces were created to represent two different customers (tenant-a and tenant-b), and each tenant was given a simple nginx web server deployment to simulate real-world application workloads. This task establishes the foundation for demonstrating that namespace separation alone is insufficient for security—network isolation must be explicitly configured.

#### 1.1 Create Tenant Namespaces

The first step in implementing multi-tenancy is creating separate namespaces for each tenant. Namespaces are Kubernetes' native mechanism for dividing cluster resources between multiple users, teams, or customers.

##### Purpose

- Create logical separation between two tenants using Kubernetes namespaces.
- Establish isolated environments for deploying tenant-specific workloads.
- Prepare the foundation for applying tenant-scoped security policies.

##### Terminal Commands

```bash
kubectl create namespace tenant-a
kubectl create namespace tenant-b
```

##### Explanation of the Commands

- `kubectl create namespace tenant-a` creates a new namespace named "tenant-a" that will serve as an isolated environment for the first customer's workloads, with the ability to apply namespace-scoped resource quotas, network policies, and RBAC rules.
- `kubectl create namespace tenant-b` creates a second namespace named "tenant-b" for the second customer, demonstrating how multiple tenants can be isolated within the same physical cluster infrastructure.

##### Evidence

![Tenant Namespaces Created](Task%201%20Two%20tenants%20on%20One%20Cluster.png)

The screenshot confirms that both tenant-a and tenant-b namespaces were successfully created.

#### 1.2 Deploy Web Servers for Each Tenant

After creating the namespace boundaries, the next step is deploying actual application workloads within each tenant namespace. In this lab, simple nginx web servers serve as representative applications that will be used to test network connectivity and isolation.

##### Purpose

- Deploy containerized web server applications within each tenant namespace.
- Create Kubernetes Services to expose the web servers internally within the cluster.
- Establish workloads that can be used to test cross-tenant network connectivity.
- Simulate real-world multi-tenant application deployments.

##### Terminal Commands

```bash
# Deploy a simple web server for each tenant
kubectl -n tenant-a create deployment web --image=nginx
kubectl -n tenant-b create deployment web --image=nginx
kubectl -n tenant-a expose deployment web --port=80
kubectl -n tenant-b expose deployment web --port=80
kubectl get pods,svc -n tenant-a
```

##### Explanation of the Commands

- `kubectl -n tenant-a create deployment web --image=nginx` creates a Deployment resource in tenant-a namespace that manages a pod running the nginx web server container image, providing a representative application workload for testing.
- `kubectl -n tenant-b create deployment web --image=nginx` creates an identical deployment in tenant-b namespace, simulating a second customer running similar infrastructure on the same shared cluster.
- `kubectl -n tenant-a expose deployment web --port=80` creates a Kubernetes Service in tenant-a that provides a stable ClusterIP endpoint for accessing the nginx deployment on port 80, making the application accessible within the cluster network.
- `kubectl -n tenant-b expose deployment web --port=80` creates a corresponding service in tenant-b, giving the second tenant's application a stable network endpoint.
- `kubectl get pods,svc -n tenant-a` lists both pods and services in tenant-a namespace to verify that the deployment succeeded and the application is running with an accessible service endpoint.

##### Evidence

![Web Server Deployments](Task%201%20Deploy%20a%20simple%20web%20server%20for%20each%20tenant.png)

The screenshot shows the successful creation of deployments and services in both namespaces, with the final output displaying the running pod and service in tenant-a, including the ClusterIP address (10.96.111.229) that will be used for connectivity testing.

#### Notes

Two separate tenant environments were successfully established with their own application workloads and network services. This configuration represents a typical multi-tenant SaaS architecture where multiple customers run similar applications on shared infrastructure. The key observation at this point is that while the namespaces provide logical separation and resource organization, they do not inherently provide network isolation—both services are accessible from anywhere within the cluster network. This default-open behavior will be demonstrated in Task 2 and addressed with NetworkPolicy in Session B. In production environments, each tenant namespace would typically also have ResourceQuotas, LimitRanges, NetworkPolicies, and RBAC policies applied to enforce comprehensive isolation and prevent cross-tenant interference.

---

### Step 2: Task 2 - Observe the Default-Open Risk

One of the most critical security risks in default Kubernetes deployments is that pod-to-pod networking is completely open by default. Without NetworkPolicy enforcement, any pod in any namespace can communicate with any other pod in any other namespace, regardless of tenant boundaries. This behavior is convenient for development but creates serious security vulnerabilities in production multi-tenant environments. If one tenant's application is compromised, an attacker could use that foothold to scan for and access services belonging to other tenants, leading to data breaches, lateral movement attacks, and compliance violations. This task demonstrates the default-open risk by proving that tenant-a can successfully connect to tenant-b's web service, even though they are logically separated into different namespaces. This before-state documentation is critical because it provides a baseline for comparison when NetworkPolicy is applied in Session B.

#### Purpose

- Demonstrate that default Kubernetes networking allows cross-tenant communication.
- Prove the security risk by showing tenant-a successfully accessing tenant-b's service.
- Document the "before" state for comparison with NetworkPolicy enforcement in Session B.
- Establish evidence of the multi-tenancy security problem that needs to be solved.

#### Terminal Commands

```bash
# Get tenant-b's service IP
kubectl get svc web -n tenant-b -o jsonpath='{.spec.clusterIP}'; echo

# From tenant-a, curl tenant-b (replace <B_IP> with actual IP)
kubectl -n tenant-a run probe --rm -it --image=curlimages/curl --restart=Never \
  -- curl -s -m 5 http://10.96.253.95 -o /dev/null -w 'HTTP %{http_code}\n'
```

#### Explanation of the Commands

- `kubectl get svc web -n tenant-b -o jsonpath='{.spec.clusterIP}'; echo` retrieves the ClusterIP address of tenant-b's web service using JSONPath query syntax, which extracts just the IP address from the service specification, making it easy to use in subsequent connectivity tests.
- `kubectl -n tenant-a run probe --rm -it --image=curlimages/curl --restart=Never` creates a temporary pod in tenant-a namespace using the curlimages/curl container image, with flags that automatically remove the pod after execution (--rm), attach to an interactive terminal (-it), and prevent pod restart on failure (--restart=Never).
- `-- curl -s -m 5 http://10.96.253.95 -o /dev/null -w 'HTTP %{http_code}\n'` executes a curl command inside the probe pod that attempts to connect to tenant-b's service IP, with silent mode (-s), a 5-second timeout (-m 5), output redirected to /dev/null, and a custom output format showing only the HTTP status code, making it easy to verify successful connection.

#### Evidence

![Tenant-B Service IP](Task%202%20Observe%20the%20Default-Open%20Risk%201.png)

The screenshot shows the retrieval of tenant-b's service ClusterIP address (10.96.253.95), which will be used as the target for the cross-tenant connectivity test.

![Cross-Tenant Access Succeeds](Task%202%20Observe%20the%20Default-Open%20Risk%202.png)

The screenshot demonstrates that the probe pod from tenant-a successfully connected to tenant-b's web service, receiving an HTTP 200 OK response, proving that cross-tenant network communication is allowed by default and confirming the security risk.

#### Notes

The default-open risk was successfully demonstrated with clear evidence that tenant-a can reach tenant-b's services despite namespace separation. The HTTP 200 response proves that the nginx web server in tenant-b processed the request from tenant-a, confirming that no network isolation exists by default. This behavior represents a significant security vulnerability in multi-tenant environments because it enables potential attack scenarios including lateral movement, data exfiltration, and service enumeration across tenant boundaries.
In a production scenario where tenant-b hosts sensitive customer data or payment processing systems, this level of access from tenant-a would be unacceptable from both security and compliance perspectives. The principle of "deny by default, permit by exception" is not implemented in default Kubernetes networking, requiring explicit NetworkPolicy configuration to enforce proper isolation. This evidence will be contrasted with the timeout result in Session B Task 4, where the same probe will fail after NetworkPolicy is applied, demonstrating effective network segmentation.

---

### Step 3: Task 3 - Contain the Noisy Neighbour (Resource Quotas)

Multi-tenancy isolation is not limited to network security; it also encompasses resource management to prevent one tenant from consuming excessive cluster resources and degrading performance for other tenants. The "noisy neighbor" problem occurs when one tenant's workloads consume disproportionate amounts of CPU, memory, or other resources, causing resource starvation for other tenants sharing the same infrastructure. This can happen accidentally through misconfigured autoscaling or memory leaks, or maliciously through denial-of-service attacks. Kubernetes ResourceQuotas provide the mechanism to enforce hard limits on resource consumption per namespace, ensuring fair resource allocation and preventing one tenant from monopolizing shared infrastructure. In this task, a ResourceQuota was applied to tenant-a that limits CPU requests, memory requests, and the number of pods, demonstrating how compute isolation can be enforced at the namespace level.

#### Purpose

- Apply resource limits to prevent one tenant from exhausting shared cluster resources.
- Demonstrate Kubernetes ResourceQuota as a compute isolation mechanism.
- Enforce fair resource allocation across multiple tenants.
- Prevent denial-of-service attacks through resource exhaustion.

#### Terminal Commands

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

kubectl describe resourcequota tenant-a-quota -n tenant-a
```

#### Explanation of the Commands

- `cat <<EOF | kubectl apply -f -` uses a here-document to provide YAML configuration inline to kubectl, creating a ResourceQuota resource that will enforce hard limits on resource consumption within the tenant-a namespace.
- The ResourceQuota specification defines three hard limits: requests.cpu limits the total CPU that can be requested by all pods in the namespace to 1 core, requests.memory limits total memory requests to 512 megabytes, and pods limits the maximum number of pods that can exist in the namespace to 5, collectively preventing resource exhaustion.
- `kubectl describe resourcequota tenant-a-quota -n tenant-a` displays detailed information about the ResourceQuota including the hard limits configured and the current resource usage against those limits, providing visibility into resource consumption and enforcement.

#### Evidence

![ResourceQuota Creation](Task%203%20Contain%20the%20Noisy%20Neighbour%201.png)

The screenshot shows the successful creation of the ResourceQuota named "tenant-a-quota" in the tenant-a namespace.

![ResourceQuota Details](Task%203%20Contain%20the%20Noisy%20Neighbour%202.png)

The screenshot displays the detailed ResourceQuota configuration showing the hard limits (pods: 5, requests.cpu: 1, requests.memory: 512Mi) and the current usage (pods: 1, requests.cpu: 0, requests.memory: 0), confirming that the quota is active and tracking resource consumption.

#### Notes

The ResourceQuota was successfully applied to tenant-a, establishing compute isolation boundaries that prevent this tenant from consuming more than 1 CPU core, 512Mi of memory, or running more than 5 pods. The "Used" column showing 0 for CPU and memory requests indicates that the existing nginx pod does not have resource requests defined, which is common in development but should always be specified in production to ensure ResourceQuota enforcement works correctly.
In production environments, ResourceQuotas should be combined with LimitRanges (which enforce default resource requests and limits on individual containers) to ensure comprehensive resource management. This quota mechanism protects tenant-b and other tenants from being impacted by excessive resource consumption in tenant-a, demonstrating that multi-tenancy security requires controls at multiple layers: network (NetworkPolicy), compute (ResourceQuota), and storage (RBAC), which will be addressed in Session B. This completes Session A, where the security problem (default-open networking) and one dimension of the solution (resource quotas) were demonstrated.

---

## Session B (Week 4) — Network & Storage Isolation

### Step 4: Task 4 - Default-Deny Network Isolation

Having established the baseline security risk in Session A where cross-tenant network communication was permitted by default, Session B focuses on implementing the security controls that enforce proper isolation. NetworkPolicy is Kubernetes' native mechanism for controlling network traffic between pods, namespaces, and external endpoints. A NetworkPolicy functions similarly to a firewall, defining rules that specify which network connections are allowed or denied. The fundamental principle of secure network architecture is "deny by default, permit by exception" (also known as whitelisting), where all traffic is blocked unless explicitly allowed by a policy rule. This approach is more secure than "permit by default, deny by exception" (blacklisting) because it ensures that only known-good traffic patterns are allowed, preventing unauthorized access through overlooked or misconfigured rules. In this task, a default-deny ingress policy was applied to tenant-b that blocks all incoming traffic from any source, and then the same connectivity test from Session A was repeated to prove that the network isolation is now enforced.

#### Purpose

- Implement default-deny NetworkPolicy to block all ingress traffic to tenant-b.
- Demonstrate the principle of "deny by default, permit by exception" in network security.
- Re-test cross-tenant connectivity to prove that the policy blocks the traffic.
- Provide before-and-after evidence of effective network segmentation.

#### Terminal Commands

```bash
# Deny ALL ingress into tenant-b
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

# Re-run the SAME probe from Task 2 — it should now TIME OUT / fail
kubectl -n tenant-a run probe --rm -it --image=curlimages/curl --restart=Never \
  -- curl -s -m 5 http://10.96.253.95 -o /dev/null -w 'HTTP %{http_code}\n'
```

#### Explanation of the Commands

- `apiVersion: networking.k8s.io/v1` specifies that this is a NetworkPolicy resource using the stable v1 API version from the networking API group, which has been supported since Kubernetes 1.7.
- `podSelector: {}` with an empty selector means this policy applies to ALL pods in the tenant-b namespace, creating a namespace-wide default-deny rule rather than targeting specific pods.
- `policyTypes: [Ingress]` indicates that this policy controls incoming traffic to the selected pods, and because no ingress rules are specified in the spec, all ingress traffic is denied by default according to Kubernetes NetworkPolicy semantics.
- The kubectl apply command creates the NetworkPolicy resource, which is immediately enforced by the Calico CNI plugin that was installed during cluster setup.
- The probe command is identical to Task 2, running a curl test from tenant-a to tenant-b's service IP (10.96.253.95), but this time the request should timeout because the NetworkPolicy now blocks ingress traffic.

#### Evidence

![NetworkPolicy Creation](Task%204%20Default-Deny%20Network%20Isolation%201.png)

The screenshot shows the successful creation of the "default-deny-ingress" NetworkPolicy in tenant-b namespace.

![Cross-Tenant Access Blocked](Task%204%20Default-Deny%20Network%20Isolation%202.png)

The screenshot demonstrates that the probe pod from tenant-a now times out when attempting to connect to tenant-b's service, with the error message "timed out waiting for the condition" indicating that the NetworkPolicy successfully blocked the connection that previously succeeded in Task 2.

#### Notes

The default-deny NetworkPolicy successfully enforced network isolation between tenants. Comparing the results side-by-side shows the effectiveness of the control:
- **Before NetworkPolicy (Task 2):** HTTP 200 - tenant-a successfully accessed tenant-b's service
- **After NetworkPolicy (Task 4):** Timeout/Error - tenant-a cannot reach tenant-b's service

This before-and-after comparison is the strongest evidence of enforced network isolation and demonstrates that NetworkPolicy is functioning correctly.
The timeout behavior indicates that network packets from tenant-a attempting to reach tenant-b are being silently dropped by Calico's network policy enforcement engine, preventing any communication. In production environments, additional NetworkPolicy rules would be created to selectively allow necessary traffic (for example, allowing ingress from an ingress controller or from specific namespaces for legitimate inter-service communication), implementing the "permit by exception" part of the security principle. This task demonstrates that network isolation is not automatic in Kubernetes—it must be explicitly configured through NetworkPolicy resources and requires a CNI plugin that enforces those policies.

---

### Step 5: Task 5 - Storage & Secret Isolation

Network isolation alone is insufficient for comprehensive multi-tenancy security; storage and data access must also be controlled to prevent tenants from accessing each other's sensitive information. Kubernetes Secrets are resources that store sensitive data such as passwords, API keys, certificates, and tokens. By default, secrets are stored in the cluster's etcd database and can be mounted into pods as files or environment variables. However, without proper RBAC configuration, a user or service account with access to one namespace could potentially list or retrieve secrets from other namespaces, leading to credential theft and unauthorized access. Kubernetes Role-Based Access Control (RBAC) provides the mechanism to enforce storage isolation by defining exactly which identities (users, groups, service accounts) can perform which actions (get, list, create, delete) on which resources (secrets, configmaps, pods) within which namespaces. In this task, secrets were created in both tenant namespaces, a service account with limited permissions was created in tenant-a, and authorization checks were performed to prove that the service account can access secrets in its own namespace but is denied access to secrets in tenant-b.

#### 5.1 Create Secrets in Each Tenant Namespace

The first step in demonstrating storage isolation is creating test secrets in each tenant namespace that will serve as sensitive data to be protected by RBAC policies.

##### Purpose

- Create Kubernetes Secrets in both tenant namespaces to simulate sensitive data storage.
- Establish test data that will be used to verify RBAC access control enforcement.
- Represent real-world scenarios where each tenant stores confidential information like database passwords or API keys.

##### Terminal Commands

```bash
# Create a secret in each tenant
kubectl -n tenant-a create secret generic data --from-literal=value=SECRET_A
kubectl -n tenant-b create secret generic data --from-literal=value=SECRET_B
```

##### Explanation of the Commands

- `kubectl -n tenant-a create secret generic data --from-literal=value=SECRET_A` creates a generic Secret resource named "data" in tenant-a namespace containing a single key-value pair (value=SECRET_A), simulating sensitive tenant-specific configuration or credentials.
- `kubectl -n tenant-b create secret generic data --from-literal=value=SECRET_B` creates an equivalent secret in tenant-b namespace with different sensitive data (SECRET_B), representing the second tenant's confidential information that should not be accessible to tenant-a.

##### Evidence

![Secrets Created](Task%205%20Storage%20&%20Secret%20Isolation%201.png)

The screenshot confirms that both secrets were successfully created in their respective namespaces.

#### 5.2 Create Service Account and RBAC Configuration for Tenant-A

After creating the secrets to protect, the next step is establishing an identity (service account) in tenant-a and granting it limited permissions through RBAC to demonstrate namespace-scoped access control.

##### Purpose

- Create a service account in tenant-a that represents an application or user identity.
- Define a Role with permission to read secrets within tenant-a namespace only.
- Bind the Role to the service account using a RoleBinding to activate permissions.
- Demonstrate proper RBAC configuration following the principle of least privilege.

##### Terminal Commands

```bash
# A service account scoped to tenant-a only
kubectl -n tenant-a create serviceaccount app-a
kubectl -n tenant-a create role reader --verb=get --resource=secrets
kubectl -n tenant-a create rolebinding rb --role=reader --serviceaccount=tenant-a:app-a
```

##### Explanation of the Commands

- `kubectl -n tenant-a create serviceaccount app-a` creates a new ServiceAccount named "app-a" in tenant-a namespace, which is a Kubernetes identity that can be granted permissions through RBAC and can be used by pods or for authorization testing.
- `kubectl -n tenant-a create role reader --verb=get --resource=secrets` defines a Role named "reader" that permits only the "get" verb on "secrets" resources, meaning this role allows reading individual secrets by name but not listing all secrets or performing write operations.
- `kubectl -n tenant-a create rolebinding rb --role=reader --serviceaccount=tenant-a:app-a` creates a RoleBinding that connects the "reader" role to the "app-a" service account, granting that identity the permissions defined in the role within the tenant-a namespace scope.

##### Evidence

![RBAC Configuration](Task%205%20Storage%20&%20Secret%20Isolation%202.png)

The screenshot shows the successful creation of the service account, role, and role binding in tenant-a namespace.

#### 5.3 Test Authorization with kubectl auth can-i

The final step is testing the RBAC configuration to verify that the service account has the intended permissions in its own namespace but is denied access to other namespaces.

##### Purpose

- Verify that the app-a service account can access secrets in tenant-a namespace.
- Confirm that the same service account cannot access secrets in tenant-b namespace.
- Prove that RBAC enforces namespace-scoped access control boundaries.
- Validate that storage isolation is correctly implemented.

##### Terminal Commands

```bash
SA=system:serviceaccount:tenant-a:app-a
kubectl auth can-i get secrets -n tenant-a --as=$SA  # expect: yes
kubectl auth can-i get secrets -n tenant-b --as=$SA  # expect: no
```

##### Explanation of the Commands

- `SA=system:serviceaccount:tenant-a:app-a` creates a shell variable containing the fully qualified service account identifier using Kubernetes' standard naming convention (system:serviceaccount:namespace:name) for use in authorization testing.
- `kubectl auth can-i get secrets -n tenant-a --as=$SA` queries the Kubernetes authorization system to determine whether the app-a service account has permission to get secrets in tenant-a namespace, which should return "yes" because the RoleBinding grants the reader role with get permissions.
- `kubectl auth can-i get secrets -n tenant-b --as=$SA` tests whether the same service account can access secrets in tenant-b namespace, which should return "no" because the Role and RoleBinding are scoped only to tenant-a namespace, demonstrating proper isolation.

##### Evidence

![Authorization Test Results](Task%205%20Storage%20&%20Secret%20Isolation%203.png)

The screenshot confirms that the authorization tests returned the expected results: "yes" for tenant-a (access granted) and "no" for tenant-b (access denied), proving that RBAC correctly enforces storage isolation between tenants.

#### Notes

The storage and secret isolation was successfully implemented and verified using Kubernetes RBAC. The service account app-a in tenant-a can only access secrets within its own namespace and is explicitly denied access to tenant-b's secrets, even though both namespaces exist in the same cluster. This demonstrates that RBAC provides namespace-scoped authorization boundaries that enforce tenant isolation for sensitive data. In production environments, similar RBAC policies would be applied to all resource types (not just secrets) and to all service accounts and users, ensuring that every identity has explicitly defined, minimal permissions following the principle of least privilege. Combined with NetworkPolicy (network isolation) and ResourceQuota (compute isolation), RBAC (storage isolation) completes the three dimensions of multi-tenancy security required for production cloud environments.

---

### Step 6: Task 6 - Data Remanence & Secure Deletion

Data remanence is a security concern that occurs when data remains on storage media after deletion operations are performed. When files are "deleted" using standard operating system commands, typically only the file system metadata is updated to mark the space as available for reuse, but the actual data bytes remain physically present on the storage device until they are overwritten by new data. This means that deleted sensitive information can potentially be recovered using data recovery tools or forensic techniques, creating a data breach risk even after deletion appears complete. In cloud computing and shared infrastructure environments, this problem is compounded because storage resources are frequently reallocated between different tenants, creating scenarios where one tenant's deleted data could theoretically be recovered by a subsequent tenant if proper sanitization is not performed. In this task, data remanence was demonstrated by creating a file containing sensitive data, deleting it normally, and showing that the data persists and can be found. Then, secure deletion was demonstrated by overwriting data before deletion to ensure it cannot be recovered. Finally, the limitations of these approaches in cloud environments were discussed, leading to cryptographic erasure as the preferred solution.

#### Purpose

- Demonstrate that normal file deletion does not remove data from storage media.
- Show that sensitive data can persist after deletion and potentially be recovered.
- Implement secure deletion by overwriting data before removal.
- Understand cryptographic erasure as the cloud-native approach to data sanitization.

#### Terminal Commands

```bash
# Create a file, delete it normally, then show the bytes may persist
docker run --rm -v ccse-vol:/data alpine sh -c \
  'echo SENSITIVE-PATIENT-RECORD > /data/phi.txt; sync; rm /data/phi.txt; \
  grep -a SENSITIVE /data/* 2>/dev/null; echo scan-done'

# Secure wipe: overwrite before delete (shred)
docker run --rm -v ccse-vol:/data alpine sh -c \
  'echo SENSITIVE > /data/phi2.txt; sync; \
  dd if=/dev/zero of=/data/phi2.txt bs=1k count=1 conv=notrunc; rm /data/phi2.txt; \
  echo wiped'
```

#### Explanation of the Commands

- `docker run --rm -v ccse-vol:/data alpine sh -c` runs a temporary Alpine Linux container with a Docker volume named "ccse-vol" mounted at /data, providing isolated storage that persists between container runs for testing data remanence.
- `echo SENSITIVE-PATIENT-RECORD > /data/phi.txt; sync; rm /data/phi.txt` creates a file containing sensitive data, forces the write to disk (sync), then deletes the file using the standard rm command, simulating normal deletion behavior.
- `grep -a SENSITIVE /data/* 2>/dev/null; echo scan-done` attempts to search for the sensitive data string in the data directory treating all files as text (-a), demonstrating whether deleted data can still be found on the storage media, with errors redirected to /dev/null for cleaner output.
- `echo SENSITIVE > /data/phi2.txt; sync` creates another test file with sensitive data and syncs it to disk.
- `dd if=/dev/zero of=/data/phi2.txt bs=1k count=1 conv=notrunc` uses the dd command to overwrite the file content with zeros from /dev/zero, with a block size of 1 kilobyte, writing one block without truncating the file, effectively destroying the sensitive data before deletion.
- `rm /data/phi2.txt; echo wiped` removes the now-sanitized file and outputs a confirmation message, demonstrating secure deletion through overwriting.

#### Evidence

![Data Remanence and Secure Deletion](Task%206%20Data%20Remanence%20&%20Secure%20Deletion.png)

The screenshot shows the execution of both commands. The first command demonstrates that the "SENSITIVE-PATIENT-RECORD" data was not found after normal deletion (scan-done appears without grep results, though in some cases remnants might be detected). The second command shows the secure wipe process with dd successfully overwriting 1024 bytes (1.0KB) and the wiped confirmation, demonstrating proper secure deletion.

#### Notes

The data remanence demonstration highlights a critical security concern: standard deletion operations do not guarantee that sensitive data is truly removed from storage. While the grep search did not detect remnants in this specific test case, data remanence can occur depending on filesystem implementation, timing, and storage characteristics. The secure wipe approach using dd to overwrite before deletion is more reliable but has significant limitations in cloud environments.
In cloud storage systems, customers rarely have direct control over physical storage blocks because storage is virtualized and abstracted by the cloud provider. Traditional secure deletion techniques like multi-pass overwriting (DoD 5220.22-M standard) are impractical or impossible in cloud environments where storage is distributed, replicated, and managed by the provider. Additionally, modern storage technologies like SSDs with wear-leveling and cloud storage with automatic replication make it difficult to guarantee that all copies of data are overwritten. The preferred cloud-native solution is **cryptographic erasure** (also called crypto-shredding): all data is encrypted at rest with strong encryption algorithms like AES-256, encryption keys are stored separately from the data (for example, in AWS KMS, Azure Key Vault, or Google Cloud KMS), and when data needs to be securely deleted, the encryption key is destroyed rather than the data itself. Without the encryption key, the encrypted data is cryptographically useless and cannot be recovered even if the ciphertext is recovered from storage, effectively achieving secure deletion without needing to overwrite or physically destroy storage media. This approach will be explored in more detail in Lab 3, where encryption and key management will be implemented.

---

## Verification Commands

After completing all configuration tasks, verification commands were executed to confirm that all security controls are properly implemented and visible in the cluster configuration. These commands provide auditable evidence that NetworkPolicy and ResourceQuota resources exist and are configured correctly.

### Purpose

- Verify that NetworkPolicy resources are deployed and active in the cluster.
- Confirm that ResourceQuota is properly configured with correct limits.
- Provide auditable evidence of security control implementation.
- Enable troubleshooting by displaying complete resource configuration.

### Terminal Commands

```bash
kubectl get networkpolicy -A
kubectl describe resourcequota tenant-a-quota -n tenant-a
```

### Explanation of the Commands

- `kubectl get networkpolicy -A` lists all NetworkPolicy resources across all namespaces in the cluster (-A flag for all namespaces), showing policy names, pod selectors, and age, providing visibility into what network isolation rules are active cluster-wide.
- `kubectl describe resourcequota tenant-a-quota -n tenant-a` displays detailed information about the specified ResourceQuota including the hard limits configured (pods, CPU, memory) and current usage against those limits, confirming that resource constraints are active and being enforced.

### Evidence

![Verification Commands Output](Verification%20Command.png)

The screenshot shows the successful execution of both verification commands. The NetworkPolicy listing confirms that "default-deny-ingress" is active in tenant-b namespace with age 62 minutes. The ResourceQuota description shows that tenant-a-quota is configured with hard limits of 5 pods, 1 CPU, and 512Mi memory, with current usage of 1 pod and 0 CPU/memory requests.

### Notes

The verification commands successfully confirmed that all security controls are properly implemented and visible in the cluster configuration. The NetworkPolicy resource exists in tenant-b and is being enforced by the Calico CNI plugin, as proven by the timeout behavior observed in Task 4. The ResourceQuota resource exists in tenant-a and is actively tracking resource consumption, ready to enforce limits when pods with resource requests are deployed. In production environments, these verification commands would be part of security audit procedures and compliance documentation, providing evidence that required security controls are in place. Additionally, cluster operators would monitor ResourceQuota usage over time to identify tenants approaching their limits and adjust quotas as needed for capacity planning.

---

## Multi-Tenancy Isolation Concepts Overview

This table summarizes the core concepts and mechanisms used to achieve secure multi-tenancy isolation in Kubernetes:

| Isolation Dimension | Kubernetes Mechanism | Purpose | How It Works |
|---------------------|---------------------|---------|--------------|
| **Logical Separation** | Namespace | Organize and separate resources between tenants, teams, or environments | Creates a virtual cluster boundary within which resources are scoped and policies can be applied |
| **Network Isolation** | NetworkPolicy | Control which pods can communicate with each other | Defines firewall-like rules enforced by CNI plugins (like Calico) that filter network traffic based on pod selectors, namespaces, and IP ranges |
| **Compute Isolation** | ResourceQuota | Prevent one tenant from consuming excessive CPU, memory, or pod count | Enforces hard limits on aggregate resource requests and usage within a namespace, preventing resource exhaustion |
| **Storage Isolation** | RBAC (Roles, RoleBindings) | Control access to Kubernetes resources like Secrets, ConfigMaps, and PersistentVolumes | Defines who (service accounts, users) can perform which actions (get, list, create, delete) on which resources within which namespaces |
| **Data Security** | Cryptographic Erasure | Securely delete sensitive data in cloud environments | Encrypts all data at rest and destroys encryption keys when deletion is needed, making data cryptographically unrecoverable |

---

## Short-Answer Questions

**Q1. Why can containers in different namespaces reach each other by default, and why is that dangerous in multi-tenant cloud?**

**Answer:** Kubernetes implements a flat network model by default where all pods can communicate with all other pods across all namespaces without restrictions. This design choice prioritizes ease of development and service discovery, allowing developers to connect microservices without complex network configuration. However, this is dangerous in multi-tenant environments because namespace boundaries provide only logical organization, not security isolation. If tenant-a's application is compromised through a vulnerability, an attacker could use that foothold to scan for and access services in tenant-b, potentially exfiltrating sensitive data, interfering with operations, or pivoting to other attack targets. In this lab, we proved this risk by showing HTTP 200 success when tenant-a probed tenant-b's web service.
This violates the principle of least privilege and creates compliance issues for regulations like PCI-DSS, HIPAA, and GDPR that require strict data isolation between customers. Network isolation is NOT automatic—it must be explicitly configured through NetworkPolicy resources and requires a CNI plugin that enforces those policies.

---

**Q2. Explain the default-deny principle and how your NetworkPolicy implements it.**

**Answer:** The default-deny principle (also known as whitelisting or "deny by default, permit by exception") is a security architecture pattern where all actions are blocked unless explicitly allowed by policy rules. This is fundamentally more secure than default-allow (blacklisting) because it prevents unauthorized access through misconfiguration, overlooked rules, or unknown attack vectors. The NetworkPolicy we implemented demonstrates default-deny through its structure:

```yaml
spec:
  podSelector: {}        # Applies to ALL pods in tenant-b
  policyTypes: [Ingress] # Controls incoming traffic
  # No ingress rules defined = deny all ingress
```

By specifying `policyTypes: [Ingress]` but providing no `ingress:` rules section, this policy creates a deny-all-ingress posture. The empty `podSelector: {}` means this applies to every pod in the namespace, not just specific pods. Once this policy exists, ALL incoming traffic to tenant-b pods is blocked by default. To allow specific traffic (for example, from an ingress controller or monitoring system), additional NetworkPolicy resources with explicit `ingress:` rules would need to be created, implementing the "permit by exception" part. This was proven when the cross-tenant probe that returned HTTP 200 in Session A timed out after policy application in Session B, demonstrating effective enforcement.

---

**Q3. How do virtual machines and containers differ in isolation strength? When would you add a VM boundary?**

**Answer:** Virtual machines and containers provide different levels of isolation due to their architectural differences:

**Virtual Machines (Strong Isolation):**
- Each VM runs its own complete operating system with its own kernel
- Isolation is enforced at the hypervisor level (hardware virtualization)
- Compromising one VM requires escaping the hypervisor, which is extremely difficult
- Kernel vulnerabilities affect only the individual VM, not other VMs
- Higher resource overhead (each VM needs full OS resources)

**Containers (Weaker Isolation):**
- All containers share the host kernel through namespaces and cgroups
- Isolation is enforced at the OS level (Linux kernel features)
- A kernel vulnerability or container escape affects all containers on that host
- Much lower resource overhead and faster startup times

**When to Add VM Boundaries:**

1. **High-security multi-tenancy:** When hosting untrusted or competing customers who must have strong isolation guarantees (e.g., public cloud providers like AWS running different customers' workloads).

2. **Compliance requirements:** Regulations like PCI-DSS or HIPAA that mandate hardware-level isolation for sensitive data processing.

3. **Different trust levels:** When workloads have vastly different security postures (e.g., public-facing applications vs. internal payment processing).

4. **Defense in depth:** For critical infrastructure, use VMs to isolate groups of related containers, then use containers within each VM for microservices—this provides layered security where an attacker must breach both VM and container boundaries.

In this lab, we used only container/namespace isolation for simplicity, but in production SaaS environments, many providers use VM-per-tenant or VM-per-tier architectures for stronger isolation, then use containers within those VMs for application deployment.

---

**Q4. What is data remanence, and why is cryptographic erasure the preferred cloud solution?**

**Answer:** Data remanence is the residual representation of data that remains on storage media even after deletion operations appear complete. When you delete a file using standard commands (rm, del), the operating system typically only removes the file system metadata (the pointer to where the data is stored), marking that space as available for reuse. The actual data bytes remain physically present on the storage device until they are overwritten by new data, which means deleted sensitive information can potentially be recovered using data recovery tools, forensic software, or direct disk analysis.

**Why Cryptographic Erasure is Preferred in Cloud:**

1. **No physical access:** Cloud tenants don't control physical storage devices and cannot perform secure wipes (multi-pass overwriting) or physical destruction of drives.

2. **Virtualized storage:** Cloud storage is abstracted, distributed, and replicated across many physical devices, making it impossible to know where all copies of data exist or ensure all copies are overwritten.

3. **SSD challenges:** Modern SSDs use wear-leveling that moves data blocks around transparently, making it impossible to guarantee that specific data blocks are overwritten.

4. **Instant deletion:** Cryptographic erasure is immediate—simply destroy the encryption key, and the encrypted data becomes cryptographically unrecoverable without needing time-consuming overwrite operations.

5. **Compliance:** Meets regulatory requirements (GDPR "right to be forgotten," HIPAA, PCI-DSS) for data deletion certificates without physical media destruction.

**How it works:** All data at rest is encrypted with strong encryption (e.g., AES-256), keys are stored in separate key management systems (AWS KMS, Azure Key Vault), and when data must be securely deleted, only the encryption key is destroyed. Without the key, the encrypted data is cryptographically worthless and cannot be recovered even if the ciphertext is physically retrieved. This approach will be implemented in Lab 3 when we explore encryption and key management.

---

**Q5. Which of the three isolation dimensions (compute, network, storage) did each task exercise?**

**Answer:**

| Task | Isolation Dimension(s) | Specific Mechanisms | What Was Demonstrated |
|------|----------------------|---------------------|----------------------|
| **Task 1** | Compute | Namespaces | Created logical separation between tenant-a and tenant-b using Kubernetes namespaces, establishing the foundation for multi-tenancy |
| **Task 2** | Network (observing lack of) | None (showing default-open) | Proved that network isolation does NOT exist by default—tenant-a successfully accessed tenant-b with HTTP 200 response |
| **Task 3** | Compute | ResourceQuota | Applied CPU, memory, and pod count limits to tenant-a, preventing resource exhaustion and "noisy neighbor" problems |
| **Task 4** | Network | NetworkPolicy | Implemented default-deny ingress policy for tenant-b, blocking cross-tenant traffic (proven by timeout vs. previous HTTP 200) |
| **Task 5** | Storage | RBAC (ServiceAccount, Role, RoleBinding) | Enforced secret access control where tenant-a service account can access tenant-a secrets but not tenant-b secrets |
| **Task 6** | Storage | Data sanitization / encryption | Demonstrated data remanence risk and secure deletion techniques; introduced cryptographic erasure as cloud solution |

**Summary of Isolation Dimensions:**
- **Compute Isolation:** Tasks 1 and 3 (namespaces for organization, ResourceQuota for resource limits)
- **Network Isolation:** Tasks 2 and 4 (demonstrating the problem, then solving it with NetworkPolicy)
- **Storage Isolation:** Tasks 5 and 6 (RBAC for access control, secure deletion for data sanitization)

All three dimensions are necessary for comprehensive multi-tenancy security. Missing any dimension creates attack vectors: without network isolation, tenants can access each other's services; without compute isolation, one tenant can exhaust shared resources; without storage isolation, tenants can read each other's sensitive data. Effective cloud security requires defense-in-depth across all layers.

---

## Security Best-Practices Checklist

The following security best practices were implemented and verified throughout this lab:

- [✓] **Tenants are separated into distinct namespaces** — tenant-a and tenant-b created with logical boundaries for resource organization and policy scoping.

- [✓] **A default-deny NetworkPolicy blocks cross-tenant traffic (verified before/after)** — Before: HTTP 200 success in Task 2; After: timeout/failure in Task 4, proving network segmentation enforcement.

- [✓] **Resource quotas prevent a noisy-neighbour from exhausting shared capacity** — tenant-a-quota limits set to 1 CPU, 512Mi memory, and 5 pods maximum.

- [✓] **Per-tenant secrets are unreadable by other tenants (RBAC enforced)** — Service account app-a can get secrets in tenant-a (yes) but not in tenant-b (no).

- [✓] **Secure deletion / cryptographic erasure is understood for data remanence** — Demonstrated data remanence risk, showed secure wipe technique, and understood crypto-erasure as cloud solution.

- [✓] **CNI plugin supports NetworkPolicy enforcement** — Calico CNI installed and verified, enabling actual policy enforcement (default kind network would silently ignore policies).

- [✓] **Before-and-after testing methodology used** — Documented baseline security risk before controls, then re-tested after controls to prove effectiveness.

- [✓] **Principle of least privilege applied** — Limited permissions granted: read-only role for secrets, namespace-scoped access, resource quotas preventing unlimited consumption.

- [✓] **Defense-in-depth implemented** — Multiple layers of security controls across compute (quotas), network (policies), and storage (RBAC) dimensions.

---

## Conclusion

This lab successfully demonstrated comprehensive multi-tenancy isolation controls across three critical security dimensions: compute, network, and storage. The lab provided hands-on experience with the reality that shared infrastructure security is not automatic—it must be explicitly configured and enforced through multiple layers of defense.

### Key Findings:

**1. Default-Open is Dangerous:** The most striking finding was the demonstration in Task 2 that Kubernetes networking is completely open by default, allowing tenant-a to freely access tenant-b's services. This default-open behavior is convenient for development but creates severe security risks in production, enabling lateral movement attacks, data breaches, and compliance violations. The principle of "isolation is NOT automatic—you must configure it" was proven through clear before-and-after evidence (HTTP 200 → timeout).

**2. Defense-in-Depth is Essential:** No single security control is sufficient for multi-tenancy. Network isolation prevents cross-tenant communication but doesn't stop resource exhaustion. ResourceQuotas prevent compute abuse but don't protect data confidentiality. RBAC secures storage access but doesn't segment network traffic. Effective security requires layered controls across all dimensions, as demonstrated by implementing NetworkPolicy + ResourceQuota + RBAC together.

**3. Cloud-Native Solutions Differ from Traditional Approaches:** Traditional security techniques like physical secure wipes don't translate well to cloud environments where storage is virtualized, distributed, and abstracted. Cloud-native approaches like cryptographic erasure provide equivalent or better security properties while being practical in distributed systems, highlighting the importance of adapting security practices to cloud architectures.


### Skills Acquired:

Through this two-session lab, practical skills were developed in:

- **Kubernetes Security Architecture:** Understanding how namespaces, NetworkPolicy, and RBAC work together to enforce multi-tenancy isolation in container orchestration platforms.

- **CNI Configuration:** Installing and configuring Calico CNI to enable NetworkPolicy enforcement, recognizing that security controls require proper infrastructure layer support.

- **Policy-as-Code:** Implementing security policies through declarative YAML configurations that can be version-controlled, audited, and consistently applied across environments.

- **Before-and-After Testing:** Using systematic testing methodology to prove that security controls work as intended, providing auditable evidence of effectiveness.

- **Defense-in-Depth Thinking:** Approaching security as layers of complementary controls rather than relying on any single mechanism, understanding that each layer addresses different attack vectors.

### Real-World Applications:

The techniques learned in this lab are directly applicable to:

- **SaaS Platform Security:** Companies hosting multiple customers on shared Kubernetes infrastructure need exactly these isolation controls to prevent cross-customer data breaches and ensure compliance with data protection regulations.

- **Internal Multi-Team Clusters:** Enterprise organizations using shared Kubernetes clusters for multiple development teams need namespace isolation, resource quotas, and RBAC to prevent teams from interfering with each other and to enforce resource allocation policies.

- **Compliance Requirements:** Regulations like GDPR (data isolation), HIPAA (PHI protection), PCI-DSS (cardholder data segmentation), and SOC 2 (logical access controls) all require the types of isolation controls demonstrated in this lab.

- **Zero-Trust Architecture:** The default-deny NetworkPolicy approach aligns with zero-trust security principles where trust is never assumed and all communication must be explicitly authorized.

### Production Considerations:

While this lab used a local kind cluster for safe experimentation, production implementations would require additional considerations:

- **Policy Management at Scale:** As the number of tenants and namespaces grows, managing individual NetworkPolicy and RBAC resources becomes complex. Tools like Kyverno, OPA Gatekeeper, or Hierarchical Namespace Controller can automate policy application and enforce policy compliance.

- **Monitoring and Alerting:** Network policy violations, resource quota exhaustion, and RBAC denials should be monitored and alerted through observability tools like Prometheus, Grafana, and centralized logging systems to detect security incidents and capacity issues.

- **Admission Control:** PodSecurityPolicies (deprecated) or Pod Security Standards should be enforced to prevent tenants from running privileged containers, accessing host namespaces, or escalating privileges, providing additional compute isolation beyond ResourceQuotas.

- **Network Encryption:** NetworkPolicy controls which traffic is allowed but doesn't encrypt that traffic. Production systems should use service mesh technologies (Istio, Linkerd) or encryption plugins to ensure confidentiality of data in transit between pods.

- **Audit Logging:** Kubernetes audit logs should be enabled and analyzed to track all API operations, providing forensic capability to investigate security incidents and prove compliance during audits.

### Future Labs:

This lab establishes the foundation for subsequent security topics:

- **Lab 3 (Encryption & Key Management):** Will implement the cryptographic erasure concept introduced in Task 6, demonstrating encryption at rest, key management, and secure key deletion.

- **Advanced NetworkPolicy:** Future exploration could include egress controls (limiting outbound traffic), allowing specific cross-namespace communication for legitimate integration, and implementing network segmentation within namespaces.

- **Pod Security:** Additional labs could explore Pod Security Admission, seccomp profiles, AppArmor/SELinux policies, and runtime security tools like Falco to further strengthen compute isolation.

---

## References

The following resources were referenced during this lab:

1. **Course lecture** — Week 3 (Secure Isolation of Physical & Logical Infrastructure), Prof. Dr. Shahrulniza Musa, UniKL MIIT.

2. **Kubernetes Network Policies** — Official Kubernetes documentation: [kubernetes.io/docs/concepts/services-networking/network-policies](https://kubernetes.io/docs/concepts/services-networking/network-policies)

3. **Calico documentation** — Calico CNI and NetworkPolicy implementation: [docs.tigera.io](https://docs.tigera.io)

4. **CSA Security Guidance v5** — Cloud Security Alliance best practices, Infrastructure & Networking domain.

5. **Kubernetes RBAC** — Official documentation on Role-Based Access Control: [kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

6. **IKB42603 Lab Manual** — Lab 2: Secure Isolation & Multi-Tenancy (Weeks 3-4).

7. **Kind Documentation** — Kubernetes IN Docker local cluster creation: [kind.sigs.k8s.io](https://kind.sigs.k8s.io)

8. **NIST SP 800-88 Rev. 1** — Guidelines for Media Sanitization (data remanence and secure deletion standards).

---

## Appendix: Cleanup Commands

To clean up the lab environment and free system resources, execute the following commands:

```bash
# Delete the Kubernetes cluster
kind delete cluster --name ccse-lab2

# Remove Docker volume used for data remanence demonstration
docker volume rm ccse-vol

# Verify cleanup
kind get clusters
docker volume ls
```

**Note:** Deleting the cluster removes all namespaces, deployments, services, NetworkPolicy resources, and RBAC configurations created during this lab. The Docker volume deletion removes any remnant data from the secure deletion demonstration. After cleanup, the system returns to its pre-lab state with no residual resources consuming system resources.

---

## Expansion Ideas (Advanced Students)

For students who want to deepen their understanding of Kubernetes security, the following advanced topics extend the concepts covered in this lab:

### 1. Egress NetworkPolicy

**Objective:** Implement default-deny egress (outbound) policies and selectively allow only necessary external connections.

**Why it matters:** The lab focused on ingress (inbound) isolation, but controlling egress prevents compromised pods from exfiltrating data or communicating with command-and-control servers.

**Implementation:** Create NetworkPolicy with `policyTypes: [Egress]` and explicitly allow only DNS queries and specific external services, blocking all other outbound traffic.

### 2. Pod Security Standards

**Objective:** Enforce Pod Security Admission (restricted profile) to prevent tenants from running privileged containers.

**Why it matters:** Without pod security controls, tenants could escape container boundaries using privileged mode, host namespaces, or host path mounts, bypassing isolation.

**Implementation:** Apply Pod Security labels to namespaces (`pod-security.kubernetes.io/enforce: restricted`) and test that privileged pod creation is blocked.

### 3. Runtime Sandbox with gVisor

**Objective:** Add gVisor as a container runtime to provide additional kernel isolation beyond standard containers.

**Why it matters:** gVisor provides a user-space kernel that intercepts system calls, adding an extra isolation layer between containers and the host kernel, mitigating kernel vulnerabilities.

**Implementation:** Install gVisor runtime class, configure pods to use it, and compare isolation strength vs. standard runc runtime.

### 4. Calico GlobalNetworkPolicy

**Objective:** Implement cluster-wide network policies that apply across all namespaces using Calico CRDs.

**Why it matters:** Kubernetes NetworkPolicy is namespace-scoped, but some security rules should apply cluster-wide (e.g., "no pod may access the cluster metadata service").

**Implementation:** Use Calico's GlobalNetworkPolicy to enforce organization-wide rules that complement namespace-specific policies.


### 5. Service Mesh mTLS

**Objective:** Deploy Istio or Linkerd to automatically encrypt all pod-to-pod traffic with mutual TLS.

**Why it matters:** NetworkPolicy controls which traffic is allowed but doesn't encrypt it. Service mesh provides automatic encryption, authentication, and authorization at the network layer.

**Implementation:** Install Istio, enable sidecar injection, configure mTLS strict mode, and verify that unencrypted traffic is rejected.

### 6. Policy-as-Code with OPA Gatekeeper

**Objective:** Use Open Policy Agent (OPA) Gatekeeper to enforce custom security policies using Rego language.

**Why it matters:** Built-in Kubernetes admission control is limited. OPA enables complex, customizable policies like "all services must have owner labels" or "cross-namespace RoleBindings are prohibited."

**Implementation:** Install Gatekeeper, write ConstraintTemplates and Constraints that enforce multi-tenancy policies, and test that violations are blocked at admission time.

---

## Acknowledgments

This lab was completed as part of the IKB42603 Cloud Computing Security Essentials course at Universiti Kuala Lumpur Malaysian Institute of Information Technology (UniKL MIIT). Special thanks to:

- **Prof. Dr. Shahrulniza Musa** for developing the comprehensive lab curriculum and providing expert guidance on cloud security principles and multi-tenancy isolation techniques.

- **The Calico Project** for providing production-grade NetworkPolicy enforcement capabilities that make secure multi-tenancy practical in Kubernetes environments.

- **The Kubernetes Project** for building the foundational container orchestration platform with security primitives (namespaces, NetworkPolicy, RBAC) that enable secure shared infrastructure.

- **The kind Team** for creating the Kubernetes IN Docker tool that makes local cluster testing accessible and practical for educational purposes.

---

**End of Report**

**Lab Completion Date:** Week 4  
**Status:** All tasks completed successfully with documented evidence  
**Submitted by:** Surya Giri A/L Shanker  
**Course:** IKB42603 Cloud Computing Security Essentials  
**Institution:** UniKL MIIT
