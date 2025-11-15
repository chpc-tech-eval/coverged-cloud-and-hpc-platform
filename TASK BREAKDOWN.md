# CHPC Task Breakdown - Projected Tasks to Complete

## Timeline Overview

| **Date** | **Task** |
|----------|-----------|
| **27 October – 31 October** | **Week 1: Completed**<br><br>**Poster Specifications** to be announced during Week 3 |
| **Preparation Before Lab** | - Allocate roles for each team member (cabling, hardware, networking, OpenStack deployment)<br>- Prepare documentation for hardware configs + IP assignments<br>- Become familiar with:<br>• **OpenStack** (core services, deployment concepts, CLI)<br>• **Kolla Ansible** (automation, configs, service management)<br>• **Ironic** (bare metal provisioning, node registration, testing)<br>- Understand Ironic requirements |
| **31 October** | **Visit CHPC Lab (9:00am)**<br><br>**Week 2 Tasks:**<br>- Inspect hardware<br>- Identify configuration changes<br>- Reconfigure hardware<br><br>**Week 3 Tasks:**<br>- Cable nodes<br>- Update switch configs<br>- Test network |
| **3 November – 9 November** | **Exam period over**<br><br>**Week 4: (Joey & Nic) — Completed**<br>- Deploy Ironic using Kolla Ansible (Nic)<br>- Setup Bifrost (Joey)<br>- Test bare metal provisioning via OpenStack<br>- Test functionality of bare metal nodes<br><br>**Week 5: Everyone**<br>- Deploy bare metal cluster using OpenStack<br>- Configure nodes into cluster<br>- Execute benchmarks<br>- Full documentation + results (Nina)<br><br>**Meeting:** 9pm on 9 November |
| **10 November – 14 November** | **Poster Development (Design, visualization, data collection)**<br><br>**Week 6 Tasks:**<br>- Deploy cluster on VMs using OpenStack (Nicoroy)<br>- Configure cluster nodes (Jazeel)<br>- Execute benchmarks (Joey)<br>- Compare VM vs bare metal performance (Nicoroy)<br><br>**Reflections (Nina):**<br>- What worked well?<br>- What challenges occurred? |
| **16 November** | **Validation and Quality Check**<br>- Peer-review documentation<br>- Verify consistency with benchmark files<br>- Backup all data to GitHub<br><br>**Submission & Presentation Prep:**<br>- Upload final report<br>- Prepare summary (deployment, benchmarks, comparison)<br><br>**Project Deadline:** Sunday, **16 Nov 2025** |
| **23 November** | Final poster update deadline (**23:59**). No further edits allowed afterward. |

---

### To Be Completed in the CHPC Lab

## Hardware Inspection and Configuration

| **Task Category** | **Details** |
|-------------------|-------------|
| **Hardware Inspection** | - Inspect each server and component<br>- Record specs and configurations<br>- Identify configuration changes (BIOS, firmware, storage)<br>- Reconfigure hardware (RAID, BIOS updates, IP settings)<br>- Verify servers power on and are reachable via BMC/management |
| **Network Setup** | - Cable nodes according to network plan<br>- Label all connections<br>- Update switch configs (VLANs, trunking, mgmt IPs)<br>- Test connectivity between nodes<br>- Validate communication with management station |
| **Tips** | - Use ping tests & switch port status checks<br>- Document IP assignments & switch configs |

