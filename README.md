
# OpenStack Installation Guide using Packstack

This README file outlines the steps required to install OpenStack using the Packstack installer.

## Prerequisites

Ensure the following before starting the installation:

- A VM with the following specs:
  - **RAM**: 10 GB
  - **CPU**: 2
  - **Disk**: 25 GB
  - **NICs**: 2
- **Operating System**: Rocky 9
- **Virtualization enabled** on the host
- **Static IP configuration** for the NICs, and **disable IPv6**.

---

## 1. Host Preparation

### 1.1 Hardware Setup
1. **Download Rocky 9 ISO**:
   - [Download Rocky 9 ISO](https://azza-permanent.oss.prod-cloud-ocb.orange-business.com/Rocky-9.4-x86_64-dvd.iso)
   
2. **Create VM** with the following resources:
   - **RAM**: 10 GB
   - **CPU**: 2
   - **Disk**: 25 GB
   - **NIC cards**: 2
   
3. **OS Installation**:
   - Define language and configure two NICs for static IP.
   - Disable IPv6 and set the manual IP as the same as the DHCP-assigned one.

4. **Reboot after installation**.

### 1.2 Validate Environment
1. **Check Internet Connectivity**:
   ```bash
   ping 8.8.8.8
   ```

2. **Check Static IPs for NICs**:
   ```bash
   cat /etc/NetworkManager/system-connections/ens160.nmconnection
   cat /etc/NetworkManager/system-connections/ens192.nmconnection
   ```

---

## 2. Host Configuration

1. **Stop Firewall**:
   ```bash
   systemctl disable firewalld
   systemctl stop firewalld
   ```

2. **Disable SELinux**:
   ```bash
   setenforce 0
   sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
   ```

3. **Set Hostname**:
   ```bash
   hostnamectl set-hostname controller
   ```

4. **Update `/etc/hosts`**:
   Add the new hostname with the IP address:
   ```bash
   echo "<IP> controller" >> /etc/hosts
   ```

---

## 3. Software Installation

1. **Install OpenStack Packages**:
   ```bash
   dnf config-manager --enable crb
   dnf install -y centos-release-openstack-antelope
   ```

2. **Update System**:
   ```bash
   dnf -y update
   ```

3. **Install Packstack**:
   ```bash
   dnf install -y openstack-packstack
   ```

---

## 4. Run Packstack to Install OpenStack

1. **Create Answer File**:
   ```bash
   packstack --gen-answer-file=/root/answers.txt --os-neutron-l2-agent=openvswitch --os-neutron-ml2-mechanism-drivers=openvswitch --os-neutron-ml2-tenant-network-types=vxlan --os-neutron-ml2-type-drivers=vxlan,flat --provision-demo=n --os-neutron-ovs-bridge-mappings=extnet:br-ex --os-neutron-ovs-bridge-interfaces=br-ex:ens192 --keystone-admin-passwd=redhat --os-heat-install=n
   ```

2. **Run Installation**:
   ```bash
   packstack --answer-file=/root/answers.txt
   ```

   The installation may take 30 to 45 minutes. Do not cancel during the process.

3. **Verify Network Interfaces**:
   ```bash
   ip a
   ```

---

## 5. Post-Installation Validation

1. **Check OpenStack Services**:
   ```bash
   systemctl list-units --type=service | grep openstack
   ```

2. **Check Network Interfaces and Storage**:
   ```bash
   lsblk
   ```

3. **Validate Cloud Access**:
   Open the cloud portal using the IP of the dashboard.
![304160615-94d41415-7c84-4392-ab1e-c161859d31b8](https://github.com/user-attachments/assets/c59de97a-646f-4597-963d-7fa5dea37662)

---

## 6. Customize Dashboard

1. **Change OpenStack Logo**:
   - Download an image.
   - Navigate to `/usr/share/openstack-dashboard/static/dashboard/img/`.
   - Replace `logo-splash.svg` with your custom image:
   ```bash
   mv logo-splash.svg logo-splash.svg.bkp
   cp <your_img_path> logo-splash.svg
   chmod 644 logo-splash.svg
   ```

---

## 7. Manage OpenStack via CLI

1. **Source Admin Credentials**:
   ```bash
   source keystonerc_admin
   ```

2. **List All Servers**:
   ```bash
   openstack server list
   ```
---

## 8. Networking Configuration

1. **Create Provider Network**:
   ```bash
   neutron net-create <external_network> --provider:network_type flat --provider:physical_network extnet --router:external
   ```

2. **Create Private Network**:
   ```bash
   neutron net-create <private_network>
   ```

3. **Create Router and Connect Networks**:
   ```bash
   neutron router-create <router>
   neutron router-gateway-set <router> <external_network>
   neutron router-interface-add <router> <private_subnet>
   ```
![1739195995138 (1)](https://github.com/user-attachments/assets/b639e742-023e-440a-b720-a67a90621ee7)

---

## 9. Instance Deployment

1. **Create Image**:
   ```bash
   curl -L http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img | glance image-create --name='cirros image' --visibility=public --container-format=bare --disk-format=qcow2
   ```

2. **Launch Instance**:
   ```bash
   openstack server create --flavor <id> --image <id> --nic net-id=<id> <instance_name>
   ```

3. **Assign Floating IP**:
   ```bash
   openstack floating ip create <external_network>
   openstack floating ip set --port <instance_port_id> <floating_ip>
   ```

4. **Attach Data Disk to Instance**:
   ```bash
   openstack volume create --size <size> <volume_name>
   openstack server add volume <instance_id> <volume_name>
   ```
![1739195995015 (1)](https://github.com/user-attachments/assets/089dafcb-f6c7-494b-ac84-a5701ef2fb91)

---

## Troubleshooting

- **Check Network Interfaces**: Ensure configurations of `br-ex` and `ens192` are correct.
- **Verify Services**: Use `systemctl list-units` to check if OpenStack services are running.
- **Firewall Rules**: Ensure SSH and ICMP ports are open.

---

## License

This project is licensed under the MIT License.
