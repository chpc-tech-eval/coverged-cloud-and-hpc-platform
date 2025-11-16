# Kubernetes Cluster Setup & Benchmarking Documentation

## Project Overview

1. [Prerequisites](#prerequisites)
2. [Step-by-Step Setup](#step-by-step-setup)
3. [Benchmarking Procedures](#benchmarking-procedures)
4. [Results And Evaluation](#results-and-evaluation)

This documentation covers the complete setup of a 2-node Kubernetes cluster on OpenStack VMs and the execution of CPU/RAM benchmarks to measure cluster performance.

### Goals
- Deploy Kubernetes cluster on OpenStack VMs
- Configure nodes to form a working cluster
- Execute benchmarks to determine cluster performance
- Compare performance with bare metal deployments

## Architecture

```
┌─────────────────┐    ┌─────────────────┐
│   vm-compute03  │    │   vm-compute01  │
│  (Control Plane)│    │   (Worker Node) │
│                 │    │                 │
│ • kube-apiserver│    │ • kubelet       │
│ • etcd          │    │ • containerd    │
│ • kube-scheduler│    │ • kube-proxy    │
│ • kube-controller│   └─────────────────┘
└─────────────────┘
          │
┌─────────────────┐
│  admin@ansible  │
│  (Management)   │
│                 │
│ • SSH access    │
│ • kubectl config│
└─────────────────┘
```

## Prerequisites

### System Requirements
- OpenStack VMs with Rocky Linux 9
- SSH access between nodes
- sudo privileges on all nodes
- Minimum 2 vCPUs, 4GB RAM per node

### Network Configuration
- All nodes must be able to communicate on ports 6443, 2379-2380, 10250-10259
- Pod network CIDR: 10.244.0.0/16
- Service network CIDR: 10.96.0.0/12

## Step-by-Step Setup

### Phase 1: Initial Node Configuration

#### 1.1 SSH Key Setup
```bash
# Generate SSH key on control plane
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Copy key to worker nodes
ssh-copy-id rocky@172.16.10.229  # vm-compute01
```

#### 1.2 Install Docker on All Nodes
```bash
# Remove podman-docker conflicts if they exist
sudo dnf remove -y podman-docker

# Install Docker
sudo dnf install -y docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

#### 1.3 Install Kubernetes Tools
```bash
# Add Kubernetes repository
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl
EOF

# Install Kubernetes packages
sudo dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
```

#### 1.4 System Configuration
```bash
# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Configure kernel modules and networking
sudo modprobe br_netfilter
echo 'br_netfilter' | sudo tee -a /etc/modules-load.d/k8s.conf

echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.bridge.bridge-nf-call-iptables = 1' | sudo tee -a /etc/sysctl.d/k8s.conf
echo 'net.bridge.bridge-nf-call-ip6tables = 1' | sudo tee -a /etc/sysctl.d/k8s.conf
sudo sysctl --system
```

#### 1.5 Container Runtime Setup (Containerd)
```bash
# Install containerd
sudo dnf install -y containerd.io

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable systemd cgroups (CRITICAL for Kubernetes)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Start containerd
sudo systemctl enable --now containerd
```

### Phase 2: Control Plane Setup (vm-compute03)

#### 2.1 Initialize Kubernetes Cluster
```bash
# Initialize the control plane
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Set up kubectl for regular user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 2.2 Install Network Plugin (Flannel)
```bash
# Install Flannel CNI
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### 2.3 Verify Control Plane
```bash
# Check cluster status
kubectl get nodes
kubectl get pods --all-namespaces
```

### Phase 3: Worker Node Setup (vm-compute01)

#### 3.1 Join Worker to Cluster
```bash
# On control plane, get join command
kubeadm token create --print-join-command

# On worker node, run the join command with sudo
sudo kubeadm join 172.16.10.244:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

#### 3.2 Verify Node Join
```bash
# Back on control plane, check nodes
kubectl get nodes

# Expected output:
# NAME                     STATUS   ROLES           AGE   VERSION
# vm-compute01.novalocal   Ready    <none>          31s   v1.28.15
# vm-compute03.novalocal   Ready    control-plane   15m   v1.28.15
```

### Phase 4: Cluster Verification

#### 4.1 Complete Cluster Check
```bash
# Check all components
kubectl get componentstatuses
kubectl get pods --all-namespaces
kubectl get nodes -o wide

# Check cluster info
kubectl cluster-info
```

## Benchmarking Procedures

### CPU & Memory Benchmarks

#### Create Benchmark Namespace
```bash
kubectl create namespace benchmarks
```

#### CPU Stress Test
```bash
kubectl apply -n benchmarks -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: cpu-stress
spec:
  template:
    spec:
      containers:
      - name: stress
        image: containerstack/cpustress
        command: ["cpustress"]
        args: ["--cpu", "4", "--timeout", "30s", "--metrics"]
        resources:
          requests:
            memory: "256Mi"
            cpu: "2"
          limits:
            memory: "512Mi"
            cpu: "4"
      restartPolicy: Never
  backoffLimit: 1
EOF
```

#### Memory Benchmark
```bash
kubectl apply -n benchmarks -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: memory-benchmark
spec:
  template:
    spec:
      containers:
      - name: memory-test
        image: ubuntu:22.04
        command: ["/bin/bash"]
        args: 
        - -c
        - |
          apt-get update && apt-get install -y stress-ng && \
          stress-ng --vm 2 --vm-bytes 512M --timeout 30s --metrics-brief
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1"
      restartPolicy: Never
  backoffLimit: 1
EOF
```

#### Monitor Benchmarks
```bash
# Watch progress
watch -n 3 "kubectl get jobs -n benchmarks && echo '' && kubectl get pods -n benchmarks && echo '' && kubectl top nodes"

# Get results after completion (wait ~35 seconds)
kubectl logs -n benchmarks job/cpu-stress
kubectl logs -n benchmarks job/memory-benchmark
kubectl top nodes
```
<p align="center">
<img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/main/WEEK%206/compute01-bm.jpg" alt="Logo" width="50%" /> 
<p align="center"><b>Figure 1:</b> Performance of compute01 of the Bare Metal Cluster with btop (tmux multiplexer) </p>
</p>

<p align="center">
<img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/main/WEEK%206/compute01%20-%20vm.jpg" alt="Logo" width="50%" /> 
<p align="center"><b>Figure 2:</b> Performance of compute02 of the Virtual Machine Cluster with btop (tmux multiplexer) </p>
</p>

#### Cleanup Benchmarks
```bash
kubectl delete namespace benchmarks
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Container Runtime Not Running
```bash
# Check containerd status
sudo systemctl status containerd

# Regenerate config if CRI issues occur
sudo systemctl stop containerd
sudo rm -f /etc/containerd/config.toml
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl start containerd
```

#### 2. Port Already in Use During kubeadm init
```bash
# Reset and clean installation
sudo kubeadm reset -f
sudo rm -rf /etc/kubernetes/ /var/lib/etcd/ ~/.kube

# Kill processes using ports
sudo lsof -ti:6443,10259,10257,10250,2379,2380 | xargs sudo kill -9 2>/dev/null || true

# Reinitialize
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

#### 3. Network Bridge Issues
```bash
# Ensure bridge modules are loaded
sudo modprobe br_netfilter
echo 'br_netfilter' | sudo tee -a /etc/modules-load.d/k8s.conf
sudo sysctl --system
```

#### 4. Node Not Joining Cluster
```bash
# Check kubelet logs on worker node
sudo journalctl -u kubelet -f

# Verify network connectivity between nodes
ping <control-plane-ip>
telnet <control-plane-ip> 6443

# Regenerate join token if expired
kubeadm token create --print-join-command
```

## Useful Commands

### Cluster Management
```bash
# Get cluster information
kubectl cluster-info
kubectl get componentstatuses

# Node management
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl cordon <node-name>    # Mark node as unschedulable
kubectl uncordon <node-name>  # Mark node as schedulable

# Resource monitoring
kubectl top nodes
kubectl top pods -A
```

### Pod and Deployment Management
```bash
# List all resources
kubectl get all -A
kubectl get pods -o wide

# Debug pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/bash

# Delete resources
kubectl delete pod <pod-name>
kubectl delete deployment <deployment-name>
kubectl delete namespace <namespace-name>
```

### Maintenance Commands
```bash
# Drain node for maintenance
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Reset node
sudo kubeadm reset
sudo rm -rf /etc/kubernetes/ /var/lib/etcd/ ~/.kube

# Check system resources
kubectl describe nodes | grep -A 10 "Allocated resources"
```

### Benchmarking Quick Reference
```bash
# One-liner for CPU/Memory benchmarks
kubectl create namespace benchmarks && \
kubectl apply -n benchmarks -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: cpu-stress
spec:
  template:
    spec:
      containers:
      - name: stress
        image: containerstack/cpustress
        command: ["cpustress"]
        args: ["--cpu", "4", "--timeout", "30s", "--metrics"]
        resources:
          requests:
            memory: "256Mi"
            cpu: "2"
          limits:
            memory: "512Mi"
            cpu: "4"
      restartPolicy: Never
  backoffLimit: 1
EOF
```
## Benchmark Results 

<img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/main/WEEK%206/Picture2.png" alt="Logo" /> 

## Results And Evaluation

| Metrics | Baremetal Cluster | VM Cluster |
|---------|-------------------|------------|
| **Total Operations (bogo-opc secs)** | 289,019 | 3,439,530 |
| **Real Time** | 30.00 | 60.06 |
| **User Time** | 239.54 | 46.28 |
| **System Time** | 0.01 | 13.78 |
| **Throughput (bogo ops/s)** | 9,633.64 | 1,206.53 |

### Methodology

To quantify the performance impact of virtualization, we conducted a CPU stress test using `stress-ng` on two environments:

* **Bare-Metal Cluster:** A physical server with direct hardware access.
* **VM Cluster:** A virtualized environment running on shared host infrastructure.

The test measured total operations, throughput (operations per second), and CPU time distribution.

## Detailed Analysis

### Total Operations & Throughput

While the raw total of operations is higher for the VM, this is misleading as the test duration was not the same. The most critical metric is **throughput (ops/sec)**.

**Table 1: Throughput Comparison**

| Metric | Bare Metal | VM Cluster | Difference |
|--------|-------------|------------|------------|
| **Test Duration** | 30.00 sec | 60.06 sec | 2x longer for VM |
| **Total Operations** | 289,019 | 3,439,530 | (Not directly comparable) |
| **Throughput (ops/sec)** | **9,633.64** | **1,206.53** | **Bare Metal is ~8x Faster** |

**Analysis:** The bare-metal node processed computations at a rate nearly eight times greater than the VM cluster. This is a direct result of the VM's virtualization overhead, including vCPU scheduling and resource contention on the physical host.

### CPU Efficiency & Overhead

The distribution of CPU time between user space (productive work) and system space (kernel overhead) is a key indicator of efficiency.

**Table 2: CPU Time Distribution**

| CPU Time | Bare Metal | VM Cluster | Interpretation |
|----------|-------------|------------|----------------|
| **User Time** | 239.54s | 46.28s | Bare metal spends more total CPU time on productive work. |
| **System Time** | 0.01s | 13.78s | VM spends a significant amount of time on kernel/overhead tasks. |
| **System/User Ratio** | **0.004%** | **29.8%** | **VM system overhead is ~7,450x higher.** |

**Analysis:** The bare-metal server dedicates almost 100% of its CPU time to useful computation. In contrast, the VM cluster spends a substantial portion (almost 30% of its user time) on system-level operations, indicating overhead from the hypervisor, virtual hardware emulation, and scheduling delays.

**Table 3: Normalized Per-Core Performance**

| Metric | Bare Metal | VM Cluster | Performance Gap |
|--------|-------------|------------|-----------------|
| **Throughput (ops/sec)** | 9,633.64 | 1,206.53 | - |
| **Assumed Cores** | 8 | 8 | (Assumed equal for comparison) |
| **Per-Core Throughput** | **1,204.2 ops/sec/core** | **150.8 ops/sec/core** | **Bare Metal is ~8x Faster per Core** |

**Analysis:** When normalized per core, the performance disparity remains clear. Each physical core in the bare-metal environment is vastly more effective than a vCPU in the VM cluster.

## Conclusion: 

The benchmark results demonstrate that the **bare-metal node delivers significantly superior performance, with approximately 8 times the throughput** of the VM cluster. The bare-metal environment exhibits high CPU efficiency with minimal overhead, making it the optimal choice for high-performance computing (HPC) and latency-sensitive workloads. The VM cluster, while offering flexibility, introduces substantial performance penalties due to virtualization overhead.

---
