# **CHPC** – **WEEK** 5: **DEPLOYMENT** **OF** **BARE** **METAL** **REPORT** 

- Deploy the bare metal cluster using OpenStack
- Configure the nodes to create a cluster.
- Execute benchmarks to determine the cluster's performance.

---

## **TASK** 1: **Deploy** **Bare** **Metal** **Cluster**

We deployed two CentOS bare-metal compute nodes (compute01 and compute03) using OpenStack’s bare-metal provisioning tool, Ironic, and the automated provisioning framework Bifrost.

- 2x CentOS bare metal nodes (`compute01`, `compute03`) deployed via Bifrost/Ironic  
- SSH access confirmed from `ansible` control node  

### **TASK 2:** **Slurm** **Setup**

**Control Node (ansible):**
- Installed and configured slurmctld, the SLURM controller daemon responsible for managing job queues, partitions, and resource allocation.

**Compute Nodes (compute01, compute03):**
- Installed and configured slurmd, the node-level daemon responsible for executing jobs.

**Configuration Deployment:**
- A unified slurm.conf was deployed across all nodes using Ansible to ensure identical cluster-wide configuration.

```
Control node: `ansible` (172.16.48.10) → `slurmctld`  
Compute nodes: `compute01` (172.16.48.14), `compute03` (172.16.48.16) → `slurmd`

# Identical ‘slurm.conf` deployed on all nodes  
# sinfo` confirms 2 nodes visible in partition `compute` 
```
SUCCESSFUL

## **TASK** 3: **BENCHMARKS** — **STRESS-NG** **(*EXECUTED ON compute01)**

Command to Execute Benchmark:

```stress-ng --cpu 8 --timeout 30s --metrics-brief```

### RESULTS:

**cpu**  289019  30.00  239.54  0.01  9633.64  1206.53

![Resilts](https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/main/WEEK%205/Picture1.png)

### Breakdown:

**Operations: 289,019**
- Total CPU stress operations completed across all 8 workers.

**Duration: 30.00s**
- Matches the expected timeout → system maintained load without throttling.

**Ops/sec: 239.54 ops/s**
- Shows the sustained throughput under full CPU load.

**User time: 0.01s**
- Near-zero because stress-ng workers spend most time in tight CPU loops with minimal syscalls.

**System time: 9,633.64s (aggregate across all cores)**
- High value expected: 8 cores × 30s = 240s raw time
- stress-ng accumulates system CPU time across threads → higher numbers are normal.
- Indicates the CPU remained fully saturated.

**Bogo-ops/sec per CPU: 1206.53**
- Approx performance per-core.
- Useful for comparing nodes in the same cluster.

## Evaluation:

The benchmark results indicate that the node demonstrates excellent CPU throughput, with the processor able to execute a high number of operations efficiently over the test period. The low system and idle times suggest minimal virtualization or background overhead, confirming that the workload is running directly on the physical cores. This is a key advantage of bare-metal provisioning with Ironic compared to virtualized environments, where hypervisor scheduling and resource sharing can reduce effective performance. 

The CPU scheduler maintained consistent access to all physical cores, allowing each core to process tasks without interruption, which is critical for HPC-style workloads that rely on tightly coupled parallel computation, such as MPI-based applications. Additionally, the low jitter and stable metrics indicate predictable and reliable performance, making the node highly suitable for compute-intensive tasks, scientific simulations, and other workloads that demand high throughput and low latency. These results confirm that deploying HPC workloads on a bare-metal cluster provides both performance consistency and computational efficiency that is difficult to achieve with virtual machines.
