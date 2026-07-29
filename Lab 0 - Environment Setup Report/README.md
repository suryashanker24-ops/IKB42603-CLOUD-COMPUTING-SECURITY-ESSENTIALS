# Lab 0: Environment Setup Report

## Course Information

| Item | Details |
|------|---------|
| Course | IKB42603 Cloud Computing Security Essentials |
| Lab | Lab 0 – Environment Setup |
| Name | SURYA GIRI A/L SHANKER|
| Student ID | 52215124322 |
| Date | 29 July 2026 |

# Objective

The main objective of Lab 0 is to prepare the local environment required for the Cloud Computing Security Essentials laboratory sessions. This setup ensures that all required software and tools are installed, configured, and verified before proceeding to subsequent labs.

The environment includes Docker, AWS CLI v2, kind, kubectl, OpenSSL, oathtool, LocalStack, and a local Kubernetes cluster. All services are configured to run locally without requiring access to a real AWS cloud environment.

---

# Learning Outcomes

After completing this lab, the following learning outcomes were achieved:

- Install and verify Docker.
- Install and configure AWS CLI v2.
- Install and verify kind and kubectl.
- Install helper tools including OpenSSL and oathtool.
- Deploy and verify LocalStack.
- Create and manage a local Kubernetes cluster using kind.
- Configure AWS CLI to communicate with LocalStack.
- Verify that the complete lab environment is ready for future practical exercises.

---

# Environment Summary

| Component | Version / Status | Evidence |
|-----------|-----------------|----------|
| Docker | *Docker version 28.5.2* | `EVIDENCE/01-docker-version.png` |
| AWS CLI | *aws-cli/2.34.56* | `EVIDENCE/03-aws-version.png` |
| kind | *kind version 0.31.0* | `EVIDENCE/04-kind-version.png` |
| kubectl | *Client version: v1.33.4, Kustomize version: v5.5.0* | `EVIDENCE/05-kubectl-version.png` |
| OpenSSL | *OpenSSL 3.5.5* | `EVIDENCE/06-openssl-version.png` |
| oathtool | *oathtool (OATH Toolkit) 2.6.14* | `EVIDENCE/07-oathtool-version.png` |
| LocalStack | Running and Healthy | `EVIDENCE/08-localstack-health.png` |
| Kubernetes Cluster | Running | `EVIDENCE/09-kind-cluster.png` |
| AWS CLI Endpoint | Configured | `EVIDENCE/10-aws-config.png` |

---


# Step 1 – Install and Verify Docker

Docker is the primary platform used throughout the laboratory to run containers. It is required for LocalStack and for hosting the local Kubernetes cluster created by kind.

## Commands Used

```bash
docker --version
docker run --rm hello-world
```

## Explanation

The first command verifies that Docker is installed correctly by displaying its version. The second command downloads and executes the **hello-world** container to confirm that Docker can successfully run containers.

## Evidence

![Docker Version](EVIDENCE/01-docker-version.png)

**Figure 1.** Docker installation verification.

![Hello World](EVIDENCE/02-hello-world.png)

**Figure 2.** Successful execution of the Hello World container.

---

# Step 2 – Install and Verify AWS CLI v2

AWS CLI v2 enables AWS-style commands to be executed against LocalStack. Since LocalStack emulates AWS services locally, no real AWS account is required.

## Commands Used

```bash
aws --version
```

## Explanation

The AWS CLI installation was verified by displaying the installed version.

## Evidence

![AWS CLI](EVIDENCE/03-aws-version.png)

**Figure 3.** AWS CLI version verification.

---

# Step 3 – Install and Verify kind and kubectl

The **kind** utility is used to create a local Kubernetes cluster using Docker containers, while **kubectl** is used to interact with the cluster.

## Commands Used

```bash
kind --version
kubectl version --client
```

## Explanation

Both commands were executed to ensure that Kubernetes management tools were installed successfully.

## Evidence

![kind](EVIDENCE/04-kind-version.png)

**Figure 4.** kind version verification.

![kubectl](EVIDENCE/05-kubectl-version.png)

**Figure 5.** kubectl client verification.

---

# Step 4 – Install and Verify Helper Tools

OpenSSL is required for encryption and certificate-related exercises, while oathtool is used to generate One-Time Passwords (OTP) during authentication-related labs.

## Commands Used

```bash
openssl version
oathtool --version
```

## Evidence

![OpenSSL](EVIDENCE/06-openssl-version.png)

**Figure 6.** OpenSSL verification.

![oathtool](EVIDENCE/07-oathtool-version.png)

**Figure 7.** oathtool verification.

---

# Step 5 – Start and Verify LocalStack

LocalStack provides a local implementation of AWS services required throughout the laboratory exercises.

## Commands Used

```bash
docker run -d --name localstack -p 4566:4566 localstack/localstack:3.0

curl http://localhost:4566/_localstack/health

docker ps
```

## Explanation

LocalStack was started using the Docker image, followed by a health check to verify that the services were available. The running container was confirmed using `docker ps`.

## Evidence

![LocalStack](EVIDENCE/08-localstack-health.png)

**Figure 8.** LocalStack health verification.

---

# Step 6 – Create and Verify Kubernetes Cluster

A local Kubernetes cluster named **ccse** was created using kind.

## Commands Used

```bash
kind create cluster --name ccse

kubectl cluster-info --context kind-ccse

kubectl get nodes
```

## Explanation

The Kubernetes cluster was created and verified to ensure that the control plane and worker node were functioning correctly.

## Evidence

![Kubernetes](EVIDENCE/09-kind-cluster.png)

**Figure 9.** Kubernetes cluster verification.

---

# Step 7 – Configure AWS CLI for LocalStack

Dummy AWS credentials were configured because LocalStack accepts any access key and secret key for local testing.

## Commands Used

```bash
aws configure set aws_access_key_id test

aws configure set aws_secret_access_key test

aws configure set region us-east-1

EP='--endpoint-url=http://localhost:4566'

aws $EP sts get-caller-identity
```

## Explanation

The AWS CLI was configured to communicate with LocalStack instead of real AWS services. The identity returned by LocalStack confirmed that the configuration was successful.

## Evidence

![AWS Config](EVIDENCE/10-aws-config.png)

**Figure 10.** AWS CLI configuration and LocalStack endpoint verification.

---

# Challenges Encountered

During the LocalStack setup, the latest Docker image (`2026.x`) required a LocalStack authentication token and exited with a licence activation error. This prevented the health endpoint from responding successfully.

The issue was resolved by removing the latest Docker image and using the Community-compatible image (`localstack/localstack:3.8`), which matches the version expected by the lab guide. After switching to the pinned image, LocalStack started successfully and the health endpoint returned a healthy status.

---

# Lessons Learned

This lab provided practical experience in preparing a cloud computing laboratory environment. It demonstrated the importance of verifying software installations before use and highlighted how Docker containers can be used to host local cloud services such as LocalStack and Kubernetes.

In addition, troubleshooting the LocalStack version issue reinforced the importance of using compatible software versions and understanding container logs when diagnosing deployment problems.

---

# References

1. IKB42603 Cloud Computing Security Essentials – Lab 0 Setup Cheatsheet.
2. Docker Documentation. https://docs.docker.com/
3. AWS CLI Documentation. https://docs.aws.amazon.com/cli/
4. kind Documentation. https://kind.sigs.k8s.io/
5. Kubernetes Documentation. https://kubernetes.io/docs/
6. LocalStack Documentation. https://docs.localstack.cloud/

---

# Conclusion

Lab 0 was completed successfully. All required software tools were installed, configured, and verified. Docker, AWS CLI v2, kind, kubectl, OpenSSL, oathtool, LocalStack, and the local Kubernetes cluster were operational. The environment is fully prepared for the subsequent Cloud Computing Security Essentials laboratory exercises.