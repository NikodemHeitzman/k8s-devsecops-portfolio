# 🛡️ Kubernetes DevSecOps & GitOps Infrastructure

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![Kyverno](https://img.shields.io/badge/Kyverno-000000?style=for-the-badge&logo=kyverno&logoColor=white)
![Trivy](https://img.shields.io/badge/Trivy-0C0438?style=for-the-badge&logo=aquasecurity&logoColor=white)
![Falco](https://img.shields.io/badge/Falco-41C2B0?style=for-the-badge&logo=falco&logoColor=white)

## 📌 Project Overview
This repository serves as a practical demonstration of a modern, secure, and declarative Kubernetes infrastructure. The primary goal of this project is to implement a **Shift-Left Security** approach combined with the **GitOps** methodology. 

Instead of treating security as an afterthought, this environment integrates continuous vulnerability scanning, policy enforcement, and real-time threat detection directly into the cluster's lifecycle.

## 🏗️ Architecture & Core Components

The infrastructure is entirely declarative and managed via **GitOps**. The cluster state is synchronized automatically from this repository, ensuring a Single Source of Truth (SSoT) and immutable infrastructure.

### 1. GitOps Delivery (ArgoCD)
* **Role:** Continuous Delivery & Configuration Management.
* **Implementation:** ArgoCD actively monitors this GitHub repository. Any changes to Kubernetes manifests are automatically detected and reconciled within the cluster, preventing configuration drift and unauthorized manual deployments (`kubectl apply` is strictly avoided in favor of Git commits).

### 2. Policy-as-Code (Kyverno)
* **Role:** Kubernetes Admission Controller.
* **Implementation:** Acts as the cluster's gatekeeper. It evaluates incoming API requests against custom security policies before they are persisted to etcd.
* **Key Policy:** `disallow-root-containers`. Prevents the deployment of any Pods attempting to run as the `root` user, significantly reducing the blast radius of a potential container breakout.

### 3. Supply Chain Security (Trivy Operator)
* **Role:** Continuous Vulnerability Scanner.
* **Implementation:** Automatically scans all container images running in the cluster for known vulnerabilities (CVEs). It generates `VulnerabilityReport` CRDs directly inside the cluster, providing immediate visibility into the security posture of deployed applications (e.g., detecting outdated packages in the deployed OWASP Juice Shop).

### 4. Runtime Threat Detection (Falco via eBPF)
* **Role:** Real-time Intrusion Detection System (IDS).
* **Implementation:** Utilizes **eBPF (Extended Berkeley Packet Filter)** to securely monitor Linux kernel syscalls without impacting performance.
* **Key Capability:** Configured to instantly detect and log anomalous behaviors, such as a malicious actor exploiting a Zero-Day vulnerability to spawn a reverse shell (`/bin/bash`) inside a running web container.

## 🔬 Security Testing Scenarios (Proof of Concept)

This environment was rigorously tested against simulated attacks to validate the DevSecOps pipeline:

1. **Policy Enforcement Test:** Attempting to deploy a non-compliant manifest (running as root) is immediately intercepted and logged/blocked by the Kyverno Mutating/Validating Webhook.
2. **Exploitation & Runtime Detection:** * A vulnerable target (Nginx 1.14) was intentionally deployed.
   * A simulated RCE (Remote Code Execution) attack was executed by attaching an interactive terminal (`kubectl exec -it ... -- /bin/bash`).
   * **Result:** The eBPF-powered Falco engine instantly intercepted the `execve` syscall and triggered a `Notice` level alert: *"A shell was spawned in a container with an attached terminal"*, capturing the precise Pod namespace, container ID, and user UID.

## 🚀 Future Enhancements (Roadmap)
* **Sealed Secrets (Bitnami):** Transitioning from plain-text Base64 Kubernetes Secrets to asymmetric encryption, allowing safe storage of credentials in the public Git repository.
* **CI Pipeline Integration:** Implementing GitHub Actions to run Trivy image scans and Kyverno manifest validation *before* code is merged to the main branch.
* **Network Policies:** Implementing strict default-deny Calico/Cilium network policies to restrict lateral movement between namespaces.

## 🛠️ Tech Stack
`Kubernetes` | `ArgoCD` | `Helm` | `Kyverno` | `Trivy` | `Falco (eBPF)` | `Git` | `Docker`
