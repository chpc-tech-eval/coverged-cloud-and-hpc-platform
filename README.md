<h1>
  Converged Cloud & HPC Platform
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/main/Technocrats%20Logo.jpg" alt="Logo" width="200" align="right"/>
</h1>  

In this project our team has been tasked to investigate **Converged Cloud and High-Performance Computing (HPC) Platforms**.  

The goal is to explore how cloud computing technologies can be utilized in an HPC platform to deliver scalable, efficient, and flexible computational infrastructure for research, analytics, and enterprise applications.  

We will investigate how modern cloud technologies—particularly **OpenStack**, **Kolla Ansible**, and **Ironic**—can be integrated to support both traditional HPC workloads (such as tightly coupled MPI-based parallel programs) and elastic cloud-style workloads (such as virtual machines, containers, and burst capacity).

## ☁️ Cloud Computing  
<p align="center">
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/main/cloud-computing-icon-for-your-website-mobile-presentation-and-logo-design-free-vector-3978331410.jpg" alt="cloud computing" width="100"/>
</p>

Cloud computing has become a major development in the world; it has fundamentally changed how businesses and individuals store, manage, and access data.  

This technology allows organizations and users to access computing services on demand through the internet without owning or maintaining physical hardware. Instead of investing in and maintaining physical hardware or data centers, organizations and users can rent resources from cloud providers like **Amazon Web Services (AWS)**, **Microsoft Azure**, or **Google Cloud Platform (GCP)**.  

This model transforms how computing power is delivered and consumed, enabling businesses to scale rapidly, reduce costs, and innovate faster.  

In research and High-Performance Computing (HPC), the cloud provides a flexible environment where computational resources can be dynamically provisioned and released as needed, ensuring both cost efficiency and performance.

## What is OpenStack?  

OpenStack serves as the foundation of the converged platform.  

**OpenStack** is a powerful, open-source, modular **IaaS (Infrastructure as a Service)** cloud management platform. Designed to control and manage large pools of compute, storage, and networking resources, it enables users to deploy virtual machines and other instances dynamically to manage and operate various tasks within a cloud environment. It supports **horizontal scaling**, which allows systems that benefit from concurrent processing to easily adjust to changing demand by adding or removing instances as needed.  

For example, a mobile application that interacts with a remote server can distribute user requests across multiple instances. As the number of users grows, new instances can be launched automatically to share the workload—ensuring smooth performance and efficient scaling as demand increases.

<p align="center">
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/main/openstack_saas_cloud_platform_implementation_guide_powerpoint_ppt_template_bundles_cl_mm_slide12-2229342777.jpg" alt="openstack" width="500"/>
</p>

### Core Components and Architecture of OpenStack  

OpenStack consists of numerous integrated components that work together to provide cloud infrastructure services. The OpenStack community has collectively defined nine essential “core” components that form the foundation of any OpenStack deployment.

| Component | Description |
|------------|-------------|
| **Nova** | Decision maker that runs your virtual machines (VMs) or bare-metal servers (through Ironic). |
| **Neutron** | Manages all the networking inside OpenStack—connecting VMs, servers, and external networks. |
| **Cinder** | Provides persistent storage volumes that you can attach to your VMs or servers—like a virtual hard drive. |
| **Swift** | Stores and manages large amounts of unstructured data (like files, images, backups, and datasets). |
| **Glance** | Manages disk images—the templates used to create VMs or bare-metal servers. |
| **Keystone** | Handles authentication and access control for all OpenStack services. |
| **Horizon** | Provides a web-based graphical interface for users and administrators to interact with OpenStack services. |

### The Main Characteristics of OpenStack (in the context of HPC)  

- **Scalability and Elasticity:** OpenStack can dynamically scale computing, storage, and networking resources to match the intensive and varying demands of HPC workloads. This enables efficient resource utilization across clusters of any size.

- **Automation and Deployment:** Through tools like Kolla Ansible, OpenStack automates the deployment, configuration, and scaling of HPC environments—reducing manual setup time and improving consistency.

- **Multi-tenancy and Resource Isolation:** Using Keystone and Nova, OpenStack supports multiple users or projects running workloads simultaneously while ensuring secure separation of data and compute resources.

- **Integration with Bare Metal (Ironic):** OpenStack allows direct provisioning of physical nodes for HPC workloads using Ironic, which offers near-native performance—essential for scientific and computational research tasks.

- **Support for Virtualization and Containerization:** OpenStack supports both virtual machines and container management frameworks (like Kubernetes), allowing flexible workload deployment strategies across HPC infrastructures.

#### OpenStack Components  
<p align="center">
  <img src="https://github.com/chpc-tech-eval/coverged-cloud-and-hpc-platform/blob/main/Screenshot%202025-10-09%20210859.png" alt="openstack" width="500"/>
</p>

## Kolla Ansible  

**Kolla Ansible** is a deployment and management tool specifically designed for OpenStack.  
It uses **Docker containers** to wrap OpenStack services and **Ansible playbooks** to automate configuration, deployment, and upgrades.  

In a national HPC center, OpenStack provides a scalable, flexible, and open-source cloud infrastructure for managing compute, storage, and networking resources, while Kolla Ansible enhances its effectiveness by automating deployment and lifecycle management through containerized services. By using Kolla Ansible, administrators can efficiently deploy the OpenStack control plane across multiple HPC clusters, ensuring consistency, reliability, and simplified maintenance.

### How It Works  

- Each OpenStack component (i.e Nova, Neutron, Cinder, etc.) runs inside a dedicated Docker container, ensuring isolation and reproducibility.
   
- Ansible playbooks handle automation of installation, networking, and lifecycle operations (deploy, upgrade, restart).

- The containerized approach minimizes dependency conflicts, simplifies rollback procedures, and enhances system reliability.

##### **Use Case in HPC Competition Kolla would be used to:** 
- Deploy the OpenStack control plane across multiple clusters.
  
- Manage containerized OpenStack services for predictable performance.  

- Each OpenStack component (like Nova, Neutron, etc.) runs in its own container, so if one breaks, the others keep running.  

- You can scale up quickly (add more nodes) and update without stopping running jobs—ideal for big HPC centers.

## Ironic  

**OpenStack Ironic** is an open-source component of the OpenStack ecosystem designed to enable **bare-metal provisioning**.  

Unlike traditional virtualization, where virtual machines are created, Ironic allows users to deploy and manage **physical servers directly**, making it ideal for high-performance workloads that demand full hardware access.

### Why Do We Need This?  

For many years, Virtual Machines played a critical role in HPC. VMs were the backbone of virtualization; their abstract physical hardware provided flexibility, scalability, and ease of management. But as cloud computing and AI/ML become more prevalent, the limitations of VMs are becoming more apparent. Organizations are faced with increasing system performance by using bare metal provisioning. 

This is due to the fact that VMs add an extra layer between applications and the physical hardware, which can reduce performance for demanding tasks like AI model training, big data analytics, or real-time processing. This additional layer can cause latency, resource contention, and performance bottlenecks. Although VMs are well-suited for many general-purpose workloads, some applications require direct access to the hardware to achieve maximum performance and efficiency.

### What is Bare Metal Provisioning?  

**Bare-metal provisioning** is the process of automatically setting up and configuring physical (real) servers and not virtual machines so they’re ready to run workloads or applications. Simply put, it is turning real hardware into cloud-like resources. This allows for systems to get the raw hardware performance with no virtualization overhead but managed and automated just like cloud VMs.

By integrating bare-metal and virtual infrastructure under one framework, Ironic bridges the gap between cloud flexibility and raw hardware performance. This gives organizations the ability to run both cloud-native and HPC workloads seamlessly, while retaining the control, efficiency, and speed that come with dedicated physical machines.

### Core Capabilities  

- **Hardware Introspection:** Automatically detects and reports hardware capabilities, configurations, and anomalies.
  
- **Zero Virtualization Overhead:** Runs workloads directly on physical hardware, making it ideal for high-performance computing (HPC) environments.
  
- **Multi-tenancy Support:** A single physical or virtual infrastructure is shared among multiple users or projects (called tenants) while keeping their resources, data, and operations isolated from each other.

- **Project-Specific Deployments:** Different projects can run entirely separate software stacks.
   
- **Infrastructure as Code:** Deployments are automated and configured through prescriptive scripts or templates, often version-controlled
  
- **Hardware Diversity:** Supports a wide range of physical hardware.
   
- **Open-source Collaboration:** Backed by fast community support.  

## Implementation: OpenStack, Kolla-Ansible, and Ironic in HPC  

#### **OpenStack**
Provides the cloud management layer for HPC clusters.  

**Key Implementation Points:**
- Manages HPC cluster resources: compute, storage, networking, and user access.  
- Isolates different research groups or projects (**multi-tenancy**).  
- Provides APIs to control virtual machines and bare-metal nodes.

#### **Kolla-Ansible**
Automates OpenStack deployment using containerized services.  

**Key Implementation Points:**
- Installs OpenStack in Docker containers on head nodes.  
- Automates deployment and updates using Ansible playbooks.  
- Makes the deployment repeatable and easy to scale.

#### **Ironic**
Manages bare-metal nodes for HPC workloads requiring maximum performance.  

**Key Implementation Points:**
- Manages bare-metal HPC nodes (real hardware, not virtual).  
- Registers nodes, deploys OS images, and resets nodes after use.  
- Works with Neutron to assign VLANs or VXLANs, keeping project workloads separate.

<details>
  <summary><b>📚 References</b></summary>  
  
- Opensource.com, 2025. *What is OpenStack?* [online] Available at: https://opensource.com/resources/what-is-openstack [Accessed 7 October 2025].  
- GeeksforGeeks, 2025. *Introduction to OpenStack.* [online] Available at: https://www.geeksforgeeks.org/cloud-computing/introduction-to-openstack/ [Accessed 8 October 2025].  
- GeeksforGeeks, 2025. *Cloud Computing.* [online] Available at: https://www.geeksforgeeks.org/cloud-computing/cloud-computing/ [Accessed 9 October 2025].  
- Amazon Web Services, 2025. *What is HPC?* [online] Available at: https://aws.amazon.com/what-is/hpc/ [Accessed 8 October 2025].  
- Ullah, A., 2025. *AI & OpenStack Ironic: Bare Metal Provisioning.* [online] LinkedIn. Available at: https://www.linkedin.com/pulse/ai-openstack-ironic-bare-metal-provisioning-ahmad-ullah-zgnqf/ [Accessed 8 October 2025].  
- StackHPC, 2025. *OpenStack and HPC Infrastructure.* [online] Available at: https://www.stackhpc.com/openstack-and-hpc-infrastructure.html [Accessed 8 October 2025].  
- OpenStack, 2025. *Building the Future on Bare Metal: How Ironic Delivers Abstraction and Automation using Open Source Infrastructure.* [online] Available at: https://www.openstack.org/use-cases/bare-metal/how-ironic-delivers-abstraction-and-automation-using-open-source-infrastructure [Accessed 9 October 2025].  
- OpenStack Foundation, 2025. *Installation Guide.* Available at: https://docs.openstack.org/install-guide/overview.html [Accessed 9 October 2025].  
- Superuser, 2020. How to Implement an OpenStack-Based Private Cloud with Kolla Ansible. OpenInfra Foundation. Available at: https://superuser.openinfra.org/articles/how-to-implement-an-openstack-based-private-cloud-with-kolla-ansible-part-1/ [Accessed 9 Oct 2025].  

</detail>
---


