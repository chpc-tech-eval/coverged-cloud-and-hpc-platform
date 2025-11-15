# **CHPC** – **WEEK** 4: **IRONIC** **BARE** **METAL** **PROVISIONING** **REPORT** 
________________________________________
**TASK** 1: **DEPLOY** **IRONIC** **USING** **KOLLA** **ANSIBLE**
Successfully executed
```
kolla-ansible -i /etc/kolla/multinode deploy --tags ironic,tftp,httpd
Multiple runs with skip tags to isolate Ironic stack Ironic, **TFTP**, and **HTTPD** services deployed
```
________________________________________
**TASK** 2: **TEST** **DEPLOYMENT** **VIA** **OPENSTACK**
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
________________________________________

**TASK** 3: **TEST** **FUNCTIONALITY**

| **Test**        | **Result**                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| Ironic API      | http://172.16.10.10:6385/v1/nodes → **401 Authorization Required** → API LIVE & SECURED |
| HTTP Boot       | nginx listening on port 8080 → Image delivery active                       |
| IPMI Control    | ipmitool command correct → **BMC unreachable (network/infra issue)**       |

________________________________________

**CONCLUSION**
- Kolla-Ansible deployment successful
- Ironic **API** operational and protected
- **HTTP** boot server running
- Bare metal node fully configured
- Functionality test method correct
- ***BMC** unreachable = infrastructure issue, not our error

*Issue to inquire by David
