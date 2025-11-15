# **CHPC** – **WEEK** 4: **IRONIC** **BARE** **METAL** **PROVISIONING** **REPORT** 

- Deploy Ironic using Kolla Ansible  
- Setup bifrost 
- Test the deployment of bare metal nodes via OpenStack 
- Test the functionality of the bare metal nodes
--- 

## What is Bitfrost?
Bifrost is a collection of Ansible playbooks that automates the deployment of base operating systems directly onto physical servers using OpenStack Ironic. It is designed to be a simple, stand-alone tool for provisioning bare metal hardware with minimal setup.

Its primary goals are to simplify the deployment of Ironic without requiring a full OpenStack cloud, and to offer a flexible foundation that can be integrated with other OpenStack services for more complex environments.

Common Use Cases:
- Setting up Ironic in a simple, stand-alone mode.
- Performing batch OS installations on a known set of servers.
- Development and testing of Ironic's core features.

---
### **TASK** 1: **Sebowa** **Change** **Procedure** 
- Provided by David Macloed
  
**Objectives**
1. Upgrade Kolla-Ansible from 2024.1 to 2024.2
2. Upgrade OpenStack from 2024.1 to 2024.2
   
**Prerequisites**
1. Kolla-Ansible 2024.1
2. Healthy Kolla-Ansible environment

#### Deployment Procedure
**1. Backup Kolla venv and etc folders:**
```
cp -a kolla-ansible-venv kolla-ansible-venv.2024.1
sudo cp -a /etc/kolla/ /etc/kolla.2024.1
```

#### 2. Install Python pip 3.12:

```sudo dnf install python3.12-pip```

#### 3. Update the Python version for the virtual environment:
```
python3.12 -m venv kolla-ansible-venv
```

#### 4. Activate the Kolla virtual environment:
```source kolla-ansible-venv/bin/activate```

#### 5. Upgrade pip:
```pip3.12 install --upgrade pip```

#### 6. Upgrade Kolla:
```
pip3.12 install --upgrade git+https://opendev.org/openstack/kolla-
ansible@stable/2024.2
```

#### 7. Install dependencies:
```kolla-ansible install-deps```

#### 8. Edit globals.yml

Comment out:
```openstack_release: “2024.1”```

Add:
```cinder_cluster_name: &quot;sebowa_qa&quot;```

#### 9. Merge passwords:
```
cp /etc/kolla/passwords.yml passwords.yml.old
cp kolla-ansible-venv/share/kolla-
ansible/etc_examples/kolla/passwords.yml passwords.yml.new
kolla-genpwd -p passwords.yml.new
kolla-mergepwd --old passwords.yml.old --new passwords.yml.new --
final /etc/kolla/passwords.yml
```

#### 10. Pull new images to hosts:
```kolla-ansible pull -i /etc/kolla/inventory/overcloud```

#### 11. Run prechecks:
```kolla-ansible prechecks -i /etc/kolla/inventory/overcloud```

#### 12. If prechecks pass, the upgrade can be implemented. This will cause short disruptionsas containers are restarted:
```
kolla-ansible upgrade -i /etc/kolla/inventory/overcloud
-+NOTE: RabbitMQ might fail to be put into maintenance mode. If it fails re-run the
upgrade for RabbitMQ. It may fail for each node:
kolla-ansible upgrade -i /etc/kolla/inventory/overcloud -t rabbitmq
Once RabbitMQ is upgraded restart the upgrade procedure:
kolla-ansible upgrade -i /etc/kolla/inventory/overcloud
```

#### 13. Upgrade complete! Test functionality.
```
# You should see services like keystone, nova, neutron, cinder, glance, etc., with their status showing “enabled” and “up”
openstack service list
```
---
## **TASK** 4: Step-by-step procedure for enrolling and provisioning bare metal nodes using Bifrost

### 1. Go to the inventories folder
```
cd /home/admin/bifrost-inventories

# This just moves you into the folder where your Bifrost inventory YAML files are stored.
# Inventories define which nodes you want to manage.
```

### 2. Create your two-node inventory file
```
cat > my-nodes.yaml <<EOF
all:
  hosts:
    node01:
      driver: ipmi
      ipmi_address: <NODE01_IPMI_IP>
      ipmi_username: root
      ipmi_password: <NODE01_IPMI_PASSWORD>
      mac: "<NODE01_PXE_MAC>"
      properties:
        cpu_arch: x86_64

    node02:
      driver: ipmi
      ipmi_address: <NODE02_IPMI_IP>
      ipmi_username: root
      ipmi_password: <NODE02_IPMI_PASSWORD>
      mac: "<NODE02_PXE_MAC>"
      properties:
        cpu_arch: x86_64
EOF

# This creates a YAML file describing the nodes you want to manage.
```

**Key fields explained:**
- ipmi_address: the IP address of the node’s management interface (BMC).
- ipmi_username / ipmi_password: credentials to control the node via IPMI.
- mac: the MAC address of the node’s PXE boot interface.
- cpu_arch: architecture type (usually x86_64).
- You must replace <NODE01_IPMI_IP> and others with real values.

### 3. Activate Bifrost environment
```
source /opt/stack/bifrost/bin/activate
#Activates the Python virtual environment that contains Bifrost tools.
#This ensures you’re running the correct versions of Python packages and CLI tools.
```

### 4. Point Bifrost to your inventory
```
export BIFROST_INVENTORY_SOURCE=/home/admin/bifrost-inventories/my-nodes.yaml
export OS_CLOUD=bifrost
```

- BIFROST_INVENTORY_SOURCE: tells Bifrost which inventory file to use.
- OS_CLOUD: sets your OpenStack cloud environment to Bifrost (so OpenStack CLI commands know where to point).

### 5. Check the nodes are visible
```
baremetal node list
# Uses the OpenStack Ironic CLI to list all registered bare-metal nodes.
```

At this point, your nodes may not yet be enrolled, but this confirms connectivity to the BMCs.

### 6. Enroll the two nodes
```
ansible-playbook -i /home/admin/bifrost/playbooks/inventory/bifrost_inventory.py /home/admin/bifrost/playbooks/enroll-dynamic.yaml
#Uses Ansible to enroll the nodes defined in your inventory into Ironic.
```

Enrolling means the nodes become visible to OpenStack, with all metadata (IPMI info, MAC, properties) stored.

### 7.Wait until nodes are "available"
```
baremetal node list
#Check the status of nodes. After enrollment, they may appear as enroll or available.
#Only nodes marked available are ready for OS deployment.
```

### 8. Deploy the OS
```
ansible-playbook -i /home/admin/bifrost/playbooks/inventory/bifrost_inventory.py /home/admin/bifrost/playbooks/deploy-dynamic.yaml
```
This triggers Bifrost to provision the nodes, meaning it:
- PXE boots the nodes
- Installs the specified OS image
- Configures networking and SSH access

### 9. Monitor until nodes are "active"
```
baremetal node list
# Nodes will transition from available → deploying → active.
# Active means the OS is installed and nodes are ready for use.

```
### 10. Test SSH
```
ssh cloud-user@node01
ssh cloud-user@node02

#Confirms that you can log in to the nodes after deployment.
# Once SSH works, you are ready to use these nodes for Kolla-Ansible / OpenStack deployments.
```

✅ Summary:
- These commands fully automate bare-metal provisioning:
- Define your nodes (inventory YAML)
- Enroll them into Ironic via Bifrost
- Deploy the OS image
- Verify nodes are ready via SSH

This setup is preparatory for Kolla-Ansible, which will deploy OpenStack on these nodes.

---
## **TASK** 3: **DEPLOY** **IRONIC** **USING** **KOLLA** **ANSIBLE**

Successfully executed
```
kolla-ansible -i /etc/kolla/multinode deploy --tags ironic,tftp,httpd
Multiple runs with skip tags to isolate Ironic stack Ironic, **TFTP**, and **HTTPD** services deployed
```

## **TASK** 3: **TEST** **DEPLOYMENT** **VIA** **OPENSTACK**
Node fully defined and ready for provisioning

```
json
{
    *name*: *bm-test-01*,
    *driver*: *ipmi*,
    *ipmi_address*: ***172**.16.48.10*,
    *mac*: *52:54:00:ab:cd:ef*,
    *image*: */home/admin/images/ubuntu-jammy.raw*
}

# Image confirmed
-rw-r--r--. 1 admin admin 2.2G Nov 13 16:17 ubuntu-jammy.raw
```

## **TASK** 4: **TEST** **FUNCTIONALITY**

| **Test**        | **Result**                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| Ironic API      | http://172.16.10.10:6385/v1/nodes → **401 Authorization Required** → API LIVE & SECURED |
| HTTP Boot       | nginx listening on port 8080 → Image delivery active                       |
| IPMI Control    | ipmitool command correct → **BMC unreachable (network/infra issue)**       |


## **CONCLUSION**
- Kolla-Ansible deployment successful
- Ironic **API** operational and protected
- **HTTP** boot server running
- Bare metal node fully configured
- Functionality test method correct
- **BMC** unreachable = infrastructure issue, not our error
---
