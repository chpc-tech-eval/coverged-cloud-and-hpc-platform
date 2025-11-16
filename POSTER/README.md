# Poster Documentation and Visuals

**Requirements for poster:**
- **A0 Size Poster (841 mm x 1189 mm)**
- Keep your message clear and focused — the main goal is to communicate your project’s story effectively.
- Structure your poster with clear sections: Title, Introduction/Problem, Methods, Results, and Conclusion.
- Use visuals, charts, and figures wherever possible; they tell your story better than long paragraphs.
- Make sure the poster has a logical flow — it should read smoothly and tell a short, engaging story about your project.
- Use large, readable fonts, contrasting colors, and simple layouts.
- You’re welcome to include QR codes that link to a digital copy of your report, your GitHub repository, or anything else you’d like to showcase to the judges and delegates.
- You are also welcome to use any design tool you are comfortable with to create your poster. Examples include Canva, PowerPoint, Adobe Illustrator, Figma, or Google Slides.

#### Poster Headings
1. Title (Members)
2. Introduction (Investigation)
3. CSIR Lab (Hardware & Networking)
4. Deployment (Bare Metal Provisoning & VM Cluster)
5. Benchmarks + Results (Comparision)
6. Challenges Faced
7. Conclusion (Reflections)

## 1. Introduction
Bare Metal Provisioning with Ironic vs Virtualised OpenStack Cluster Performance

This project investigates how converged cloud technologies can be used to support High-Performance Computing (HPC). We explored OpenStack, Kolla Ansible, and Ironic to understand how cloud principles can be applied to HPC environments. Our objective was to deploy and provision a bare-metal cluster using Ironic and compare its performance against a virtualised OpenStack VM-based cluster. The study evaluates scalability, efficiency, and workload suitability across both models.

Expanded:
This project investigates Converged Cloud and High-Performance Computing (HPC) platforms and explores the integration of OpenStack, Kolla Ansible, and Ironic. to support both traditional HPC workloads—such as tightly coupled MPI-based applications—and elastic cloud-style workloads like VMs, containers, and burst environments.

As part of this investigation, we will deploy a bare-metal HPC cluster using OpenStack, Kolla Ansible, and Ironic, provision physical nodes through automated orchestration, and run performance benchmarks. These results will then be compared against an equivalent VM-based cluster to evaluate differences in speed, efficiency, and resource overhead between bare metal and virtualized environments.

## 2. CSIR Lab (Hardware & Networking)
**Hardware** 
-2× bare-metal compute nodes
-1× deployment / controller node
-Gigabit switch, management network, IPMI interfaces

**Network Configuration**
-2x Dell 1U servers in the rack
-Each server has 4 physical network interfaces
-Both servers connect to a single large managed company switch
-No separate lab-isolated switch available
-Provisioning network for Ironic will require its own dedicated interface

## 3. Deploy Ironic with Kolla Ansible
A. Bare Metal Provisioning (Ironic + Bifrost)
This phase involved preparing and provisioning physical compute nodes directly onto hardware using OpenStack Ironic and Bifrost. Unlike traditional virtualization, where virtual machines are created, Ironic allows users to deploy and manage physical servers directly, making it ideal for high-performance workloads that demand full hardware access.

After activating the Bifrost environment, the nodes were enrolled into the Ironic service and validated using baremetal node list. Once the nodes reached the “available” state, Ironic deployed the operating system through automated PXE booting. Successful provisioning was confirmed when each node entered the “active” state and allowed SSH access as cloud-user.

B. Deploy the bare metal cluster using OpenStack
After provisioning the hardware nodes, the next step was integrating them into an OpenStack-managed environment. Using Kolla Ansible, the controller and compute services required for cluster operation were deployed in containerized form.

C. Deploy the cluster on VMs using OpenStack
To create a comparable virtualized cluster, OpenStack was used to deploy multiple compute instances (VMs) using the same base operating system image as the bare-metal cluster.

## 4. Deploy the bare metal cluster using OpenStack
- Checked all configuration changes with mentor documentation before modifying anything in /etc/kolla.

- Provisioning nodes with Bifrost was essential before using Kolla—nodes had to reach the active state.

- Used key validation commands:
   -BareMetal node list
   -ansible-playbook enroll-dynamic.yaml
   -ansible-playbook deploy-dynamic.yaml

- Only proceeded with Kolla-Ansible after successful Bifrost provisioning to avoid interface or firmware-related issues.

## 6. Execute benchmarks on clusters
Benchmarking was performed to evaluate and compare the performance of both the bare-metal and VM-based clusters.

Each test was run under identical software configurations and workload conditions. The results highlighted the performance differences between direct hardware execution and virtualized environments, showcasing the impact of the hypervisor, resource sharing, and I/O virtualization

## 7. Compare the performance to the cluster deployed on bare metal
Chart:


## 8. Discuss the Results


## 9. Challenges Faced
This project gave us real insight into how cloud tools support HPC. By deploying both bare-metal and VM clusters, we saw firsthand how Ironic delivers faster, HPC-focused performance, while VMs offer flexibility for general workloads. 

Through setup, troubleshooting, and benchmarking, we gained practical experience with OpenStack and Kolla Ansible, and a clearer understanding of how modern cloud platforms power high-performance systems.

### EXpanded:
During the deployment of our bare metal HPC cluster using OpenStack and Kolla-Ansible, we encountered several challenges:

1. **Configuration Errors**
   - We made some errors while editing configuration files in `kolla_config`.
   - This caused the server to reboot unexpectedly.
   - Our mentor clarified that the files we were editing are used only for system reset and are not part of the active deployment (`/etc/kolla` is the active configuration).

2. **Interface Misconfiguration**
   - Editing `globals.yml` incorrectly led to interface mismatches.
   - Interfaces in `globals.yml` must not be changed unless necessary.
   - Node-specific interface overrides should be set in the inventory file: `/etc/kolla/inventory/overcloud`.

3. **Bootloader & Firmware Compatibility**
   - The latest bootloader broke compatibility with older UEFI firmware on some nodes.
   - Using the default deployment image avoids this issue unless a `dnf update` is applied.

4. **Node Provisioning Issues**
   - Nodes were not correctly provisioned initially using Kolla.
   - Mentor recommended provisioning nodes with **Bifrost** first before attempting Kolla-Ansible deployment.
   - Needed to create and edit a Bifrost inventory for each node.


#### Mentor Feedback

- Edit configuration files carefully; `/etc/kolla` contains the active deployment.
- Node interfaces should be overridden only in the inventory, not `globals.yml`.
- Bootloader issues may affect older hardware — use default images.
- Bifrost should be used to provision nodes independently before Kolla deployment.
- Upgrade the environment to OpenStack 2024.2 before modifying Kolla configurations.

## 10. Reflections

1. **Lesson Learned**
   - Understanding the distinction between reset configurations and active deployment files is critical.
   - Always check with mentor documentation or guides before modifying configuration files in `/etc/kolla`.

2. **Bifrost Provisioning**
   - Successfully provisioning nodes using Bifrost ensured that nodes reached the `active` state and were ready for Kolla deployment.
   - Commands such as `baremetal node list`, `ansible-playbook enroll-dynamic.yaml`, and `deploy-dynamic.yaml` were essential to validate nodes.

3. **Kolla-Ansible Readiness**
   - Only after nodes were successfully provisioned via Bifrost could we safely start working with Kolla-Ansible.
   - This step prevents errors related to interface misconfigurations or firmware issues.

4. **Upgrade Awareness**
   - Upgrading OpenStack and Kolla-Ansible to the latest stable version (2024.2) before deployment improves compatibility and reduces potential issues.

---

## Summary

- Proper environment setup and following best practices in configuration management are key to successful HPC deployments.
- Bifrost provisioning is a crucial first step before using Kolla-Ansible.
- Careful planning, mentorship guidance, and incremental testing helped overcome deployment challenges.

