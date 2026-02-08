# KernelWatch 🐝🛡️

![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-blue)
![eBPF](https://img.shields.io/badge/Tech-eBPF-orange)
![Cilium](https://img.shields.io/badge/CNI-Cilium-green)
![Falco](https://img.shields.io/badge/Security-Falco-red)

**KernelWatch** is a next-generation security and observability platform for Kubernetes, built entirely on **eBPF (Extended Berkeley Packet Filter)** technologies.

By bypassing traditional `iptables` and sidecar proxies, KernelWatch achieves high-performance networking and deep visibility into system calls, offering a robust defense against runtime threats.

## 🧠 Architecture

The cluster is architected to run in a "kube-proxy-free" mode, delegating all networking and security logic to the Linux Kernel via eBPF.



* **Networking (Cilium):** Replaces `kube-proxy` for service load balancing and implements Layer 7 (HTTP) filtering.
* **Observability (Hubble):** Provides a visual map of network flows and dropped packets.
* **Runtime Security (Falco):** Taps into the kernel stream to detect suspicious syscalls (e.g., shell execution, sensitive file access).

## 🚀 Key Features

### 1. True eBPF Datapath (No Kube-Proxy)
Standard Kubernetes relies on `iptables` for service discovery, which can become a bottleneck. KernelWatch removes `kube-proxy` entirely (`kubeProxyReplacement=strict`), handling all packet routing in the kernel.

### 2. Runtime Threat Detection
Using the Falco eBPF probe, the system monitors syscalls in real-time.
* **Scenario:** An attacker exploits a vulnerability to open a terminal inside a container.
* **Response:** Falco detects the `execve` syscall for `/bin/bash` and triggers a critical alert instantly.

### 3. Layer 7 Network Policies
We go beyond simple IP/Port blocking.
* **Policy:** Allow `GET /public` but Block `POST /admin`.
* **Mechanism:** Cilium parses the HTTP headers in the kernel and drops unauthorized requests before they reach the application.

## 🛠️ Prerequisites

* **Docker**
* **Kind** (Kubernetes in Docker)
* **Helm** (Package Manager)
* **Cilium CLI**

## ⚙️ Installation & Setup

### 1. Spin up the Cluster (Advanced Config)
We must explicitly disable the default CNI and kube-proxy to allow Cilium to take over.

```bash
cat <<EOF | kind create cluster --name kernelwatch --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true   # We will use Cilium
  kubeProxyMode: "none"     # eBPF will handle this
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF
