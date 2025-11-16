### Setting up the Server at the CSIR Lab - Deployment Reflection & Documentation

During the course of the project timeline, the Technocrats team went to the CSIR HPC lab to physically work on the hardware. We moved the servers into the correct rack positions and recabled the network connections from scratch to ensure a clean, known, controlled wiring topology. We documented each cable connection by creating a network configuration table that maps each server NIC to the specific switch port it connects to. This will help prevent confusion later when configuring bonds, VLANs, provisioning networks, and troubleshooting.

During this session we also reviewed what Ironic actually handles within the OpenStack architecture - specifically bare metal provisioning, life cycle control of physical nodes, and driving PXE boot + image deployment. This leads directly into the most critical open question for tomorrow's planning: network design for Ironic. We currently have the following networks involved in our OpenStack design: API, Tenant, Management, Provisioning, Provider, Storage.

After reviewing Ironic requirements and deployment references, we confirmed that Ironic does need its own dedicated provisioning network. This network should ideally have its own physical interface, separate from the management/API networks. This avoids interference with tenant traffic, prevents DHCP conflicts, and ensures that PXE boot and image deployment are clean and predictable.

#### What the goal of the setup session?

- HPC cluster build
- Network Configuration
- OpenStack/Kolla
- Ironic prep

#### Lab Environment Overview

**Available hardware summary:**

- Switches
- 2x 1U Dell Servers
- PSUs
- Ethernet cables

<p align="center">
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/Nina/WEEK%202%263%20-%20CSIR%20Lab/control.jpg" width="18%" />
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/Nina/WEEK%202%263%20-%20CSIR%20Lab/network%20ports.jpg" width="18%" />
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/Nina/WEEK%202%263%20-%20CSIR%20Lab/psu.jpg" width="18%" />
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/Nina/WEEK%202%263%20-%20CSIR%20Lab/servers.jpg" width="18%" />
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/Nina/WEEK%202%263%20-%20CSIR%20Lab/switch.jpg" width="18%" />
</p>

<p align="center"><b>Figure 1:</b> CSIR Lab Server Hardware, Rack Positioning & Cabling Work</p>

**Network conditions:**

- 2x Dell 1U servers in the rack
- Each server has 4 physical network interfaces
- Both servers connect to a single large managed company switch
- No separate lab-isolated switch available
- Provisioning network for Ironic will require its own dedicated interface

| Server Port | Switch Port |
|-------------|-------------|
| 1           | 17          |
| 2           | 18          |
| 3           | 19          |
| 4           | 20          |
| iDRAC       | 25          |

| Server Port | Switch Port |
|-------------|-------------|
| 1           | 21          |
| 2           | 22          |
| 3           | 23          |
| 4           | 24          |
| iDRAC       | 26          |

**Tasks Completed During Lab Session:**

- Inspected nodes (set up physical servers)
- Checked network requirements for Ironic
- Verified BIOS + firmware settings
- Confirmed boot priority + PXE configs
- Tested connectivity
- Sent public keys to David

Confirmed access to the server:

- We accessed the servers remotely by SSHing through a jumpbox, and established a secure WireGuard tunnel to reach the internal network. This allowed us to remotely manage and interact with the lab servers as if we were directly connected on-site.

<p align="center">
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/Nina/WEEK%202%263%20-%20CSIR%20Lab/btop.jpg" width="50%"/>
</p>

<p align="center"><b>Figure 2:</b> Btop Running on the server</p>
