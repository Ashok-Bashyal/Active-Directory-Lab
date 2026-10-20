
# Active Directory SOC HomeLab
### Active Directory | Splunk Enterprise | Sysmon | VMware | Kali Linux | NetExec | Atomic Red Team

![Platform](https://img.shields.io/badge/Platform-VMware_Workstation-blue)
![Operating Systems](https://img.shields.io/badge/Windows-10_%7C_Server_2022-blue)
![SIEM](https://img.shields.io/badge/SIEM-Splunk_Enterprise-orange)
![Monitoring](https://img.shields.io/badge/Monitoring-Sysmon-green)
![Security](https://img.shields.io/badge/Security-Purple_Team-purple)

## Project Overview

This project documents the design, implementation, and security monitoring of a fully virtualized Active Directory environment using VMware Workstation Pro.

The primary objective was to simulate a small enterprise network where I could configure Windows infrastructure, collect security logs, generate controlled attack activity, and investigate suspicious behavior using Splunk Enterprise.

The environment consists of four virtual machines:

- **Windows Server 2022:** Active Directory Domain Controller and DNS Server.
- **Windows 10:** Domain-joined workstation used for security monitoring and testing.
- **Ubuntu Server:** Hosts Splunk Enterprise for centralized security monitoring.
- **Kali Linux:** Security testing machine used to simulate controlled authentication attacks.

To collect endpoint telemetry, I installed Microsoft Sysmon using Olaf Hartong's modular configuration and deployed Splunk Universal Forwarder on both Windows machines.

I configured Splunk Enterprise to collect Windows Security, System, Application, and Sysmon events in a dedicated index named `endpoint`.

After configuring the environment, I performed controlled RDP authentication testing using NetExec and examined the resulting Windows authentication events in Splunk.

I also used Atomic Red Team to simulate selected adversary techniques mapped to the MITRE ATT&CK framework.

The overall purpose of this project was to develop practical experience in:

- Active Directory administration
- Windows Server configuration
- Network configuration and troubleshooting
- Security Information and Event Management (SIEM)
- Windows event log analysis
- Endpoint monitoring
- Threat detection
- Security investigations
- Attack simulation
- Detection engineering

> **Disclaimer:** All security testing was performed within a personally controlled virtual environment for educational purposes. The attack simulations and techniques documented in this project were used only against authorized lab systems.

---

## Table of Contents

1. [Network Architecture](#1-network-architecture)
2. [Tools and Technologies](#2-tools-and-technologies)
3. [Virtual Machine Installation](#3-virtual-machine-installation)
4. [VMware Network Configuration](#4-vmware-network-configuration)
5. [Ubuntu Server and Splunk Installation](#5-ubuntu-server-and-splunk-installation)
6. [Active Directory Domain Services](#6-active-directory-domain-services)
7. [Active Directory Users and Organizational Units](#7-active-directory-users-and-organizational-units)
8. [Joining Windows 10 to Active Directory](#8-joining-windows-10-to-active-directory)
9. [Sysmon Installation and Configuration](#9-sysmon-installation-and-configuration)
10. [Splunk Universal Forwarder Configuration](#10-splunk-universal-forwarder-configuration)
11. [Kali Linux Network Configuration](#11-kali-linux-network-configuration)
12. [RDP Authentication Testing with NetExec](#12-rdp-authentication-testing-with-netexec)
13. [Investigating Authentication Events in Splunk](#13-investigating-authentication-events-in-splunk)
14. [Atomic Red Team Attack Simulation](#14-atomic-red-team-attack-simulation)
15. [Splunk Detection Queries](#15-splunk-detection-queries)
16. [Challenges and Troubleshooting](#16-challenges-and-troubleshooting)
17. [Project Results](#17-project-results)
18. [Lessons Learned](#18-lessons-learned)
19. [Security Considerations](#19-security-considerations)
20. [References](#20-references)

---

# 1. Network Architecture

The first stage of the project involved designing the network architecture for the Active Directory SOC HomeLab.

The network was designed to simulate a small organization consisting of an Active Directory Domain Controller, a Windows workstation, a centralized Splunk server, and a separate security testing machine.

All virtual machines communicate through a VMware NAT network.

### Network Diagram

![Active Directory SOC HomeLab Network Architecture](assets/screenshots/architecture/01-lab-logical-diagram.jpeg)

The architecture illustrates the relationship between the virtual machines, centralized monitoring infrastructure, and virtual network.

### Network Configuration

| Component | IP Address | Purpose |
|---|---|---|
| Network | 192.168.10.0/24 | Private lab network |
| VMware NAT Gateway | 192.168.10.2 | Internet gateway |
| VMware Host Adapter | 192.168.10.1 | Host-side network interface |
| Domain Controller | 192.168.10.7 | AD DS and DNS |
| Splunk Server | 192.168.10.10 | Centralized log collection |
| Windows 10 Target | 192.168.10.100 | Domain-joined workstation |
| Kali Linux | 192.168.10.250 | Security testing machine |

### Active Directory Domain

```text
DNS Domain: adlab.local
NetBIOS Domain: ADLAB
Domain Controller: ADDC01
Windows Client: target-PC
```

The original network diagram uses HomeLab as an informal project label. The actual Active Directory DNS domain configured during implementation was `adlab.local`.

### How the Network Works

The Windows Server 2022 machine acts as the Domain Controller and provides centralized authentication and DNS services.

Windows 10 is joined to the Active Directory domain, allowing domain users to authenticate using their assigned accounts.

The Ubuntu Splunk Server receives logs from both Windows machines using Splunk Universal Forwarder.

Kali Linux is used to generate controlled security events against the Windows 10 workstation.

The activity is recorded by Windows Event Logging and Sysmon, forwarded to Splunk Enterprise, and analyzed using Splunk Search Processing Language (SPL).

### Security Monitoring Workflow

```text
Kali Linux
192.168.10.250
      |
      | Controlled security testing
      v
Windows 10 Target
192.168.10.100
      |
      | Windows Security Logs
      | Sysmon Events
      v
Splunk Universal Forwarder
      |
      | TCP 9997
      v
Splunk Enterprise
192.168.10.10
      |
      v
Security Event Analysis
      |
      v
Threat Detection and Investigation
```

The Domain Controller also sends its Windows Security and Sysmon logs to Splunk, allowing domain authentication activity to be correlated with events on the workstation.

---

# 2. Tools and Technologies

The following technologies were used to implement the project.

| Technology | Purpose |
|---|---|
| VMware Workstation Pro | Virtual machine management |
| Windows Server 2022 | Active Directory infrastructure |
| Windows 10 Pro | Domain-joined endpoint |
| Ubuntu Server | Linux operating system for Splunk |
| Kali Linux | Authorized security testing |
| Active Directory Domain Services | User and computer authentication |
| DNS Server | Active Directory domain name resolution |
| Splunk Enterprise | SIEM and centralized security monitoring |
| Splunk Universal Forwarder | Event log forwarding |
| Microsoft Sysmon | Detailed Windows endpoint telemetry |
| Olaf Hartong Sysmon Modular | Sysmon filtering configuration |
| NetExec | Controlled RDP authentication testing |
| Atomic Red Team | ATT&CK-mapped attack simulation |
| MITRE ATT&CK | Adversary behavior classification |
| PowerShell | Windows configuration and administration |
| SPL | Security log searching and analysis |

The tools were selected to provide experience with both Windows system administration and SOC-focused security monitoring.

---

# 3. Virtual Machine Installation

The environment was created using VMware Workstation Pro.

Four virtual machines were prepared to represent the different components of the organization.

## 3.1 Windows 10 Installation

The first virtual machine was Windows 10 Pro.

I downloaded the Windows 10 installation media from Microsoft's official website.

**Download:**

https://www.microsoft.com/software-download/windows10

### Installation Procedure

1. Download the Windows 10 Media Creation Tool.
2. Run the Media Creation Tool.
3. Accept the Microsoft license agreement.
4. Select **Create installation media**.
5. Select the preferred language and 64-bit architecture.
6. Choose **ISO file**.
7. Save the installation ISO.
8. Open VMware Workstation Pro.
9. Select **Create a New Virtual Machine**.
10. Choose **Typical (recommended)**.
11. Select the downloaded ISO.
12. Name the virtual machine `Windows 10 AD-Lab`.
13. Assign 50 GB of virtual disk capacity.
14. Review the virtual hardware.
15. Start the virtual machine.
16. Follow the Windows installation wizard.
17. Select Windows 10 Pro.
18. Choose **Custom: Install Windows only**.
19. Complete Windows setup using a local account.

Windows 10 Pro was selected because it supports Active Directory domain membership and Remote Desktop hosting.

After installation, I configured its network settings and changed the computer name to `target-PC`.

### Windows 10 Installation Screenshots

![Windows 10 VM Configuration](assets/screenshots/windows10-installation/10-vm-review.png)

<details>
<summary><strong>View all Windows 10 installation screenshots</strong></summary>

![Installation Media](assets/screenshots/windows10-installation/01-windows-10-installation-media.png)

![License Terms](assets/screenshots/windows10-installation/02-windows-10-license-terms.png)

![Create Installation Media](assets/screenshots/windows10-installation/03-create-installatin-media.png)

![Recommended Options](assets/screenshots/windows10-installation/04-use-recommended-option-when-upgrading.png)

![ISO Selection](assets/screenshots/windows10-installation/05-choose-iso-file.png)

![VMware Creation](assets/screenshots/windows10-installation/06-vmware-vm-creation-type.png)

![Attach ISO](assets/screenshots/windows10-installation/07-choose-iso-file-in-vmware.png)

![VM Naming](assets/screenshots/windows10-installation/08-vm-naming.png)

![Disk Capacity](assets/screenshots/windows10-installation/09-vm-capacity.png)

![Windows Setup](assets/screenshots/windows10-installation/11-windows-10-installation-setup.png)

![Install Now](assets/screenshots/windows10-installation/12-windows-10-installation-setup-install-now.png)

![Product Key](assets/screenshots/windows10-installation/13-product-key.png)

![Windows Edition](assets/screenshots/windows10-installation/14-choose-version.png)

![Windows License](assets/screenshots/windows10-installation/15-license-term.png)

![Custom Installation](assets/screenshots/windows10-installation/16-choose-custom.png)

![Disk Configuration](assets/screenshots/windows10-installation/17-disk-info.png)

![Windows Installing](assets/screenshots/windows10-installation/18-windows-10-installing.png)

</details>

## 3.2 Kali Linux Installation

Kali Linux was deployed using the official pre-built VMware virtual machine.

**Download:**

https://www.kali.org/get-kali/#kali-virtual-machines

### Installation Procedure

1. Navigate to Kali Linux's official website.
2. Select **Virtual Machines**.
3. Download the VMware image.
4. Extract the `.7z` archive using 7-Zip.
5. Open the extracted virtual machine directory.
6. Locate the VMware configuration file.
7. Open the configuration using VMware Workstation.
8. Verify the virtual hardware settings.
9. Configure the network adapter.
10. Start Kali Linux.

The version used for the project was Kali Linux 2026.2.

Kali was selected because it provides tools for network testing, authentication testing, security investigations, and other cybersecurity exercises.

### Kali Linux Installation Screenshots

![Kali Linux VMware](assets/screenshots/kali-vm-installation/03-linux-in-vmware.png)

<details>
<summary><strong>View additional Kali installation screenshots</strong></summary>

![Kali Download](assets/screenshots/kali-vm-installation/01-kali-linux-pre-built.png)

![Open Prebuilt Image](assets/screenshots/kali-vm-installation/02-open-pre-built.png)

</details>

## 3.3 Windows Server 2022 Installation

Windows Server 2022 was used as the Active Directory Domain Controller.

**Download:**

https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022

### Installation Procedure

1. Download the Windows Server 2022 ISO.
2. Create a new VM in VMware.
3. Attach the installation ISO.
4. Allocate the required CPU, RAM, and disk resources.
5. Install Windows Server 2022.
6. Configure the local Administrator password.
7. Sign in to Windows Server.
8. Verify Server Manager opens.
9. Configure the static network address.
10. Rename the server to `ADDC01`.

Windows Server was later promoted to a Domain Controller and configured with Active Directory Domain Services and DNS.

### Windows Server Installation Screenshots

![Windows Server Installation Complete](assets/screenshots/windows-server-installation/04-server-installation-complete.png)

<details>
<summary><strong>View additional Windows Server installation screenshots</strong></summary>

![Windows Server ISO](assets/screenshots/windows-server-installation/01-windows-server-iso-page.png)

![Server Installation](assets/screenshots/windows-server-installation/02-windows-server-installing.png)

![Administrator Setup](assets/screenshots/windows-server-installation/03-admin-user-creation-on-server.png)

</details>

## 3.4 Ubuntu Server Installation

Ubuntu Server was selected to host Splunk Enterprise.

**Download:**

https://ubuntu.com/server

### Installation Procedure

1. Download Ubuntu Server.
2. Create a VMware virtual machine.
3. Attach the Ubuntu Server ISO.
4. Boot into the installer.
5. Select **Try or Install Ubuntu Server**.
6. Configure the language and keyboard.
7. Complete the network and storage setup.
8. Create the Linux user account.
9. Complete the Ubuntu installation.
10. Restart and sign in.

After installation, I updated the operating system:

```bash
sudo apt update
sudo apt upgrade -y
```

These commands refresh the package metadata and install available package upgrades.

### Ubuntu Installation Screenshots

![Ubuntu Server Login](assets/screenshots/ubuntu-installation/04-splunk-server-login-page.png)

<details>
<summary><strong>View additional Ubuntu installation screenshots</strong></summary>

![Ubuntu Download](assets/screenshots/ubuntu-installation/01-ubuntu-server-page.png)

![Ubuntu Installer](assets/screenshots/ubuntu-installation/02-ubuntu-server-installation-page.png)

![Ubuntu User Creation](assets/screenshots/ubuntu-installation/03-ubuntu-server-user-account-creation.png)

![Ubuntu Updates](assets/screenshots/ubuntu-installation/05-splunk-server-updates.png)

![Ubuntu Package Updates](assets/screenshots/ubuntu-installation/06-splunk-server-packet-updates.png)

</details>

---

# 4. VMware Network Configuration

After installing the virtual machines, the next step was configuring a network that would allow all systems to communicate.

I used VMware Workstation Pro's Virtual Network Editor to configure the built-in NAT network `VMnet8`.

## 4.1 Configuring VMnet8

In VMware Workstation:

1. Open **Edit**.
2. Select **Virtual Network Editor**.
3. Select `VMnet8`.
4. Configure the network type as NAT.
5. Set the subnet IP to `192.168.10.0`.
6. Set the subnet mask to `255.255.255.0`.
7. Open **NAT Settings**.
8. Configure the gateway address as `192.168.10.2`.
9. Apply the settings.

### Network Settings

```text
Network: VMnet8
Type: NAT
Subnet: 192.168.10.0
CIDR: /24
Subnet Mask: 255.255.255.0
NAT Gateway: 192.168.10.2
Host Adapter: 192.168.10.1
```

The `/24` CIDR notation is equivalent to the subnet mask `255.255.255.0`.

### Why NAT?

NAT provides outbound connectivity through the VMware host while placing the virtual machines on a separate virtual subnet.

This allows the systems to communicate without directly bridging them onto the physical LAN.

However, NAT is not complete isolation. Firewall controls and appropriate lab boundaries are still important.

## 4.2 Assigning the Network to Each VM

For every virtual machine:

1. Open **Virtual Machine Settings**.
2. Select **Network Adapter**.
3. Choose **Custom: Specific virtual network**.
4. Select `VMnet8`.
5. Enable **Connect at power on**.
6. Save the settings.

This ensured all four machines used the same virtual network.

## 4.3 Configuring the Domain Controller IP

The Windows Server was assigned:

```text
IP Address: 192.168.10.7
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.2
DNS Server: 192.168.10.7
```

The Domain Controller uses its own DNS service after AD DS and DNS configuration.

## 4.4 Configuring Windows 10

The Windows workstation was configured with:

```text
IP Address: 192.168.10.100
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.2
DNS Server: 192.168.10.7
```

Using the Domain Controller as the DNS server is necessary for reliable Active Directory discovery and domain joining.

## 4.5 Configuring the Ubuntu Splunk Server

Ubuntu was assigned a static IPv4 address using Netplan.

Open the configuration file:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Example configuration:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.10.10/24
      nameservers:
        addresses:
          - 8.8.8.8
      routes:
        - to: default
          via: 192.168.10.2
```

This is a simplified representation. The original lab file also included interface MAC matching and interface naming.

Save the file, then apply the configuration:

```bash
sudo netplan generate
sudo netplan apply
```

Verify the address:

```bash
ip a
```

Verify routing:

```bash
ip route
```

Test connectivity:

```bash
ping -c 4 192.168.10.2
ping -c 4 8.8.8.8
ping -c 4 google.com
```

### Network Troubleshooting

During configuration, the Ubuntu server initially could not reach the internet.

The issue occurred because `192.168.10.1` was already assigned to the Windows host-side VMware network adapter while I had also attempted to use it as the NAT gateway.

I corrected the NAT gateway to `192.168.10.2` and updated Ubuntu's default route accordingly.

After applying the changes, Ubuntu successfully communicated with the gateway and external hosts.

This troubleshooting experience helped reinforce the differences between a host virtual adapter, virtual gateway, and endpoint IP address.

### VMware Network Screenshots

![VMware Network Configuration](assets/screenshots/vmware-network/01-virtual-network-configuration.png)

<details>
<summary><strong>View all additional VMware networking screenshots</strong></summary>

![Custom VM Network](assets/screenshots/vmware-network/02-set-vm-to-custom-virtual-network.png)

![Splunk Network](assets/screenshots/vmware-network/03-splunk-server-network-configuration.png)

![Splunk IP](assets/screenshots/vmware-network/04-ip-address-splunk-server1.png)

![Splunk IP Verification](assets/screenshots/vmware-network/05-ip-address-splunk-server2.png)

![Splunk Ping](assets/screenshots/vmware-network/06-splunk-server-ping.png)

![Server IP](assets/screenshots/vmware-network/07-windows-server-ip-change.png)

![Server Name](assets/screenshots/vmware-network/08-windows-server-name-change.png)

![Server Connectivity](assets/screenshots/vmware-network/09-windows-server-ping.png)

![Target Rename](assets/screenshots/vmware-network/10-rename-target-pc.png)

![Target Name Verification](assets/screenshots/vmware-network/11-verify-target-machine-name.png)

![Target Ethernet](assets/screenshots/vmware-network/12-ethernet-target-machine.png)

![Target IP](assets/screenshots/vmware-network/13-target-machine-ip-change.png)

</details>

---

# 5. Ubuntu Server and Splunk Installation

The Ubuntu Server was prepared to run Splunk Enterprise as the centralized monitoring platform.

## 5.1 Installing VMware Tools

VMware Tools improves communication between the host and guest operating systems.

On Ubuntu Server, I installed `open-vm-tools`:

```bash
sudo apt update
sudo apt install open-vm-tools -y
```

Enable the service:

```bash
sudo systemctl enable --now open-vm-tools
```

Verify:

```bash
systemctl status open-vm-tools
```

The service reported an active running status.

## 5.2 Installing SSH

To transfer the Splunk installer from my Windows host to Ubuntu, I installed OpenSSH Server.

```bash
sudo apt install openssh-server -y
```

Enable SSH:

```bash
sudo systemctl enable --now ssh
```

Verify:

```bash
sudo systemctl status ssh
```

Where UFW is active, access to SSH can be restricted to the Windows VMware host:

```bash
sudo ufw allow from 192.168.10.1 to any port 22 proto tcp
```

## 5.3 Downloading Splunk Enterprise

I downloaded the Linux `.deb` package for Splunk Enterprise from the official Splunk website.

https://www.splunk.com/

The lab used Splunk Enterprise version 10.6.1.1.

The downloaded installer was stored on my Windows computer before being transferred to Ubuntu.

## 5.4 Transferring Splunk to Ubuntu

I used SCP to transfer the installation package.

From Windows PowerShell:

```powershell
cd F:\
```

Then:

```powershell
scp .\splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb rangeuser21@192.168.10.10:/home/rangeuser21/
```

SCP transfers files securely using SSH.

After the transfer, I verified the package existed on Ubuntu:

```bash
ls -lh ~/splunk*.deb
```

Because Splunk was distributed as a `.deb` installer, mounting an ISO was unnecessary.

## 5.5 Installing Splunk Enterprise

On Ubuntu:

```bash
sudo apt install ~/splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb
```

Verify the package:

```bash
dpkg -s splunk | grep -E 'Status|Version'
```

The main Splunk directory is:

```text
/opt/splunk/
```

Important subdirectories:

| Directory | Purpose |
|---|---|
| `/opt/splunk/bin/` | Splunk executables |
| `/opt/splunk/etc/` | Splunk configuration |
| `/opt/splunk/var/log/splunk/` | Internal Splunk logs |
| `/opt/splunk/var/lib/splunk/` | Indexed data |

## 5.6 Starting Splunk Enterprise

The documented setup used the dedicated `splunk` service account.

```bash
sudo -u splunk /opt/splunk/bin/splunk start --accept-license
```

During initial startup, Splunk administrator credentials were configured.

Boot-start configuration can be enabled using supported Splunk CLI options for the installed version and service account.

## 5.7 Accessing Splunk Web

From the Windows host, I accessed:

```text
http://192.168.10.10:8000
```

Port usage:

| Port | Purpose |
|---|---|
| TCP 8000 | Splunk Web |
| TCP 9997 | Splunk Forwarder receiving port |
| TCP 22 | SSH access to Ubuntu |

### Splunk Storage Troubleshooting

During Splunk installation, the Ubuntu filesystem ran out of available space.

The installer produced:

```text
No space left on device
```

I expanded the VMware virtual disk to 40 GB.

However, expanding the virtual disk in VMware did not automatically expand the filesystem inside Ubuntu.

The system used LVM, so I examined the partition layout:

```bash
lsblk
df -h
```

The lab's root volume was located on:

```text
/dev/ubuntu-vg/ubuntu-lv
```

After verifying the partition layout and expanding the VMware disk, I extended the Linux partition and LVM storage:

```bash
sudo growpart /dev/sda 3
sudo pvresize /dev/sda3
sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
```

These commands apply to the specific LVM layout in this lab and should not be run blindly on another system.

After resizing, Ubuntu recognized approximately 38 GB of root filesystem capacity, allowing the Splunk installation to complete.

This demonstrated the difference between virtual disk capacity and usable space inside the guest operating system.

### Splunk Server Screenshots

![Splunk Dashboard](assets/screenshots/splunk-server/10-splunk-server-dashboard.png)

<details>
<summary><strong>View all additional Splunk installation screenshots</strong></summary>

![Splunk Download](assets/screenshots/splunk-server/01-splunk-enterprise-image-download.png)

![VMware Tools](assets/screenshots/splunk-server/02-open-vm-tools-verify-splunk-server.png)

![SSH Service](assets/screenshots/splunk-server/03-systemctl-status-ssh.png)

![SSH Firewall Rule](assets/screenshots/splunk-server/04-allow-ssh-from-windows-in-splunk-server.png)

![Installer Transfer](assets/screenshots/splunk-server/05-splunk-installation-file-transfer-to-vmware.png)

![Transfer Verification](assets/screenshots/splunk-server/06-file-transfer-verify.png)

![Install Splunk](assets/screenshots/splunk-server/07-install-splunk.png)

![Splunk Directory](assets/screenshots/splunk-server/08-splunk-directory-users-and-groups.png)

![Splunk Startup](assets/screenshots/splunk-server/09-splunk-server-start-with-the-user-splunk.png)

![Splunk Web](assets/screenshots/splunk-server/11-target-machine-splunk.png)

</details>

---

# 6. Active Directory Domain Services

After configuring Splunk, I installed and configured Active Directory Domain Services on Windows Server 2022.

Active Directory provides centralized identity management, authentication, authorization, and administration for Windows domain environments.

## 6.1 Installing AD DS

On Windows Server:

1. Open **Server Manager**.
2. Click **Manage**.
3. Select **Add Roles and Features**.
4. Choose **Role-based or feature-based installation**.
5. Select the local Windows Server.
6. Select **Active Directory Domain Services**.
7. Add the required management features.
8. Continue through the installation wizard.
9. Click **Install**.

After installing the AD DS role, the server must be promoted to a Domain Controller.

## 6.2 Promoting the Server

In Server Manager:

1. Click the notification flag.
2. Select **Promote this server to a domain controller**.
3. Choose **Add a new forest**.
4. Enter the domain name:

```text
adlab.local
```

5. Configure the Domain Controller options.
6. Set the Directory Services Restore Mode password.
7. Review DNS configuration.
8. Review the AD DS database paths.
9. Complete prerequisite checks.
10. Click **Install**.

The Domain Controller restarts after promotion.

## 6.3 Active Directory Database

During configuration, the wizard displays the paths used by Active Directory.

One important database file is:

```text
NTDS.dit
```

This database stores important Active Directory information.

From a security perspective, unauthorized access to AD database files can represent a serious credential exposure risk.

SOC analysts should understand why suspicious access to Active Directory database files deserves further investigation.

## 6.4 Verifying the Domain Controller

After promotion, I verified Active Directory management tools were available.

PowerShell commands:

```powershell
Get-ADDomain
```

```powershell
Get-ADForest
```

The domain was configured as:

```text
DNS Domain: adlab.local
NetBIOS Domain: ADLAB
Domain Controller: ADDC01
```

### Active Directory Setup Screenshots

![AD DS Installation Complete](assets/screenshots/active-directory-setup/11-active-directory-installation-complete.png)

<details>
<summary><strong>View all additional Active Directory installation screenshots</strong></summary>

![Add Roles](assets/screenshots/active-directory-setup/01-add-roles-and-features.png)

![Role-Based Installation](assets/screenshots/active-directory-setup/02-role-based-or-feature-based.png)

![Destination Server](assets/screenshots/active-directory-setup/03-select-destination-server.png)

![Server Roles](assets/screenshots/active-directory-setup/04-select-server-roles.png)

![AD DS Selected](assets/screenshots/active-directory-setup/05-active-directory-domain-services-selected.png)

![Post Deployment](assets/screenshots/active-directory-setup/06-ad-post-deployment-configuration.png)

![New Forest](assets/screenshots/active-directory-setup/07-add-a-new-forest.png)

![Forest Password Setup](assets/screenshots/active-directory-setup/08-forest-password-set.png)

![Active Directory Paths](assets/screenshots/active-directory-setup/09-ad-paths.png)

![Prerequisites](assets/screenshots/active-directory-setup/10-prerequisites-check.png)

![Domain Administrator Login](assets/screenshots/active-directory-setup/12-adlab-administrator-logon.png)

</details>

---

# 7. Active Directory Users and Organizational Units

After installing Active Directory, I configured Organizational Units and created user accounts for the environment.

Organizational Units provide a way to organize users, computers, and other Active Directory objects.

## 7.1 Opening Active Directory Users and Computers

Navigate to:

```text
Server Manager
    |
    Tools
    |
    Active Directory Users and Computers
```

I explored the domain's built-in containers and groups to understand how Active Directory organizes objects.

## 7.2 Creating Organizational Units

I created Organizational Units representing departments within the simulated organization.

Examples:

```text
adlab.local
|
+-- IT
|
+-- HR
```

This provides a basic departmental structure.

In enterprise environments, Organizational Units can also support Group Policy application and delegated administrative responsibilities.

## 7.3 Creating Domain Users

Inside the IT Organizational Unit, I created a test user named Jenny Smith.

Additional test users were created for Active Directory authentication and access-control exercises.

### Procedure

1. Right-click the selected Organizational Unit.
2. Select **New → User**.
3. Enter the user's name.
4. Configure the logon name.
5. Set an appropriate test password.
6. Select the applicable account settings.
7. Complete the user creation wizard.

I also reviewed user account properties and group membership.

These exercises helped me understand how domain users are created, managed, and authenticated.

### Active Directory Users Screenshots

![User Group Verification](assets/screenshots/active-directory-users/07-user-group-verify2.png)

<details>
<summary><strong>View additional user-management screenshots</strong></summary>

![AD Users and Computers](assets/screenshots/active-directory-users/01-ad-users-and-computers.png)

![Built-in Groups](assets/screenshots/active-directory-users/02-user-and-computers-builtins-groups.png)

![IT OU](assets/screenshots/active-directory-users/03-it-ou-creation.png)

![User Creation](assets/screenshots/active-directory-users/04-user-creation.png)

![User Password Configuration](assets/screenshots/active-directory-users/05-user-password-set.png)

![User Group](assets/screenshots/active-directory-users/06-user-group-verify.png)

</details>

---

# 8. Joining Windows 10 to Active Directory

The next stage involved joining the Windows 10 target machine to the `adlab.local` domain.

Domain membership allows the workstation to authenticate domain users and communicate with centralized Active Directory services.

## 8.1 Configure DNS

Before joining the domain, I configured the Windows 10 DNS server:

```text
192.168.10.7
```

This points the workstation to the Domain Controller's DNS service.

## 8.2 Join the Domain

On Windows 10:

1. Open **This PC → Properties**.
2. Select **Advanced system settings**.
3. Open the **Computer Name** tab.
4. Click **Change**.
5. Select **Domain**.
6. Enter:

```text
adlab.local
```

7. Provide authorized domain credentials.
8. Confirm the domain join.
9. Restart Windows.

After successfully joining, Windows displayed a welcome message confirming domain membership.

## 8.3 Troubleshooting Domain Join

The initial domain join attempt failed because the workstation was not correctly configured to locate the Domain Controller through DNS.

After changing the preferred DNS server to `192.168.10.7`, the domain join succeeded.

To verify DNS and domain discovery:

```powershell
ipconfig /all
```

```powershell
nltest /dsgetdc:adlab.local
```

The `nltest` command confirms that the Windows workstation can locate the Domain Controller.

### Domain Join Screenshots

![Successful Domain Join](assets/screenshots/domain-join/07-domain-joined.png)

<details>
<summary><strong>View additional domain join screenshots</strong></summary>

![Windows System Properties](assets/screenshots/domain-join/01-w10-system-properties.png)

![Join Domain](assets/screenshots/domain-join/02-w10-join-domain.png)

![Domain Join Error](assets/screenshots/domain-join/03-join-domain-error.png)

![DNS Correction](assets/screenshots/domain-join/04-add-dns-server-as-windows-server-ip.png)

![IP Configuration](assets/screenshots/domain-join/05-ipconfig-all.png)

![Domain Credentials](assets/screenshots/domain-join/06-credentials-for-domain-change.png)

</details>

---

# 9. Sysmon Installation and Configuration

Sysmon (System Monitor) is a Microsoft Sysinternals utility that records detailed Windows system activity.

Sysmon provides additional endpoint visibility beyond traditional Windows Security logs.

This includes:

- Process creation
- Network connections
- File creation
- Registry modifications
- Image and DLL loading
- DNS queries
- Selected process-access activity

Sysmon was installed on:

- Windows 10 Target
- Windows Server 2022 Domain Controller

## 9.1 Download Sysmon

Official download:

https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

After downloading Sysmon, I extracted the package.

## 9.2 Olaf Hartong's Sysmon Modular Configuration

Instead of using only default Sysmon settings, I configured Sysmon using Olaf Hartong's modular configuration.

Repository:

https://github.com/olafhartong/sysmon-modular

The modular configuration contains event-filtering rules designed to collect security-relevant telemetry while reducing unnecessary noise.

I used the balanced configuration appropriate for the installed Sysmon version.

## 9.3 Install Sysmon

With `Sysmon64.exe` and the selected configuration file in the same directory, open PowerShell as Administrator.

```powershell
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

**Explanation:** Installs Sysmon using the specified monitoring configuration.

If Sysmon is already installed, update the configuration instead:

```powershell
.\Sysmon64.exe -c .\sysmonconfig.xml
```

Verify the Sysmon service:

```powershell
Get-Service Sysmon64
```

## 9.4 View Sysmon Logs

Sysmon events are available in:

```text
Event Viewer
    |
    Applications and Services Logs
    |
    Microsoft
    |
    Windows
    |
    Sysmon
    |
    Operational
```

### Important Sysmon Event IDs

| Event ID | Description | Security Relevance |
|---|---|---|
| 1 | Process Creation | Suspicious command execution |
| 3 | Network Connection | Suspicious outbound/inbound activity |
| 7 | Image Loaded | DLL monitoring |
| 10 | Process Access | Possible credential access |
| 11 | File Creation | Suspicious file activity |
| 12 | Registry Object Created/Deleted | Registry monitoring |
| 13 | Registry Value Set | Persistence investigation |
| 22 | DNS Query | Suspicious domain resolution |
| 25 | Process Tampering | Process manipulation |

The actual events recorded depend on the Sysmon configuration and system activity.

### Sysmon Screenshots

![Sysmon Installed](assets/screenshots/sysmon/04-sysmon-installation.png)

<details>
<summary><strong>View additional Sysmon screenshots</strong></summary>

![Olaf Sysmon Link](assets/screenshots/sysmon/01-olaf-configuration-file-link.png)

![Olaf Configuration](assets/screenshots/sysmon/02-olaf-configuration.png)

![Sysmon Extracted](assets/screenshots/sysmon/03-sysmon-file-extracted.png)

</details>

---

# 10. Splunk Universal Forwarder Configuration

Splunk Universal Forwarder is a lightweight agent designed to collect and forward machine data to a Splunk server.

In this project, the Universal Forwarder was installed on both Windows 10 and Windows Server 2022.

The Forwarder sends Windows Event Logs to the Ubuntu Splunk Enterprise server.

## 10.1 Download Universal Forwarder

Official website:

https://www.splunk.com/en_us/download/universal-forwarder.html

I downloaded the Windows x64 installation package.

## 10.2 Universal Forwarder Installation

During installation, I configured the receiving indexer:

```text
Receiving Server: 192.168.10.10
Receiving Port: 9997
```

The deployment server field was left blank because a separate deployment server was not used in this lab.

The documented setup used Local System for access to Windows Security logs. This is suitable for the controlled lab configuration but is more privileged than desirable for many production deployments.

## 10.3 Configure inputs.conf

The Universal Forwarder uses configuration files to determine which data sources to monitor.

The configuration file was created at:

```text
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

### Configuration

```ini
[WinEventLog://Application]
index = endpoint
disabled = 0

[WinEventLog://Security]
index = endpoint
disabled = 0

[WinEventLog://System]
index = endpoint
disabled = 0

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = 0
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

### Configuration Explanation

**Application:** Collects application events and errors.

**Security:** Collects authentication, authorization, account activity, and other security-related events.

**System:** Collects Windows operating system and service events.

**Sysmon:** Collects detailed endpoint telemetry generated by Sysmon.

**index = endpoint:** Sends the events to the dedicated Splunk index.

**renderXml = true:** Preserves the Sysmon events in XML format, which is useful for detailed field extraction.

## 10.4 Restart Universal Forwarder

After modifying `inputs.conf`:

```powershell
Restart-Service SplunkForwarder
```

Verify the service:

```powershell
Get-Service SplunkForwarder
```

## 10.5 Create the endpoint Index

In Splunk Web:

1. Open **Settings**.
2. Select **Indexes**.
3. Click **New Index**.
4. Enter:

```text
endpoint
```

5. Save the new index.

This index stores the Windows and Sysmon data collected from the endpoints.

## 10.6 Configure Receiving Port

Navigate to:

```text
Settings
    |
Forwarding and Receiving
    |
Configure Receiving
    |
New Receiving Port
```

Enter:

```text
9997
```

This allows Splunk Enterprise to receive data from the Universal Forwarders.

## 10.7 Verify Forwarder Connectivity

On Windows:

```powershell
Test-NetConnection 192.168.10.10 -Port 9997
```

Expected result:

```text
TcpTestSucceeded : True
```

## 10.8 Verify Data in Splunk

In Search & Reporting:

```spl
index=endpoint
| stats count by host source
| sort -count
```

This displays the event sources associated with each monitored host.

The observed environment contained event data from:

```text
ADDC01
target-PC
```

The collected sources included:

```text
WinEventLog:Application
WinEventLog:System
WinEventLog:Security
XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

### Universal Forwarder Screenshots

![Splunk Event Sources](assets/screenshots/splunk-forwarder/15-splunk-index-source-check.png)

<details>
<summary><strong>View additional Splunk Forwarder screenshots</strong></summary>

![Download](assets/screenshots/splunk-forwarder/01-splunk-universal-forwarder-image-download.png)

![Setup Start](assets/screenshots/splunk-forwarder/02-splunk-universal-forwarder-setup-start.png)

![Setup](assets/screenshots/splunk-forwarder/03-splunk-universal-forwarder-setup.png)

![Indexer Address](assets/screenshots/splunk-forwarder/04-splunk-universal-forwarder-splunk-server-ip.png)

![Administrator Notepad](assets/screenshots/splunk-forwarder/05-open-notepad-as-admin.png)

![Inputs Configuration](assets/screenshots/splunk-forwarder/06-save-the-inputs-conf-file.png)

![Forwarder Service](assets/screenshots/splunk-forwarder/07-splunk-fordwarder-service-check.png)

![Forwarder Service Verification](assets/screenshots/splunk-forwarder/08-splunk-fordwarder-service1.png)

![Forwarder Service Details](assets/screenshots/splunk-forwarder/09-splunk-fordwarder-service2.png)

![New Index](assets/screenshots/splunk-forwarder/10-create-a-new-index.png)

![Forwarding and Receiving](assets/screenshots/splunk-forwarder/11-forwarding-and-receiving.png)

![Receiving Port](assets/screenshots/splunk-forwarder/12-forwarding-and-receiving-add-the-port-number.png)

![Host Search](assets/screenshots/splunk-forwarder/13-splunk-search-query-host-check.png)

![Updated Host Search](assets/screenshots/splunk-forwarder/14-splunk-query-host-update.png)

</details>

---

# 11. Kali Linux Network Configuration

After configuring the monitoring infrastructure, I prepared Kali Linux for authorized security testing.

Kali was assigned a static IP address on the same virtual network.

### Network Configuration

```text
IP Address: 192.168.10.250
Prefix: /24
Gateway: 192.168.10.2
DNS: 192.168.10.7
```

During initial configuration, a public DNS resolver was used for internet testing. The Domain Controller's DNS address was later used for Active Directory name resolution.

## 11.1 Configure the Kali Network

Open the network settings and navigate to:

```text
Advanced Network Configuration
    |
Wired Connection 1
    |
IPv4 Settings
    |
Method: Manual
```

Enter the network information and save.

## 11.2 Verify Connectivity

```bash
ip a
```

```bash
ip route
```

Test Splunk connectivity:

```bash
ping -c 4 192.168.10.10
```

Test Windows 10 connectivity:

```bash
ping -c 4 192.168.10.100
```

Test Domain Controller connectivity:

```bash
ping -c 4 192.168.10.7
```

## 11.3 Troubleshooting ICMP

I initially encountered a situation where Windows 10 could ping Kali, but Kali could not ping Windows 10.

The issue involved Windows Firewall's inbound ICMP rules.

I kept Windows Firewall enabled and allowed ICMP requests from the Kali lab IP address.

This helped demonstrate that successful communication in one direction does not necessarily mean the opposite direction is allowed.

I also verified RDP separately because ICMP connectivity and TCP port connectivity are different.

### Kali Network Screenshots

![Kali Linux IP](assets/screenshots/kali-network/05-kali-linux-ip-a2.png)

<details>
<summary><strong>View additional Kali network screenshots</strong></summary>

![Kali Network](assets/screenshots/kali-network/01-kali-linux-network.png)

![Kali Connection](assets/screenshots/kali-network/02-kali-linux-network-connection.png)

![Kali Static IP](assets/screenshots/kali-network/03-kali-linux-network-configuration.png)

![Kali IP Verification](assets/screenshots/kali-network/04-kali-linux-ip-a1.png)

![Kali Ping](assets/screenshots/kali-network/06-kali-linux-ping.png)

</details>

---

# 12. RDP Authentication Testing with NetExec

After confirming network connectivity and log forwarding, I performed controlled RDP authentication testing against the Windows 10 target machine.

This exercise was intended to generate authentication events that could be monitored and investigated in Splunk.

## 12.1 Enable Remote Desktop

On Windows 10:

1. Open System Properties.
2. Select the Remote tab.
3. Enable Remote Desktop.
4. Keep Network Level Authentication enabled.
5. Select authorized users.
6. Add the designated test domain account.

The Windows 10 target was configured to accept the lab's authorized domain user for Remote Desktop testing.

## 12.2 Verify RDP Connectivity

From Kali Linux:

```bash
nmap -Pn -p 3389 192.168.10.100
```

The scan showed that TCP port 3389 was open.

This confirmed that the RDP service was reachable from Kali Linux.

An open port alone does not establish that authentication will succeed.

## 12.3 Why NetExec Was Used

The project initially followed a tutorial that demonstrated RDP authentication testing using Crowbar.

However, the Kali Linux 2026.2 environment included FreeRDP 3, while Crowbar's older integration expected different command-line behavior.

This caused parsing problems and false-positive authentication results.

After troubleshooting, I selected NetExec as an alternative because it correctly distinguished rejected credentials from accepted credentials in the current lab environment.

## 12.4 Installing NetExec

On Kali Linux:

```bash
sudo apt update
sudo apt install netexec -y
```

Verify:

```bash
nxc --version
```

The lab used NetExec version 1.5.1.

## 12.5 Preparing the Controlled Test

I used a small lab password list and a disposable Active Directory account.

The purpose was to test the detection pipeline without performing large-scale password testing or affecting production systems.

The wordlist was stored locally in the Kali project directory and excluded from the published repository.

Before testing, I verified the target address, account permissions, RDP availability, and account-lockout considerations.

## 12.6 Running the Authentication Test

The command used in the documented lab was:

```bash
nxc -t 1 --jitter 3 rdp 192.168.10.100 -d ADLAB -u tsmith -p passwords.txt
```

### Command Explanation

| Parameter | Purpose |
|---|---|
| `nxc` | Execute NetExec |
| `-t 1` | Use one concurrent worker |
| `--jitter 3` | Add spacing between attempts |
| `rdp` | Use the RDP protocol |
| `192.168.10.100` | Windows 10 target |
| `-d ADLAB` | Specify Active Directory domain |
| `-u tsmith` | Specify the disposable test account |
| `-p passwords.txt` | Use the lab password list |

The command was executed only against the authorized Windows 10 VM.

## 12.7 Authentication Results

NetExec displayed rejected credentials using:

```text
STATUS_LOGON_FAILURE
```

When the deliberately valid test credential was evaluated, NetExec displayed a successful authentication indicator.

The successful result was accompanied by a `Pwn3d!` indicator.

This indicates that NetExec recognized successful authentication and reported elevated access characteristics for the account. It does not by itself prove that a complete RDP desktop session was opened.

### NetExec Authentication Test

![NetExec Authentication Test](assets/screenshots/authentication-testing/10-netexec-password-brute-force-attack.png)

> **Privacy note:** The published screenshot should have the valid test password covered before uploading to a public GitHub repository.

## 12.8 Security Significance

Repeated failed authentication attempts can indicate:

- Password guessing
- Misconfigured services
- Credential misuse
- Unauthorized access attempts
- Account lockout activity

A successful credential validation following multiple failures is an especially useful sequence for SOC analysts to investigate.

However, the events must be correlated before concluding that an account was compromised.

### Authentication Testing Screenshots

<details>
<summary><strong>View additional authentication testing screenshots</strong></summary>

![NetExec Version](assets/screenshots/authentication-testing/01-netexec-version.png)

![NetExec Installation](assets/screenshots/authentication-testing/02-kali-linux-apt-update-and-install-netexec.png)

![NetExec Help](assets/screenshots/authentication-testing/03-netexec-help.png)

![Password File Permissions](assets/screenshots/authentication-testing/04-password-file-permissions.png)

![Wordlist Preparation](assets/screenshots/authentication-testing/05-unzip-the-password-file.png)

![Small Wordlist](assets/screenshots/authentication-testing/06-get-first-20-password.png)

![Wordlist Verification](assets/screenshots/authentication-testing/07-new-file-read.png)

![Remote Desktop](assets/screenshots/authentication-testing/08-allow-remote-connection.png)

![Remote Desktop Users](assets/screenshots/authentication-testing/09-add-users-in-remote-connection.png)

![Event ID 4625](assets/screenshots/authentication-testing/11-eventid-4625.png)

![Splunk Query After Test](assets/screenshots/authentication-testing/12-splunk-query-after-attack.png)

![Attacker IP Investigation](assets/screenshots/authentication-testing/13-splunk-check-ip-of-the-attacker.png)

</details>

---

# 13. Investigating Authentication Events in Splunk

After conducting the controlled RDP authentication tests, I used Splunk Enterprise to investigate the resulting Windows security events.

The Windows 10 workstation and Domain Controller both forwarded logs to the `endpoint` index.

## 13.1 Verify Windows Event Sources

```spl
index=endpoint
| stats count by host source
| sort -count
```

This confirms which hosts are sending data and which event sources are available.

## 13.2 Investigate Failed Logons

Windows Security Event ID 4625 indicates that an account failed to log on.

```spl
index=endpoint source="WinEventLog:Security" EventCode=4625
| table _time host _raw
| sort -_time
```

### Explanation

- `index=endpoint` selects the monitoring index.
- `source="WinEventLog:Security"` limits the search to Windows Security logs.
- `EventCode=4625` filters for failed logons.
- `table` displays selected event information.
- `sort -_time` places the newest events first.

The saved project evidence includes a Splunk search showing **60 Event ID 4625 occurrences** in a filtered result.

This is the count shown in that captured search, not a claim about the total number of failures throughout the entire project.

## 13.3 Investigate Active Directory Credential Validation

Windows Event ID 4776 records NTLM credential validation activity.

The Domain Controller recorded authentication activity involving the test account and the Kali workstation.

Important error codes:

| Error Code | Meaning |
|---|---|
| `0x0` | Successful credential validation |
| `0xC000006A` | Incorrect password |
| `0xC0000234` | Account locked out |

### SPL Query

```spl
index=endpoint host=ADDC01 source="WinEventLog:Security" EventCode=4776
| table _time host _raw
| sort -_time
```

This displays the NTLM validation events collected from the Domain Controller.

## 13.4 Extract Authentication Result Codes

To distinguish successful and failed authentication, I used SPL field extraction.

```spl
index=endpoint host=ADDC01 source="WinEventLog:Security" EventCode=4776 "tsmith"
| rex field=_raw "Error Code:\s+(?<AuthStatus>0x[0-9A-Fa-f]+)"
| eval Result=case(
    lower(AuthStatus)="0x0", "Success",
    lower(AuthStatus)="0xc000006a", "Incorrect Password",
    true(), "Other"
)
| stats count by Result
```

### Explanation

**rex:** Extracts the NTLM error code from the raw event.

**eval:** Creates a readable result category.

**case:** Uses conditions to classify the event as a success, incorrect password, or other outcome.

**stats:** Counts events in each category.

This helped identify successful and failed NTLM credential validation in Splunk.

## 13.5 Investigating RDP Logons

A successful NTLM validation does not necessarily mean a full RDP desktop session was established.

To investigate Windows remote-interactive logons, I can search for Event ID 4624 with Logon Type 10:

```spl
index=endpoint host="target-PC" source="WinEventLog:Security" EventCode=4624
| rex field=_raw "Logon Type:\s+(?<LogonType>\d+)"
| where LogonType="10"
| table _time host LogonType _raw
| sort -_time
```

This helps distinguish complete remote-interactive logons from credential validation alone.

## 13.6 Investigating Sysmon Events

One important issue I discovered was that some Sysmon XML events did not expose `EventCode` as a normal extracted field.

An initial search using:

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| stats count by host EventCode
```

did not produce the expected statistics table.

The events were present, but the field extraction was different.

To solve this, I extracted the event identifier directly from the XML.

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| stats count by host SysmonEventID
| sort -count
```

This successfully displayed the different Sysmon Event IDs collected from both Windows machines.

The output included:

- Process creation
- Network connections
- Image loading
- File creation
- Registry modifications
- DNS queries

### Why This Matters

A log being successfully received does not automatically mean every field has been extracted correctly.

SOC analysts must understand the underlying data format and adapt their searches accordingly.

---

# 14. Atomic Red Team Attack Simulation

After completing the authentication monitoring exercises, I used Atomic Red Team to simulate additional security-relevant activity on the Windows 10 target.

Atomic Red Team provides individual security tests based on techniques from the MITRE ATT&CK framework.

These tests allow security professionals to evaluate whether endpoint monitoring tools record behavior associated with known adversary techniques.

## 14.1 Why Atomic Red Team?

Atomic Red Team was included to:

- Generate known security-relevant activities.
- Practice recognizing MITRE ATT&CK techniques.
- Validate Windows and Sysmon logging.
- Investigate process and registry telemetry.
- Learn how offensive simulations support defensive investigations.

The tests were executed on the disposable Windows 10 VM.

## 14.2 Installing Atomic Red Team

The project used Red Canary's Invoke-AtomicRedTeam framework.

Repository:

https://github.com/redcanaryco/invoke-atomicredteam

The documented installation procedure included:

```powershell
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
```

Then:

```powershell
Install-AtomicRedTeam -getAtomics
```

Scripts retrieved from the internet should be reviewed before execution, and tests should be run only in an authorized disposable environment.

The original lab notes also include an experimental Microsoft Defender whole-drive exclusion screenshot. That weakens endpoint protection and is not recommended as a normal procedure. Security controls should remain enabled wherever possible.

## 14.3 MITRE ATT&CK Technique T1136.001

**Technique:** Create Account — Local Account

MITRE ATT&CK ID:

```text
T1136.001
```

This technique involves the creation of local user accounts.

Attackers can abuse account creation to maintain access or establish persistence.

The documented lab command was:

```powershell
Invoke-AtomicTest T1136.001
```

After performing the selected test, I searched for related activity in Splunk.

```spl
index=endpoint "NewLocalUser"
```

The documented Splunk search returned **14 raw events** associated with the activity.

This demonstrated that the monitoring pipeline captured activity related to the Atomic Red Team test.

A full investigation would examine the account name, executing process, user context, timestamps, and other event information.

## 14.4 MITRE ATT&CK Technique T1059.001

**Technique:** Command and Scripting Interpreter — PowerShell

MITRE ATT&CK ID:

```text
T1059.001
```

PowerShell is a legitimate Windows administration tool, but it is also used by attackers to execute commands and scripts.

The documented lab command was:

```powershell
Invoke-AtomicTest T1059.001
```

After the test, I investigated the relevant events in Splunk.

The recorded results included Sysmon registry-related telemetry referencing the Background Activity Moderator (BAM) UserSettings path.

This observation demonstrates available endpoint telemetry after test execution. It does not mean every registry modification is malicious or that the observed BAM activity independently proves persistence.

## 14.5 Atomic Red Team Cleanup

Before running additional Atomic tests, I recommend reviewing the details and cleanup actions.

Example:

```powershell
Invoke-AtomicTest T1136.001 -ShowDetails
```

```powershell
Invoke-AtomicTest T1059.001 -ShowDetails
```

After the selected tests, cleanup should be performed when supported, or the virtual machine should be restored to a known-good snapshot.

### Atomic Red Team Evidence

![Atomic Red Team Splunk Investigation](assets/screenshots/atomic-red-team/08-splunk-check-after-atomic-red-team-run.png)

<details>
<summary><strong>View additional Atomic Red Team screenshots</strong></summary>

![Execution Policy](assets/screenshots/atomic-red-team/01-set-exectution-policy.png)

![Experimental Defender Configuration](assets/screenshots/atomic-red-team/02-executions-add-c-drive.png)

![Atomic Installation](assets/screenshots/atomic-red-team/03-install-atomic-red-team.png)

![Atomic Directory](assets/screenshots/atomic-red-team/04-atomic-red-team-folder.png)

![Atomic Test Execution](assets/screenshots/atomic-red-team/05-atomic-red-team-folder-run.png)

![MITRE ATTACK](assets/screenshots/atomic-red-team/06-mitre-att-ck.png)

![ATTACK Techniques](assets/screenshots/atomic-red-team/07-testing-differnt-techniques-id.png)

![PowerShell Atomic Investigation](assets/screenshots/atomic-red-team/09-splunk-check-after-running-powershell-atomic-red-team.png)

</details>

---

# 15. Splunk Detection Queries

The following SPL queries demonstrate how the collected Windows and Sysmon events can be used for security monitoring and threat hunting.

These are investigation and detection-engineering examples based on the data sources configured in the lab. They should not all be interpreted as saved production alerts.

## Detection 1: Count Failed Windows Logons

```spl
index=endpoint source="WinEventLog:Security" EventCode=4625
| stats count as FailedLogons by host
| sort -FailedLogons
```

**Purpose:** Identify Windows hosts recording failed login activity.

This helps SOC analysts locate systems experiencing unusual authentication failures.

## Detection 2: Monitor NTLM Incorrect Passwords

```spl
index=endpoint host=ADDC01 source="WinEventLog:Security" EventCode=4776
| rex field=_raw "Logon Account:\s+(?<Account>\S+)"
| rex field=_raw "Source Workstation:\s+(?<Workstation>[^\r\n]+)"
| rex field=_raw "Error Code:\s+(?<AuthStatus>0x[0-9A-Fa-f]+)"
| where lower(AuthStatus)="0xc000006a"
| stats count as FailedAttempts by Account Workstation
| sort -FailedAttempts
```

**Purpose:** Identify user accounts and workstations associated with rejected NTLM credentials.

## Detection 3: Failed Authentication Followed by Success

```spl
index=endpoint host=ADDC01 source="WinEventLog:Security" EventCode=4776
| rex field=_raw "Logon Account:\s+(?<Account>\S+)"
| rex field=_raw "Source Workstation:\s+(?<Workstation>[^\r\n]+)"
| rex field=_raw "Error Code:\s+(?<AuthStatus>0x[0-9A-Fa-f]+)"
| eval Result=case(
    lower(AuthStatus)="0xc000006a", "Failure",
    lower(AuthStatus)="0x0", "Success"
)
| where isnotnull(Result) AND isnotnull(Account) AND isnotnull(Workstation)
| sort 0 _time
| streamstats time_window=10m count(eval(Result="Failure")) as FailedAttempts by Account Workstation
| where Result="Success" AND FailedAttempts>=5
| table _time host Account Workstation FailedAttempts Result
| sort -_time
```

**Purpose:** Identify successful NTLM credential validation preceded by repeated incorrect-password attempts from the same account and workstation within a 10-minute window.

This pattern can be consistent with password guessing followed by successful authentication.

The query provides a lead for investigation, not definitive proof of compromise.

## Detection 4: Sysmon Process Creation

```spl
index=endpoint host="target-PC" source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| where SysmonEventID="1"
| table _time host _raw
| sort -_time
```

**Purpose:** Investigate process creation events captured by Sysmon.

This is useful for reviewing executed commands, parent processes, and process relationships.

## Detection 5: Sysmon Event Distribution

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| stats count by host SysmonEventID
| sort -count
```

**Purpose:** Identify which Sysmon event types generate the most telemetry.

This can help tune event filtering and evaluate storage requirements.

## Detection 6: Windows Authentication Timeline

```spl
index=endpoint source="WinEventLog:Security"
(EventCode=4624 OR EventCode=4625 OR EventCode=4771 OR EventCode=4776 OR EventCode=4740)
| table _time host EventCode _raw
| sort -_time
```

**Purpose:** Review successful logons, failed logons, Kerberos authentication failures, NTLM credential validation, and account lockouts across monitored systems.

This helps build an investigation timeline.

## Detection 7: Specific Account Investigation

```spl
index=endpoint source="WinEventLog:Security" "tsmith"
| table _time host EventCode _raw
| sort -_time
```

**Purpose:** Investigate authentication activity associated with an individual lab account.

## Detection 8: Review Splunk Data Sources

```spl
index=endpoint
| stats count by host source
| sort -count
```

**Purpose:** Verify that the monitoring pipeline continues receiving data from both Windows machines.

---

# 16. Challenges and Troubleshooting

Throughout the project, I encountered several technical issues that required troubleshooting.

These problems provided practical opportunities to improve my understanding of system administration, networking, Windows authentication, and SIEM configuration.

## 16.1 VMware NAT Gateway Conflict

**Problem:** Ubuntu could not communicate with external networks.

**Cause:** The VMware host adapter and NAT gateway were both configured to use `192.168.10.1`.

**Solution:** Changed the NAT gateway to `192.168.10.2` and updated Ubuntu's default route.

**Lesson:** Each network interface and gateway must have a unique, valid IP address.

## 16.2 Splunk Storage Issue

**Problem:** Splunk installation failed with a no-space-left error.

**Cause:** Ubuntu's root filesystem had insufficient available capacity.

**Solution:** Expanded the VMware virtual disk and extended Ubuntu's partition and LVM filesystem.

**Lesson:** Increasing virtual disk capacity does not automatically expand guest partitions.

## 16.3 Active Directory Domain Join Failure

**Problem:** Windows 10 could not join the domain.

**Cause:** The workstation was not using the Domain Controller for DNS resolution.

**Solution:** Configured `192.168.10.7` as the preferred DNS server.

**Lesson:** Active Directory depends on correct DNS configuration.

## 16.4 Kali Cannot Ping Windows 10

**Problem:** Windows 10 could ping Kali, but Kali could not ping Windows 10.

**Cause:** Inbound ICMP requests were not permitted by the Windows Firewall configuration.

**Solution:** Allowed the authorized Kali host to send ICMP requests without disabling Windows Firewall.

**Lesson:** Firewall policies can allow outbound traffic while restricting inbound traffic.

## 16.5 Splunk Searches Returned No Events

**Problem:** Initial searches using `index=main` returned no expected Windows authentication events.

**Cause:** The Windows logs were stored in the dedicated `endpoint` index.

**Solution:** Updated searches to use:

```spl
index=endpoint
```

**Lesson:** SOC analysts must understand the actual index and source configuration before writing detection queries.

## 16.6 Sysmon Field Extraction

**Problem:** Sysmon events appeared in Splunk, but grouping by `EventCode` returned no statistics.

**Cause:** The events were ingested in XML format and required additional event ID extraction.

**Solution:** Used SPL `rex` to extract `<EventID>` from `_raw`.

**Lesson:** Event collection and field extraction are separate stages of the log-processing pipeline.

## 16.7 Crowbar and FreeRDP Compatibility

**Problem:** Crowbar generated command-line errors and later reported false authentication successes.

**Cause:** Crowbar's older FreeRDP integration was incompatible with the newer FreeRDP version installed on Kali Linux 2026.2.

**Solution:** Switched to NetExec for controlled authentication testing.

**Lesson:** Security testing tools can become unreliable when their dependencies change. Their results should be verified independently against system logs.

## 16.8 Kali Linux Mouse Cursor Issue

**Problem:** The cursor was invisible inside the Kali VM, even though mouse clicking still worked.

**Cause:** A guest graphics and cursor-rendering issue.

**Solution:** Used an Xorg software cursor configuration to correct rendering.

**Lesson:** Not every input issue is a hardware problem; virtualization display integration can also affect usability.

---

# 17. Project Results

The final environment successfully demonstrated centralized monitoring of Windows and Active Directory activity.

## Confirmed Outcomes

| Objective | Result |
|---|---|
| Configure VMware virtual network | Completed |
| Deploy Windows 10 | Completed |
| Deploy Windows Server 2022 | Completed |
| Install Ubuntu Splunk Server | Completed |
| Configure Active Directory | Completed |
| Create test users and OUs | Completed |
| Join Windows 10 to the domain | Completed |
| Install Sysmon | Completed |
| Configure Splunk Forwarder | Completed |
| Create endpoint index | Completed |
| Forward Windows logs | Completed |
| Receive Sysmon telemetry | Completed |
| Generate authorized RDP authentication activity | Completed |
| Investigate failed authentications | Completed |
| Identify accepted test credentials | Completed |
| Run selected Atomic Red Team tests | Documented |
| Investigate security events in Splunk | Completed |

## Authentication Analysis

The Domain Controller recorded NTLM credential validation events associated with the test account.

The Windows workstation recorded failed logon events.

Splunk allowed me to investigate and compare the activity across the monitored systems.

The collected events included:

- Windows Security Event ID 4625
- Windows Security Event ID 4624
- NTLM credential validation Event ID 4776
- Sysmon process creation events
- Sysmon file creation events
- Sysmon registry activity

The project demonstrated how endpoint and Domain Controller telemetry can be combined to investigate potentially suspicious activity.

## Atomic Red Team Results

Selected Atomic Red Team tests generated Windows endpoint activity that could be examined using Splunk.

The account-creation test produced events associated with local account management, while the PowerShell-related test provided additional endpoint telemetry.

These observations reinforced the importance of using realistic, controlled simulations to validate monitoring coverage.

---

# 18. Lessons Learned

## 1. Networking Is the Foundation of a Functional Lab

Configuring static IP addresses, subnet masks, default gateways, and DNS correctly was essential to establishing communication between the virtual machines.

I learned how VMware NAT works and why a small gateway configuration mistake can prevent connectivity.

## 2. Active Directory Relies on Proper DNS Configuration

During domain joining, I discovered that even when two machines can ping each other, Active Directory may still fail if the workstation cannot resolve the domain through the correct DNS server.

Correcting the DNS configuration allowed Windows 10 to join the domain successfully.

## 3. Centralized Security Monitoring Requires Multiple Components

Installing Splunk alone does not automatically provide visibility into Windows endpoints.

Sysmon, Universal Forwarder, event inputs, receiving ports, and Splunk indexes must all work together to establish a complete monitoring pipeline.

## 4. Security Logs Must Be Interpreted Correctly

I learned that Windows authentication events have different meanings depending on their Event ID, source, logon type, and status codes.

A successful NTLM credential validation does not always mean an interactive desktop session was established.

Correlating multiple events is necessary to understand what happened.

## 5. Troubleshooting Is an Important Cybersecurity Skill

This project required troubleshooting Windows networking, Linux storage, Splunk field extraction, authentication auditing, and tool compatibility.

Working through these issues improved my ability to investigate problems systematically instead of relying on assumptions.

## Conclusion

This Active Directory SOC HomeLab provided hands-on experience building a virtual enterprise environment and implementing security monitoring across multiple systems.

By configuring Active Directory, Splunk Enterprise, Sysmon, and Splunk Universal Forwarder, I developed a better understanding of how security telemetry is generated, collected, and analyzed.

Using NetExec and Atomic Red Team allowed me to simulate selected security-relevant activities and examine how those behaviors appeared in Windows logs and Splunk.

The project strengthened my understanding of network administration, Windows authentication, log analysis, threat detection, and the investigative workflow followed by SOC analysts.

It also established a reusable laboratory environment for future cybersecurity learning and detection-engineering exercises.

---

# 19. Security Considerations

This environment was intentionally created for cybersecurity education and testing.

Several security practices are important when maintaining or reproducing the lab:

1. Perform authentication testing only against systems and accounts you own or are explicitly authorized to assess.
2. Use disposable test credentials and avoid reusing passwords from personal or production accounts.
3. Keep Windows Firewall enabled and permit only necessary connections.
4. Restrict access to Splunk Web and receiving ports.
5. Take VMware snapshots before running tests that modify the operating system.
6. Review Atomic Red Team cleanup procedures.
7. Avoid disabling antivirus or EDR protection globally.
8. Monitor Splunk storage utilization and configure appropriate data retention.
9. Do not publish actual passwords, tokens, credentials, or security database files.
10. Review screenshots for sensitive information before uploading them to GitHub.

The project is intended to demonstrate controlled security testing and defensive monitoring, not unauthorized access or real-world exploitation.

---

# 20. References

The following resources were used for software downloads, technical configuration, and additional background learning.

### Microsoft

- [Windows 10 Installation Media](https://www.microsoft.com/software-download/windows10)
- [Windows Server 2022 Evaluation](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022)
- [Active Directory Domain Services Documentation](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/)
- [Sysmon — Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Windows Security Auditing](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/)

### VMware

- [VMware Workstation Documentation](https://techdocs.broadcom.com/)

### Ubuntu

- [Ubuntu Server](https://ubuntu.com/server)
- [Ubuntu Server Documentation](https://documentation.ubuntu.com/server/)

### Splunk

- [Splunk Enterprise](https://www.splunk.com/)
- [Splunk Documentation](https://docs.splunk.com/)
- [Splunk Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html)
- [Splunk Search Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/)

### Security Testing and Monitoring

- [Kali Linux](https://www.kali.org/)
- [Olaf Hartong — Sysmon Modular](https://github.com/olafhartong/sysmon-modular)
- [NetExec Documentation](https://www.netexec.wiki/)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [Invoke-AtomicRedTeam](https://github.com/redcanaryco/invoke-atomicredteam)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

## Documentation and AI Transparency

**AI Transparency:** This GitHub README was organized and written with assistance from ChatGPT using my original project documentation and screenshots.

The virtual machines, Active Directory configurations, Splunk environment, monitoring setup, and security testing described in this repository are based on the hands-on work performed during my lab.

AI assistance was used to improve the organization, clarity, technical explanations, and readability of the documentation.

The project is maintained as a personal cybersecurity learning and portfolio resource.

---

## Final Note

This project represents a practical learning environment focused on Windows administration, centralized security monitoring, endpoint telemetry, authentication analysis, and controlled purple-team testing.

The skills developed through this lab are relevant to Security Operations Center (SOC) Analyst, Cybersecurity Analyst, and other defensive security roles.

**All testing was performed within an authorized virtual HomeLab environment.**
