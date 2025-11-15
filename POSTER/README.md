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

## 2. CSIR Lab (Hardware & Networking)

## 3. Deploy Ironic with Kolla Ansible

## 4. Deploy the bare metal cluster using OpenStack

## 5. Deploy the cluster on VMs using OpenStack

## 6. Execute benchmarks on clusters

## 7. Compare the performance to the cluster deployed on bare metal

## 8. Discuss the Results

## 9. Challenges Faced

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

