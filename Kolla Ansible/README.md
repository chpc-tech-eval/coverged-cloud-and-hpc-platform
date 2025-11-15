# Kolla-Ansible: Complete Deployment Guide  
(Requirements → Deploy → Validate)

## Table of Contents
1. Quick overview  
2. Requirements (hardware, OS, network, software)  
3. Architecture options (all-in-one vs multinode)  
4. Prepare control workstation (where you run Ansible)  
5. Prepare target hosts (bootstrap tasks)  
6. Example inventory (all-in-one and multinode)  
7. /etc/kolla config files — globals.yml, passwords.yml (templates)  
8. Install Kolla-Ansible (pip / from source)  
9. Deploy commands (bootstrap → pull images → deploy → post-check)  
10. Validation and smoke tests  
11. Common adjustments and useful kolla-ansible variables  
12. Backup, upgrade, and operational notes  
13. Troubleshooting checklist & common errors  
14. References  

---

## 1) Quick overview
Kolla Ansible deploys OpenStack services in containers on your hosts using Ansible playbooks. It requires a control machine (where you run kolla ansible / ansible playbooks) and one or more target hosts (baremetal or VMs). The official quick start and user guides are the authoritative references. (OpenStack Docs)

---

## 2) Requirements

### Host / hardware (minimum for evaluation / all-in-one)
• CPU: 2+ cores (more for production)  
• Memory: ≥ 8 GB for an evaluation all-in-one (production needs much more).  
• Disk: ≥ 40 GB free (evaluate); production requires larger volumes for images/volumes/DB).  
These are evaluation minima, scale up for production. (OpenStack Docs)

### Operating systems (supported)
• Commonly supported: Ubuntu LTS, CentOS / RHEL derivatives, and other distributions listed in the Kolla docs. Match the Kolla-Ansible release compatibility. (OpenStack Docs)

### Network
• At least 2 network interfaces recommended: one for management/internal traffic and one for external (public) traffic / provider network.  
• Use a consistent IP plan; configure DNS or /etc/hosts entries for all hosts.  
• Floating IPs and an external network are required for public access to instances. (OpenStack Docs)

### Software
• Python 3.x on control and target hosts.  
• Ansible: compatible major version per Kolla release (check release notes). Typically, Ansible 4+ for modern Kolla releases. (OpenStack Docs)  
• Container runtime (Docker or other supported container runtime) on target hosts; Kolla deploys services as containers. (OpenStack Docs)  
• pip3, git, python3-venv recommended on the control host.

### Access
• SSH connectivity from control host to all target hosts. Passwordless SSH keys are highly recommended for Ansible. Ansible can use passwords but keys are preferred. (OpenDev: Free Software Needs Free Tools)

---

## 3) Architecture options

### All-in-one
Single host runs all OpenStack containers, for evaluation only.

### Multinode
(separate roles) control (API/db/message), compute (nova compute), storage (cinder, swift), network (neutron) nodes.  
Use inventory to map hosts to roles. See production architecture guide for clustering and HA patterns. (GitHub)

---

## 4) Prepare control workstation (where you run Ansible / kolla-ansible)
Run these commands on your control machine (example uses Rocky Linux):

```bash
# --- Install system dependencies ---
sudo dnf -y update
sudo dnf -y install epel-release
sudo dnf -y install python3 python3-pip python3-virtualenv python3-devel git gcc openssl-devel libffi-devel

# --- Create a virtual environment (recommended) ---
python3 -m venv ~/kolla-venv
source ~/kolla-venv/bin/activate

# --- Clone the Kolla-Ansible repository ---
git clone https://github.com/openstack/kolla-ansible.git
cd kolla-ansible

# (Optional) Checkout a stable release branch — example for 2024.1:
# git checkout stable/2024.1

# --- Install Python requirements and Kolla-Ansible itself ---
pip install -U pip wheel
pip install -r requirements.txt
pip install .

```
## 5) Prepare target hosts (bootstrap tasks)

On each target host (control/compute/storage):

```bash
sudo dnf -y update
sudo dnf -y install epel-release
sudo dnf -y install openssh-server python3 python3-pip sudo rsync ca-certificates curl

# Enable and start SSH service (if not already running)
sudo systemctl enable --now sshd

# --- Create a user for Ansible / Kolla operations ---
sudo useradd -m -s /bin/bash kolla

# Set a password (optional, if not using SSH keys)
# sudo passwd kolla

# Copy your SSH public key for passwordless access
# (Run this from the control node after generating an SSH key)
# ssh-copy-id kolla@<target-host>

# --- Grant passwordless sudo privileges to the 'kolla' user ---
echo "kolla ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/kolla
sudo chmod 440 /etc/sudoers.d/kolla

# --- Install and enable container runtime (Docker example) ---
sudo dnf -y install docker
sudo systemctl enable --now docker

# Add the 'kolla' user to the docker group so it can manage containers
sudo usermod -aG docker kolla

# Verify Docker is working
sudo docker run hello-world || echo "Docker test container executed"

```
## 6) Example Ansible inventory

Multinode (simple 3-node example)
```
cat ~inventory/multinode:

# [control]
controller1 ansible_host=192.0.2.11
controller2 ansible_host=192.0.2.12
controller3 ansible_host=192.0.2.13

# [network]
controller1
controller2
controller3

# [compute]
compute1 ansible_host=192.0.2.21
compute2 ansible_host=192.0.2.22

# [storage]
storage1 ansible_host=192.0.2.31
```

Modify groups and hostnames to reflect your deployment.
Kolla ships sample inventory files—adjust them for your environment. (OpenDev: Free Software Needs Free Tools)

## 7) /etc/kolla configuration files (copy-paste templates)
```
Create /etc/kolla and put these files there.
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
```

### A) globals.yml (minimal example — copy & edit)
```
--- Kolla-Ansible Global Config (Rocky Linux Example) ---
kolla_base_distro: "rocky"
kolla_install_type: "binary"
openstack_release: "2024.1"

kolla_internal_vip_address: "10.0.0.100"
kolla_external_vip_address: "192.168.0.100"

network_interface: "eth0"
neutron_external_interface: "eth1"

enable_haproxy: "yes"
enable_cinder: "yes"
enable_neutron_provider_networks: "yes"
enable_heat: "no"

kolla_ansible_user: "{{ lookup('env','USER') }}"
```

### B) passwords.yml
```
keystone_admin_password: "REPLACE_WITH_STRONG_PASSWORD"
keystone_admin_token: "REPLACE_IF_USED"
admin_password: "REPLACE_WITH_STRONG_PASSWORD"
mysql_root_password: "REPLACE_WITH_STRONG_PASSWORD"
rabbitmq_password: "REPLACE_WITH_STRONG_PASSWORD"
# many more keys, use kolla tools to generate the full file or copy from examples

# Generate recommended password file:

kolla-genpwd
# this will populate /etc/kolla/passwords.yml with generated passwords
```

## 8) Install Kolla-Ansible (detailed) 
Two common approaches: 

### A) Install from PyPI (released package) 
```
#--- Install Kolla-Ansible via PyPI (inside your venv) --- pip install -U pip wheel 
pip install kolla-ansible 

#--- Create configuration directories --- 
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla 

--- Copy example configuration and inventory templates --- # (these come with the installed Python package) cp -r $(python3 -c "import kolla_ansible;  

print(kolla_ansible.__path__[0])")/etc/kolla/* /etc/kolla/ 
```
Create an inventory directory for your setup mkdir -p ~/inventory 
```
(Optional) Copy provided example inventories cp /usr/local/share/kolla-ansible/ansible/inventory/*  ~/inventory/ 2>/dev/null || true 
```
Notes for Rocky Linux: 
• Files from kolla-ansible are installed under /usr/local/share/kolla ansible/ansible/. 
• You can verify installation with: 
B) Install from source (recommended for choosing branches) 
```
# --- Clone the official Kolla-Ansible repository --- git clone https://github.com/openstack/kolla-ansible.git cd kolla-ansible 
# (Optional) Checkout a stable release branch 
# Example: git checkout stable/2024.1 

# --- Install requirements and Kolla-Ansible itself --- pip install -U pip wheel 
pip install -r requirements.txt 
pip install . 

# --- Copy configuration templates to /etc/kolla --- sudo mkdir -p /etc/kolla 
sudo chown $USER:$USER /etc/kolla 
cp -r etc/kolla/* /etc/kolla/ 

# --- Prepare your inventory folder --- 
mkdir -p ~/inventory 
cp -r ansible/inventory/* ~/inventory/ 2>/dev/null || true 
```
#### Verification 
After either installation method, confirm it works:

**Should display version info** 
```kolla-ansible --version ```

**List available Ansible playbooks** 
```ls /usr/local/share/kolla-ansible/ansible/ 2>/dev/null || ls  ansible/``` 

## 9) Deploy commands (bootstrap → pull → deploy) 
Run from your control host (in the virtualenv where kolla-ansible is installed). Use the  inventory you prepared. 

### 1. Bootstrap hosts (installs docker, creates users, configures kernel, etc.) 
```
# example, using inventory file at 'inventory/multinode' kolla-ansible -i inventory/multinode bootstrap-servers 
```
### 2. Prechecks / hosts prep 
```
kolla-ansible -i inventory/multinode prechecks 
kolla-ansible -i inventory/multinode pull 
pull will fetch container images required for the configured OpenStack release. 3. Run deploy playbooks 
kolla-ansible -i inventory/multinode deploy 
```
### 3. Post-deploy (configure admin openrc) 
```
# Generate admin-openrc.sh 
kolla-ansible post-deploy 
# Then source the file created in /etc/kolla or check docs  for exact path 
source /etc/kolla/admin-openrc.sh 

# Important: for development quickstart there are simplified scripts; for production you  may want to use the full recommended sequence in the quickstart guide. (OpenStack  Docs) 
```
### 10) Validation and smoke tests 
After post-deploy: 
```
# source admin credentials (path from post-deploy) source /etc/kolla/admin-openrc.sh 
# check keystone services / token 
openstack token issue
# list services 
openstack service list 
# try creating small temp resources (flavor, network, boot a  tiny instance) 
openstack flavor create --id 0 --ram 64 --disk 1 --vcpus 1  tiny 
You can also use kolla-ansible provided healthchecks and monitoring playbooks; see  operating docs. (OpenStack Docs) 
```
### 11) Common variables & useful tweaks 
(Examples you may change in /etc/kolla/globals.yml) 
•  openstack_release - must match the images you pull / desired release.  (OpenDev: Free Software Needs Free Tools) 
• kolla_install_type: source|binary - choose whether to run services from source or  prebuilt packages. 
• docker_namespace and kolla_internal_vip_address - set to control image names  and cluster VIPs. 
• neutron_plugin_agent: openvswitch - common default; set to linuxbridge if you  prefer. 

Refer to etc/kolla/globals.yml for the complete list. (OpenDev: Free Software  Needs Free Tools) 

### 12) Backup, upgrade, and operational notes 
• Keep backups of /etc/kolla/globals.yml, /etc/kolla/passwords.yml, and your  Ansible inventory - these are critical. 
• For upgrades: follow Kolla-Ansible release notes and upgrade procedures; test in  a staging environment first. (OpenStack Docs) 

### 13) Troubleshooting checklist & common errors 
• Ansible SSH failures: ensure SSH keys, ansible_user/kolla_ansible_user match,  and sudo permissions exist. (Ansible will log the failing task.) (OpenDev: Free  Software Needs Free Tools)
• Image pull errors: ensure the control host and target hosts can reach the  container registry; consider mirroring images to a private registry if bandwidth  constrained. 
• Service start failures (containers exiting): docker logs <container> on host;  check /var/log/kolla and journalctl for system issues. 
• Network issues: validate host routing, bridge interfaces, and MTU; check  neutron logs inside containers. 
• Mismatched versions: ensure Ansible, Docker, and OS versions are compatible  with chosen Kolla-Ansible release. Consult release notes. (OpenStack Docs) 

### 14) References (read first) 
• Kolla-Ansible - Quickstart & User guides (main docs, step-by-step):  (OpenStack Docs) 
• Kolla-Ansible project (GitHub) (source, README): (GitHub) 
• Ansible deployment notes and inventory guidance (repo docs): (OpenDev:  Free Software Needs Free Tools) 
• Globals.yml reference (variables) (opendev): (OpenDev: Free Software Needs  Free Tools) 
• Operating Kolla (post-deploy, upgrades, maintenance): (OpenStack Docs)

---
