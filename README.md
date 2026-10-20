
# Active Directory SOC HomeLab

### Building and Monitoring an Active Directory Environment Using VMware, Splunk, Sysmon, Kali Linux, NetExec, and Atomic Red Team

![Platform](https://img.shields.io/badge/Virtualization-VMware_Workstation-blue)
![Operating System](https://img.shields.io/badge/Server-Windows_Server_2022-blue)
![SIEM](https://img.shields.io/badge/SIEM-Splunk_Enterprise-orange)
![Monitoring](https://img.shields.io/badge/Endpoint-Sysmon-green)
![Security](https://img.shields.io/badge/Security-Purple_Teaming-purple)

---

## Index

1. [Introduction](#introduction)
2. [Network Architecture](#network-architecture)
3. [Setting Up the Virtual Machines](#setting-up-the-virtual-machines)
   - [Software Requirements](#software-requirements)
   - [Installing Windows 10](#installing-windows-10)
   - [Installing Kali Linux](#installing-kali-linux)
   - [Installing Windows Server 2022](#installing-windows-server-2022)
   - [Installing Ubuntu Server](#installing-ubuntu-server)
4. [Configuring the VMware Network](#configuring-the-vmware-network)
5. [Installing and Configuring Splunk Enterprise](#installing-and-configuring-splunk-enterprise)
6. [Installing Splunk Universal Forwarder and Sysmon](#installing-splunk-universal-forwarder-and-sysmon)
7. [Installing and Configuring Active Directory](#installing-and-configuring-active-directory)
8. [Creating Organizational Units and Users](#creating-organizational-units-and-users)
9. [Joining Windows 10 to Active Directory](#joining-windows-10-to-active-directory)
10. [Configuring Kali Linux for Security Testing](#configuring-kali-linux-for-security-testing)
11. [RDP Authentication Testing Using NetExec](#rdp-authentication-testing-using-netexec)
12. [Investigating Authentication Events in Splunk](#investigating-authentication-events-in-splunk)
13. [Testing with Atomic Red Team](#testing-with-atomic-red-team)
14. [Splunk Detection Queries](#splunk-detection-queries)
15. [Challenges and Troubleshooting](#challenges-and-troubleshooting)
16. [Project Results](#project-results)
17. [Lessons Learned](#lessons-learned)
18. [Conclusion](#conclusion)
19. [References](#references)

---

## Introduction

The purpose of this project was to build a virtual Active Directory environment and implement centralized security monitoring using Splunk Enterprise and Microsoft Sysmon.

I wanted to gain practical experience with Windows system administration, Active Directory, network configuration, SIEM technologies, endpoint monitoring, and security event investigation.

The project also provided an opportunity to understand how security analysts detect and investigate suspicious activity within an organization's network.

To accomplish this, I created four virtual machines using VMware Workstation Pro.

- **Windows Server 2022:** Hosts Active Directory Domain Services and DNS.
- **Windows 10:** Acts as a domain-joined workstation and monitored endpoint.
- **Ubuntu Server:** Hosts Splunk Enterprise for centralized log management.
- **Kali Linux:** Performs authorized security testing against the Windows workstation.

After establishing the virtual environment, I installed Splunk Universal Forwarder and Sysmon on the Windows machines to collect Windows Security, System, Application, and endpoint telemetry.

I then performed controlled RDP authentication testing from Kali Linux using NetExec and investigated the resulting authentication events in Splunk.

Finally, I used Atomic Red Team to simulate selected MITRE ATT&CK techniques and analyze the endpoint activity they generated.

The goal was not simply to run security tools, but to understand how attacker behavior can be observed through Windows logs and analyzed from a defensive security perspective.

> **Disclaimer:** This project was conducted entirely within an authorized virtual HomeLab. All attack simulations and authentication tests were performed against systems and accounts created specifically for educational purposes.

---

## Network Architecture

Before creating the virtual machines, I designed a network diagram to understand how the different components would communicate.

The lab consists of one Domain Controller, one domain-joined Windows workstation, one Splunk Enterprise server, and one Kali Linux security testing machine.

![Active Directory Lab Network Diagram](images/Lab%20Logical%20Diagram.jpeg)

### Network Configuration

| Component | IP Address | Description |
|---|---|---|
| Network | 192.168.10.0/24 | VMware NAT network |
| VMware NAT Gateway | 192.168.10.2 | Provides outbound connectivity |
| VMware Host Adapter | 192.168.10.1 | Host-side virtual interface |
| Splunk Server | 192.168.10.10 | Centralized SIEM |
| Active Directory Server | 192.168.10.7 | Domain Controller and DNS |
| Windows 10 Target | 192.168.10.100 | Domain-joined workstation |
| Kali Linux | 192.168.10.250 | Security testing machine |

### Active Directory Information

```text
Domain Name: adlab.local
NetBIOS Domain: ADLAB
Domain Controller: ADDC01
Windows Workstation: target-PC
```

All four machines were connected to the same VMware NAT network.

The Domain Controller manages authentication and domain services, while the Windows 10 workstation represents an employee computer.

The Ubuntu server receives Windows event logs and Sysmon telemetry through Splunk Universal Forwarder.

Kali Linux is used to generate controlled security activity against the Windows workstation.

### Security Monitoring Process

```text
                 KALI LINUX
                192.168.10.250
                       |
                       | Controlled Attack Simulation
                       |
                       v
                WINDOWS 10 TARGET
                192.168.10.100
                       |
                  Windows Events
                  Sysmon Events
                       |
                       v
               SPLUNK UNIVERSAL
                   FORWARDER
                       |
                       | TCP Port 9997
                       |
                       v
                SPLUNK ENTERPRISE
                 192.168.10.10
                       |
                       v
                SECURITY ANALYSIS
                       |
                       v
            Threat Detection and Investigation
```

The Domain Controller also forwards security events to Splunk, allowing authentication requests to be investigated across multiple machines.

---

# Setting Up the Virtual Machines

## Software Requirements

The following software and operating systems were used during the project.

| Software | Purpose |
|---|---|
| VMware Workstation Pro | Virtualization |
| Windows 10 Pro | Target workstation |
| Windows Server 2022 | Active Directory Domain Controller |
| Ubuntu Server | Splunk Enterprise host |
| Kali Linux 2026.2 | Security testing |
| Splunk Enterprise | Centralized SIEM |
| Splunk Universal Forwarder | Log forwarding |
| Microsoft Sysmon | Endpoint telemetry |
| Olaf Hartong Sysmon Modular | Sysmon configuration |
| NetExec | Controlled authentication testing |
| Atomic Red Team | Adversary technique simulation |
| PowerShell | Administration and automation |

Each virtual machine was created and configured based on its role in the network.

---

## Installing Windows 10

The first virtual machine I configured was Windows 10.

This machine acts as a target workstation and is later joined to the Active Directory domain.

### Step 1: Download Windows 10

I downloaded the Windows 10 installation media from Microsoft's official website.

**Download:** https://www.microsoft.com/software-download/windows10

After downloading the Media Creation Tool, I opened it and accepted the Microsoft license terms.

![Windows 10 Download](images/Windows%2010%20Installation%20media.png)

The next step was to select **Create installation media** rather than upgrading the current computer.

![Create Installation Media](images/create%20installatin%20media.png)

I selected the recommended language and architecture settings.

![Recommended Installation Settings](images/Use%20recommended%20option%20when%20upgrading.png)

Next, I selected the ISO file option so the installation media could be attached to VMware.

![Choose ISO File](images/choose%20iso%20file.png)

### Step 2: Create the Virtual Machine

After obtaining the ISO file, I opened VMware Workstation Pro and selected **Create a New Virtual Machine**.

I selected **Typical (recommended)** because the standard VMware configuration was sufficient for the lab.

![Create Virtual Machine](images/VMware%20VM%20creation%20type.png)

I then selected the Windows installation ISO.

![Select Installation ISO](images/choose%20iso%20file%20in%20VMware.png)

The virtual machine was named:

```text
Windows 10 AD-Lab
```

![Virtual Machine Name](images/VM%20naming.png)

For the virtual disk, I allocated 50 GB.

![Virtual Disk Configuration](images/VM%20Capacity.png)

After reviewing the virtual machine configuration, I selected Finish.

![Review Virtual Machine](images/VM%20review.png)

### Step 3: Install Windows 10

After starting the virtual machine, the Windows installation wizard appeared.

![Windows Installation Wizard](images/Windows%2010%20installation%20setup.png)

I selected **Install Now**.

![Install Windows](images/Windows%2010%20installation%20setup%20install%20now.png)

During installation, I selected Windows 10 Pro.

Windows 10 Pro was important for this project because it supports Active Directory domain joining and hosting Remote Desktop connections.

![Windows Edition](images/choose%20version.png)

Next, I selected **Custom: Install Windows only (advanced)**.

![Custom Installation](images/choose%20custom.png)

I selected the available virtual disk and continued the installation.

![Windows Virtual Disk](images/disk%20info.png)

![Windows Installation Progress](images/Windows%2010%20installing.png)

After completing the installation, I configured a local account and finished the initial Windows setup.

This machine was later renamed `target-PC` and joined to the Active Directory domain.

<details>
<summary><strong>Additional Windows 10 installation screenshots</strong></summary>

![Windows License Terms](images/Windows%2010%20License%20terms.png)

![Product Key](images/product%20key.png)

![License Agreement](images/license%20term.png)

</details>

---

## Installing Kali Linux

The next machine I installed was Kali Linux.

Kali Linux is a Debian-based distribution commonly used for penetration testing, vulnerability assessments, and cybersecurity training.

For this project, I downloaded the pre-built VMware image rather than performing a traditional ISO installation.

**Download:** https://www.kali.org/get-kali/#kali-virtual-machines

### Step 1: Download the Pre-Built Image

On the Kali Linux website, I selected the VMware virtual machine image.

![Kali Linux Pre-Built Image](images/kali%20linux%20pre-built.png)

The downloaded package was compressed in `.7z` format.

I extracted it using 7-Zip and located the VMware virtual machine configuration file.

### Step 2: Import the Machine into VMware

After extracting the archive, I opened the VMware configuration file.

![Open Kali Linux Configuration](images/open%20pre-built.png)

VMware automatically recognized the virtual machine.

![Kali Linux in VMware](images/linux%20in%20VMware.png)

After importing the virtual machine, I reviewed the settings and configured its network adapter to use the lab's custom NAT network.

Kali Linux was used later to perform controlled security testing against the Windows 10 workstation.

---

## Installing Windows Server 2022

Windows Server 2022 was installed to host Active Directory Domain Services.

**Download:** https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022

### Step 1: Download Windows Server

I downloaded the Windows Server 2022 ISO from Microsoft's official evaluation website.

![Windows Server Download](images/Windows%20Server%20iso%20page.png)

### Step 2: Create the Virtual Machine

I created a new VMware virtual machine and attached the Windows Server ISO.

The installation process was similar to the Windows 10 installation.

I selected the Windows Server edition with Desktop Experience so I could configure Active Directory through Server Manager and graphical administration tools.

![Windows Server Installation](images/Windows%20Server%20installing.png)

### Step 3: Create the Administrator Account

After installation, Windows Server prompted me to configure the local Administrator password.

![Administrator Account Setup](images/Admin%20User%20creation%20on%20Server.png)

Once the password was configured, I signed in as Administrator.

The Server Manager dashboard appeared, confirming that the initial operating system installation was complete.

![Windows Server Installation Complete](images/Server%20installation%20complete.png)

The next steps involved assigning a static IP address, renaming the server, and configuring Active Directory.

---

## Installing Ubuntu Server

Ubuntu Server was installed to host Splunk Enterprise.

**Download:** https://ubuntu.com/server

### Step 1: Download Ubuntu Server

I downloaded the Ubuntu Server ISO and created another virtual machine in VMware Workstation.

![Ubuntu Server Download](images/Ubuntu%20Server%20page.png)

### Step 2: Install Ubuntu

After attaching the ISO and starting the virtual machine, I selected **Try or Install Ubuntu Server**.

![Ubuntu Installation](images/Ubuntu%20Server%20installation%20page.png)

During installation, I configured the server's hostname, local username, and password.

![Ubuntu Account Configuration](images/Ubuntu%20Server%20User%20account%20creation.png)

After installation, I restarted the server and signed in.

![Ubuntu Server Login](images/Splunk%20Server%20login%20page.png)

### Step 3: Update Ubuntu

I updated the package lists and installed available updates:

```bash
sudo apt update
sudo apt upgrade -y
```

The first command refreshes information about packages available in the configured repositories.

The second command installs available upgrades.

![Ubuntu Updates](images/splunk%20server%20updates.png)

![Ubuntu Package Upgrades](images/splunk%20server%20packet%20updates.png)

After completing the updates, I proceeded to configure VMware networking.

---

# Configuring the VMware Network

Before installing security tools, I needed to make sure all virtual machines could communicate with one another.

I used VMware Workstation's Virtual Network Editor to configure the NAT network.

## Step 1: Configure VMnet8

In VMware Workstation, I opened:

**Edit → Virtual Network Editor**

I selected `VMnet8` and configured it as a NAT network.

The subnet was changed to:

```text
Network: 192.168.10.0
Subnet Mask: 255.255.255.0
CIDR: /24
NAT Gateway: 192.168.10.2
```

![VMware Virtual Network Configuration](images/Virtual%20Network%20Configuration.png)

The NAT gateway allows the virtual machines to access external networks through the VMware host.

One issue I encountered was attempting to use `192.168.10.1` as the gateway.

However, `192.168.10.1` was assigned to the VMware host adapter.

I corrected the NAT gateway to `192.168.10.2`, which resolved the issue.

## Step 2: Connect All Virtual Machines to VMnet8

For each virtual machine, I opened:

**Virtual Machine Settings → Network Adapter**

I selected **Custom: Specific virtual network**, then chose `VMnet8`.

![Configure VMware Network Adapter](images/Set%20VM%20to%20custom%20Virtual%20Network.png)

This step was repeated for all four virtual machines.

## Step 3: Assign a Static IP to the Splunk Server

The Splunk server required a static IP address because Windows endpoints needed a consistent destination for forwarding logs.

I opened Ubuntu's Netplan configuration:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

The network was configured with the following information:

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

The lab's original configuration also included interface matching and naming information.

![Splunk Static IP Configuration](images/splunk%20server%20network%20configuration.png)

After saving the file, I applied the settings:

```bash
sudo netplan apply
```

To verify the assigned IP address:

```bash
ip a
```

![Verify Splunk IP Address](images/ip%20address%20splunk%20server1.png)

![Splunk IP Address Updated](images/ip%20address%20splunk%20server2.png)

Next, I tested connectivity:

```bash
ping -c 4 google.com
```

```bash
ping -c 4 192.168.10.2
```

![Splunk Connectivity Test](images/splunk%20server%20ping.png)

The ping tests confirmed that the server could communicate over the network.

## Step 4: Configure Windows Server Networking

The Windows Server was assigned:

```text
IP Address: 192.168.10.7
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.2
DNS Server: 192.168.10.7
```

![Windows Server IP Configuration](images/windows%20server%20ip%20change.png)

I renamed the Windows Server to `ADDC01`.

![Rename Windows Server](images/Windows%20Server%20name%20change.png)

I also tested network connectivity.

![Windows Server Ping Test](images/Windows%20server%20ping.png)

## Step 5: Configure Windows 10 Networking

The Windows 10 workstation was assigned:

```text
Computer Name: target-PC
IP Address: 192.168.10.100
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.2
DNS Server: 192.168.10.7
```

I first renamed the workstation.

![Rename Windows 10](images/rename%20target%20pc.png)

Next, I configured its IP settings.

![Windows 10 Ethernet Settings](images/Ethernet%20Target%20machine.png)

![Windows 10 Static IP](images/target%20machine%20ip%20change.png)

The Windows 10 machine uses the Domain Controller as its DNS server so that it can resolve Active Directory services.

---

# Installing and Configuring Splunk Enterprise

After completing the virtual network setup, I proceeded to install Splunk Enterprise on the Ubuntu Server.

Splunk is a Security Information and Event Management platform that can collect, index, search, and analyze machine-generated data.

For this project, Splunk was used to monitor security events generated by Windows Server 2022 and Windows 10.

## Step 1: Download Splunk Enterprise

I visited the official Splunk website and downloaded the Linux `.deb` installation package.

**Website:** https://www.splunk.com/

The version used during this project was Splunk Enterprise 10.6.1.1.

![Splunk Enterprise Download](images/Splunk%20enterprise%20image%20download.png)

## Step 2: Install VMware Tools

Before transferring the Splunk installer, I installed `open-vm-tools` to improve integration between Ubuntu and VMware.

```bash
sudo apt install open-vm-tools -y
```

I then enabled the service:

```bash
sudo systemctl enable --now open-vm-tools
```

To verify:

```bash
systemctl status open-vm-tools
```

![VMware Tools Verification](images/open-vm-tools%20verify%20splunk%20server.png)

The service was running successfully.

## Step 3: Configure SSH

To transfer the Splunk installation package from my Windows computer to Ubuntu, I installed OpenSSH Server.

```bash
sudo apt install openssh-server -y
```

Then:

```bash
sudo systemctl enable --now ssh
```

Verify the service:

```bash
sudo systemctl status ssh
```

![SSH Service Verification](images/systemctl%20status%20ssh.png)

When Ubuntu's UFW firewall is enabled, SSH access can be restricted to the host computer:

```bash
sudo ufw allow from 192.168.10.1 to any port 22 proto tcp
```

![SSH Firewall Configuration](images/Allow%20SSH%20from%20Windows%20in%20Splunk%20Server.png)

## Step 4: Transfer the Splunk Installer

I downloaded the Splunk installation file to my Windows host computer and transferred it to Ubuntu using SCP.

SCP stands for Secure Copy Protocol and uses SSH to securely transfer files between systems.

From Windows PowerShell, I navigated to the drive containing the installer.

```powershell
cd F:\
```

Then I ran:

```powershell
scp .\splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb rangeuser21@192.168.10.10:/home/rangeuser21/
```

![Transferring Splunk Installer](images/splunk%20installation%20file%20transfer%20to%20VMware.png)

After the transfer, I verified the installer existed on Ubuntu:

```bash
ls -lh ~/splunk*.deb
```

![Splunk File Verification](images/file%20transfer%20verify.png)

## Step 5: Install Splunk

Because Splunk was downloaded as a Debian package, I did not need to mount an ISO file.

I installed it directly using:

```bash
sudo apt install ~/splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb
```

![Splunk Installation](images/install%20splunk.png)

After installation, I verified the Splunk files were located under:

```text
/opt/splunk/
```

### Important Splunk Directories

| Directory | Purpose |
|---|---|
| `/opt/splunk/bin/` | Executables |
| `/opt/splunk/etc/` | Configuration files |
| `/opt/splunk/var/log/splunk/` | Internal operational logs |
| `/opt/splunk/var/lib/splunk/` | Indexed event data |

![Splunk Directory](images/splunk%20directory%20users%20and%20groups.png)

## Step 6: Start Splunk Enterprise

For the documented lab setup, I used the dedicated `splunk` account to start the service.

```bash
sudo -u splunk /opt/splunk/bin/splunk start --accept-license
```

![Starting Splunk](images/splunk%20server%20start%20with%20the%20user%20splunk.png)

During the initial startup, I configured the Splunk administrator credentials.

## Step 7: Access Splunk Web

Once Splunk started successfully, I opened a browser on my Windows host and navigated to:

```text
http://192.168.10.10:8000
```

Port 8000 is used for accessing Splunk Web.

![Splunk Enterprise Dashboard](images/splunk%20server%20dashboard.png)

At this point, Splunk Enterprise was running successfully.

### Storage Issue Encountered

During the installation, Ubuntu initially ran out of storage space.

The system returned:

```text
No space left on device
```

I expanded the VMware virtual disk to 40 GB.

Because Ubuntu used LVM, the additional disk space was not immediately available to the operating system.

After examining the storage configuration with:

```bash
lsblk
df -h
```

I expanded the relevant partition and logical volume:

```bash
sudo growpart /dev/sda 3
sudo pvresize /dev/sda3
sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
```

These commands applied to this lab's verified LVM configuration and are not universal disk-resizing commands.

After expanding the filesystem, Ubuntu had sufficient available space to complete the installation.

This troubleshooting process helped me understand the difference between expanding a VMware virtual disk and expanding the Linux guest filesystem.

---

# Installing Splunk Universal Forwarder and Sysmon

The next stage involved installing Splunk Universal Forwarder and Microsoft Sysmon on both Windows 10 and Windows Server 2022.

These tools allow endpoint events to be collected and sent to the centralized Splunk Enterprise Server.

## Splunk Universal Forwarder

Universal Forwarder is a lightweight Splunk agent used to collect data and forward it to a Splunk receiver.

### Step 1: Download Universal Forwarder

I downloaded the Windows x64 version from the official Splunk website.

**Download:** https://www.splunk.com/en_us/download/universal-forwarder.html

![Universal Forwarder Download](images/splunk%20universal%20forwarder%20image%20download.png)

After downloading the installer, I opened it on Windows 10.

![Universal Forwarder Installer](images/splunk%20universal%20forwarder%20setup%20start.png)

I followed the installation wizard and configured the receiving indexer.

### Receiving Indexer Configuration

```text
Splunk Server: 192.168.10.10
Receiving Port: 9997
```

![Splunk Receiving Indexer](images/splunk%20universal%20forwarder%20splunk%20server%20ip.png)

I left the Deployment Server field blank because I was not using a dedicated deployment server.

<details>
<summary><strong>Additional Universal Forwarder installation screenshot</strong></summary>

![Universal Forwarder Setup](images/splunk%20universal%20forwarder%20setup%20.png)

</details>

## Installing Sysmon

Sysmon is a Microsoft Sysinternals tool that monitors Windows system activity and generates detailed event logs.

It records security-relevant activities such as process creation, network connections, file creation, registry changes, and DNS queries.

### Step 1: Download Sysmon

**Microsoft Sysmon:** https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

I downloaded Sysmon from Microsoft's official website.

For the configuration, I used Olaf Hartong's Sysmon Modular project.

**GitHub:** https://github.com/olafhartong/sysmon-modular

![Olaf Sysmon Configuration](images/Olaf%20configuration%20file%20link.png)

I selected the balanced configuration and downloaded the XML file.

![Sysmon Configuration](images/Olaf%20configuration.png)

### Step 2: Extract Sysmon

After downloading Sysmon, I extracted the files.

![Sysmon Extracted](images/sysmon%20file%20extracted.png)

### Step 3: Install Sysmon

I opened PowerShell as Administrator and navigated to the extracted Sysmon directory.

The installation command was:

```powershell
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

The configuration file path must match the actual location where the downloaded XML was saved.

![Sysmon Installation](images/sysmon%20installation.png)

### Important Sysmon Event IDs

| Event ID | Description |
|---|---|
| 1 | Process creation |
| 3 | Network connection |
| 7 | Image loaded |
| 10 | Process access |
| 11 | File creation |
| 12 | Registry object created/deleted |
| 13 | Registry value set |
| 22 | DNS query |
| 25 | Process tampering |

These events provide additional visibility into activities occurring on Windows endpoints.

The specific events generated depend on the Sysmon filtering configuration.

## Configuring Splunk Universal Forwarder

Installing Universal Forwarder alone does not tell Splunk which Windows events should be collected.

I therefore configured an `inputs.conf` file.

### Step 1: Create inputs.conf

The configuration file was saved at:

```text
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

The `local` directory is used for custom settings.

I opened Notepad as Administrator so I could save the file.

![Open Notepad as Administrator](images/open%20notepad%20as%20admin.png)

### Step 2: Configure Windows Event Logs

I added the following configuration:

```ini
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

### Explanation

- **Application:** Collects Windows application events.
- **Security:** Collects authentication and security-related events.
- **System:** Collects operating system and service events.
- **Sysmon:** Collects detailed endpoint telemetry.
- **index = endpoint:** Directs events to the Splunk `endpoint` index.
- **renderXml = true:** Preserves the Sysmon events in XML format.

![Save inputs.conf](images/save%20the%20inputs.conf%20file.png)

### Step 3: Restart Universal Forwarder

After saving the configuration, I restarted the Splunk Forwarder service.

```powershell
Restart-Service SplunkForwarder
```

To verify:

```powershell
Get-Service SplunkForwarder
```

The service was running successfully.

![Forwarder Service](images/splunk%20fordwarder%20service%20check.png)

<details>
<summary><strong>Additional Forwarder service screenshots</strong></summary>

![Forwarder Service Verification](images/splunk%20fordwarder%20service1.png)

![Forwarder Service Configuration](images/splunk%20fordwarder%20service2.png)

</details>

### Step 4: Create the endpoint Index

Next, I opened Splunk Enterprise.

Navigate to:

**Settings → Indexes → New Index**

I created an index named:

```text
endpoint
```

![Create endpoint Index](images/create%20a%20new%20index.png)

This index stores the Windows and Sysmon telemetry collected by the Universal Forwarders.

### Step 5: Enable Splunk Receiving

In Splunk Enterprise, I navigated to:

**Settings → Forwarding and Receiving → Configure Receiving**

![Forwarding and Receiving](images/forwarding%20and%20receiving.png)

I configured TCP port:

```text
9997
```

![Configure Receiving Port](images/forwarding%20and%20receiving%20add%20the%20port%20number.png)

### Step 6: Verify Log Collection

After completing the configurations, I opened Search & Reporting.

The following SPL query was used:

```spl
index=endpoint
```

I then checked the hosts sending data.

```spl
index=endpoint
| stats count by host source
```

![Splunk Host Verification](images/splunk%20search%20query%20host%20check.png)

After installing Universal Forwarder and Sysmon on both Windows machines, Splunk showed events from:

```text
target-PC
ADDC01
```

![Both Windows Hosts](images/splunk%20query%20host%20update.png)

The collected data included Windows Security, Application, System, and Sysmon logs.

![Splunk Log Sources](images/splunk%20index%20source%20check.png)

![Splunk Access from Target](images/target%20machine%20splunk.png)

At this stage, the centralized logging environment was functioning successfully.

---

# Installing and Configuring Active Directory

After the virtual network and Splunk monitoring infrastructure were configured, I installed Active Directory Domain Services on Windows Server 2022.

Active Directory allows organizations to centrally manage users, computers, permissions, and authentication.

## Step 1: Install Active Directory Domain Services

On Windows Server, I opened Server Manager and selected:

**Manage → Add Roles and Features**

![Add Roles and Features](images/add%20roles%20and%20features.png)

I selected **Role-based or feature-based installation**.

![Installation Type](images/role-based%20or%20feature-based.png)

Next, I selected the local Windows Server.

![Select Destination Server](images/select%20destination%20server.png)

Under Server Roles, I selected **Active Directory Domain Services**.

![Select Server Roles](images/Select%20server%20roles.png)

![Active Directory Domain Services](images/active%20directory%20domain%20services%20selected.png)

After installing the role, I proceeded to promote the server to a Domain Controller.

## Step 2: Promote the Server

In Server Manager, I selected the notification flag and clicked:

**Promote this server to a domain controller**

![AD DS Post Deployment](images/AD-post-deployment%20configuration.png)

Because this was a completely new domain environment, I selected **Add a new forest**.

The domain name was:

```text
adlab.local
```

![Create New Forest](images/Add%20a%20new%20forest.png)

I then configured the Directory Services Restore Mode password.

![Domain Controller Password](images/Forest%20password%20set.png)

### Active Directory Database

During the installation, the configuration wizard displayed the Active Directory database paths.

![Active Directory Database Paths](images/AD%20paths.png)

One important file associated with Active Directory is:

```text
NTDS.dit
```

This database contains sensitive Active Directory information, including account-related data and password hashes.

Unauthorized attempts to access or extract this database can represent a serious security concern.

From a SOC perspective, suspicious access to Active Directory database files should be investigated carefully.

## Step 3: Complete Installation

After reviewing the installation configuration, I completed the prerequisite checks.

![Prerequisite Checks](images/Prerequisites%20check.png)

Once all prerequisite checks passed, I selected Install.

![Active Directory Installation Complete](images/Active%20directory%20installation%20complete.png)

The server restarted automatically after promotion.

Upon signing in, the account was associated with the newly created Active Directory domain.

![Domain Administrator Login](images/ADLAB-Administrator%20logon.png)

Active Directory Domain Services was now configured successfully.

---

# Creating Organizational Units and Users

After configuring Active Directory, I created Organizational Units and domain users.

Organizational Units (OUs) help administrators organize Active Directory objects based on departments, organizational structure, or administrative requirements.

## Step 1: Open Active Directory Users and Computers

From Server Manager, I selected:

**Tools → Active Directory Users and Computers**

![Active Directory Users and Computers](images/AD%20Users%20and%20computers.png)

I reviewed the built-in Active Directory groups and their membership settings.

![Built-in Active Directory Groups](images/User%20and%20computers%20Builtins%20Groups.png)

## Step 2: Create Organizational Units

To simulate an organization, I created departmental Organizational Units.

For example:

```text
adlab.local
|
+--- IT
|
+--- HR
```

I created the IT Organizational Unit by right-clicking the domain and selecting:

**New → Organizational Unit**

![IT Organizational Unit](images/IT%20OU%20Creation.png)

## Step 3: Create Domain Users

Inside the IT Organizational Unit, I created a domain user.

To create a user:

1. Right-click the Organizational Unit.
2. Select **New → User**.
3. Enter the user's name and account information.
4. Configure the account password.
5. Review the account settings.
6. Finish the user creation wizard.

![Create Active Directory User](images/User%20Creation.png)

I configured a test password for the account.

![User Password Configuration](images/User%20Password%20set.png)

In this controlled environment, I disabled the requirement to change the password at the next logon for a selected test account.

In production environments, appropriate password policies and account lifecycle controls should be enforced.

## Step 4: Verify Users and Groups

After creating the users, I verified they appeared under the correct Organizational Units.

![User Group Verification](images/User%20group%20verify.png)

![Additional User Verification](images/User%20group%20verify2.png)

This exercise helped me understand how Active Directory manages users and groups and how OU structures can support administration.

---

# Joining Windows 10 to Active Directory

After configuring the Domain Controller and creating users, I joined the Windows 10 workstation to the domain.

The domain name was:

```text
adlab.local
```

## Step 1: Open Windows System Properties

On Windows 10, I opened:

**This PC → Properties → Advanced System Settings**

![Windows 10 System Properties](images/W10%20System%20properties.png)

Under the Computer Name tab, I selected Change.

## Step 2: Join the Domain

I selected the Domain option and entered:

```text
adlab.local
```

![Join Windows 10 to Domain](images/W10%20join%20domain.png)

Initially, Windows could not locate the domain.

![Domain Join Error](images/Join%20domain%20error.png)

### Why Did the Domain Join Fail?

The workstation's DNS configuration was not pointing to the Active Directory DNS server.

Active Directory relies heavily on DNS records to locate Domain Controllers and authentication services.

To correct this, I changed the Windows 10 preferred DNS server to:

```text
192.168.10.7
```

![Configure Domain Controller DNS](images/Add%20DNS%20server%20as%20Windows%20Server%20ip.png)

## Step 3: Verify DNS Settings

I opened PowerShell and ran:

```powershell
ipconfig /all
```

![Verify Windows IP Configuration](images/ipconfig%20all.png)

After confirming the DNS server, I attempted the domain join again.

This time Windows requested authorized domain credentials.

![Domain Join Credentials](images/Credentials%20for%20Domain%20change.png)

After authenticating, Windows successfully joined `adlab.local`.

![Domain Joined Successfully](images/Domain%20joined.png)

The machine was restarted to apply the domain configuration.

### What I Learned

This exercise demonstrated that network connectivity alone does not guarantee Active Directory connectivity.

Even if two machines can ping each other, domain joining can fail when DNS is configured incorrectly.

After updating the preferred DNS server to the Domain Controller, the domain join succeeded.

---

# Configuring Kali Linux for Security Testing

With Active Directory and Splunk monitoring configured, I prepared Kali Linux for controlled authentication testing.

## Step 1: Configure Kali's Static IP

On Kali Linux, I opened the Advanced Network Configuration interface.

![Kali Network Settings](images/kali%20linux%20network.png)

I selected Wired Connection 1 and opened its settings.

![Kali Wired Connection](images/kali%20linux%20network%20connection.png)

Under IPv4 Settings, I changed the method to Manual.

The final lab settings were:

```text
IP Address: 192.168.10.250
Subnet Mask: 255.255.255.0
Gateway: 192.168.10.2
DNS Server: 192.168.10.7
```

![Kali Static IP Configuration](images/kali%20linux%20network%20configuration.png)

## Step 2: Verify the IP Address

I opened a terminal and ran:

```bash
ip a
```

Initially, the configured IP address was not reflected in the active interface.

![Kali IP Before Reconnection](images/kali%20linux%20ip%20a1.png)

I disconnected and reconnected the network interface to apply the new configuration.

Afterward, the address was correct.

![Kali Static IP Verified](images/kali%20linux%20ip%20a2.png)

## Step 3: Test Connectivity

I tested communication with the Splunk Server:

```bash
ping -c 4 192.168.10.10
```

I also tested the Windows 10 workstation:

```bash
ping -c 4 192.168.10.100
```

![Kali Network Connectivity](images/kali%20linux%20ping.png)

Once connectivity was verified, I proceeded with the authorized authentication testing exercise.

---

# RDP Authentication Testing Using NetExec

One of the main security exercises in this project involved generating authentication activity against a domain-joined Windows 10 workstation.

The goal was to simulate repeated failed password attempts and observe how Windows and Active Directory recorded the authentication events.

These events were then investigated using Splunk Enterprise.

## Step 1: Enable Remote Desktop on Windows 10

Before running the authentication test, I enabled Remote Desktop on the Windows 10 target.

I opened:

**System Properties → Remote**

Then selected:

**Allow remote connections to this computer**

I kept Network Level Authentication enabled.

![Enable Remote Desktop](images/allow%20remote%20connection.png)

Next, I selected the authorized domain user who would be allowed to access the system through RDP.

![Configure Remote Desktop Users](images/add%20users%20in%20remote%20connection.png)

## Step 2: Verify the RDP Service

From Kali Linux, I verified that Remote Desktop was accessible.

```bash
nmap -Pn -p 3389 192.168.10.100
```

The result confirmed:

```text
3389/tcp open
```

Port 3389 is commonly used by Remote Desktop Protocol.

The open port confirmed that the service was reachable from Kali Linux.

## Step 3: Install NetExec

I originally experimented with Crowbar, but encountered compatibility problems between Crowbar and FreeRDP 3 on Kali Linux 2026.2.

Instead of replacing the Kali operating system, I switched to NetExec, which successfully distinguished accepted and rejected credentials in my environment.

NetExec is an open-source network security assessment tool that supports multiple authentication protocols.

Official documentation:

https://www.netexec.wiki/

I installed it with:

```bash
sudo apt update
sudo apt install netexec -y
```

![NetExec Installation](images/kali%20linux%20apt%20update%20and%20install%20netexec.png)

I verified the installed version:

```bash
nxc --version
```

![NetExec Version](images/netexec%20version.png)

I also reviewed the available options:

```bash
nxc --help
```

![NetExec Help](images/netexec%20help.png)

## Step 4: Prepare the Test Environment

I created a project directory on Kali Linux:

```bash
mkdir -p ~/Desktop/ad-project
cd ~/Desktop/ad-project
```

I used a small password list for the controlled exercise.

Kali Linux includes the RockYou wordlist under `/usr/share/wordlists`.

The lab documentation shows extracting the wordlist and selecting a small number of entries.

![Wordlist Extraction](images/unzip%20the%20password%20file.png)

I used a limited set of entries rather than performing a large-scale test.

For example:

```bash
head -n 20 rockyou.txt
```

![Preview Wordlist](images/Get%20first%2020%20password.png)

The selected entries were stored in `passwords.txt`.

The test used a disposable Active Directory account created specifically for this lab.

### Security Considerations

The password list was used only against the authorized Windows 10 virtual machine.

Repeated incorrect passwords can lock Active Directory accounts, so authentication testing should be planned with appropriate account-lockout policies and test boundaries.

<details>
<summary><strong>Additional password-file preparation</strong></summary>

![Password File Permissions](images/password%20file%20permissions.png)

<!--
The screenshot below is intentionally disabled because it may display
the contents of the lab password list. Review and redact it before publishing.

![Password File Contents](images/new%20file%20read.png)
-->

</details>

## Step 5: Perform Controlled RDP Authentication Testing

The documented test used the following NetExec command:

```bash
nxc -t 1 --jitter 3 rdp 192.168.10.100 -d ADLAB -u tsmith -p passwords.txt
```

### Command Explanation

| Parameter | Description |
|---|---|
| `nxc` | Runs NetExec |
| `-t 1` | Uses one concurrent worker |
| `--jitter 3` | Adds spacing between attempts |
| `rdp` | Selects Remote Desktop Protocol |
| `192.168.10.100` | Windows 10 target |
| `-d ADLAB` | Specifies the domain |
| `-u tsmith` | Specifies the lab account |
| `-p passwords.txt` | Uses the test password list |

The test produced multiple rejected authentication attempts.

NetExec displayed:

```text
STATUS_LOGON_FAILURE
```

This indicates that Windows rejected the supplied credentials.

When the known valid lab credential was tested, NetExec reported successful authentication and displayed an administrative-access indicator.

**The original screenshot contains the lab password. Redact it before enabling this image in a public README.**

<!--
![NetExec Authentication Testing](images/netexec%20password%20brute%20force%20attack.png)
-->

## Step 6: Investigate the Events in Splunk

After running the authentication test, I opened Splunk Search & Reporting.

The relevant Windows Security events were forwarded to the `endpoint` index.

The Windows 10 workstation recorded failed logon activity.

![Event ID 4625](images/EventID%204625.png)

The documented investigation showed **60 failed logon events** in one filtered Splunk result.

This count represents the events visible in that particular search, not necessarily the total number of authentication failures produced during the entire project.

I then filtered the events to investigate the test account.

```spl
index=endpoint source="WinEventLog:Security" EventCode=4625 "tsmith"
| table _time host _raw
| sort -_time
```

![Splunk Investigation After Authentication Test](images/splunk%20query%20after%20attack.png)

The logs provided information about the failed authentication activity, including account information and the originating system.

![Investigate Source Address](images/Splunk%20check%20ip%20of%20the%20attacker.png)

### Why This Matters

A large number of failed logins against the same account within a short period may indicate:

- Password guessing
- Unauthorized authentication attempts
- Misconfigured applications
- Credential misuse
- Automated login activity

As a SOC analyst, it is important to investigate the account, source workstation, event timestamps, failure codes, and any successful authentications that occur afterward.

---

# Investigating Authentication Events in Splunk

After the controlled authentication tests, I continued investigating the Windows Security events collected by Splunk.

The two main systems involved were:

- **ADDC01:** Domain Controller
- **target-PC:** Windows 10 workstation

## Windows Event ID 4625 — Failed Logon

Event ID 4625 indicates a failed Windows logon attempt.

Example SPL:

```spl
index=endpoint source="WinEventLog:Security" EventCode=4625
| table _time host _raw
| sort -_time
```

This query displays failed logon events from the monitored Windows systems.

The raw event data can include:

- Target account
- Account domain
- Logon type
- Source network address
- Authentication package
- Failure reason
- Status and substatus codes

These fields help determine whether an authentication failure came from a user mistake, application issue, or suspicious activity.

## Windows Event ID 4624 — Successful Logon

Event ID 4624 records successful Windows logons.

```spl
index=endpoint source="WinEventLog:Security" EventCode=4624
| table _time host _raw
| sort -_time
```

It is important to review the logon type because Event ID 4624 can represent different forms of authentication.

For example:

| Logon Type | Description |
|---|---|
| 2 | Interactive logon |
| 3 | Network logon |
| 5 | Service logon |
| 10 | Remote interactive logon |
| 11 | Cached interactive logon |

Logon Type 10 is particularly relevant when investigating established RDP sessions.

## Windows Event ID 4776 — NTLM Credential Validation

Event ID 4776 records NTLM credential validation.

During the project, the Domain Controller recorded failed and successful NTLM validation requests involving the test account.

The relevant authentication codes included:

| Error Code | Meaning |
|---|---|
| `0x0` | Credential validation succeeded |
| `0xC000006A` | Incorrect password |
| `0xC0000234` | Account locked out |

I used the following search to investigate these events:

```spl
index=endpoint host=ADDC01 source="WinEventLog:Security" EventCode=4776 "tsmith"
| table _time host _raw
| sort -_time
```

### Extract Authentication Results

I also used `rex` and `eval` to classify authentication results.

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

This query categorizes NTLM credential validations based on their error code.

### Important Distinction

A successful Event ID 4776 indicates that credentials were accepted by NTLM credential validation.

It does not automatically prove that a complete RDP desktop session was established.

To confirm an actual remote-interactive logon, additional events such as Event ID 4624 with Logon Type 10 should be investigated.

---

# Testing with Atomic Red Team

After completing the authentication monitoring exercise, I used Atomic Red Team to generate additional endpoint security telemetry.

Atomic Red Team is an open-source project maintained by Red Canary.

It contains individual tests mapped to techniques from the MITRE ATT&CK framework.

These tests can help defenders understand how certain attacker behaviors appear in endpoint logs.

**Repository:** https://github.com/redcanaryco/atomic-red-team

## Step 1: Prepare the Windows 10 Target

Atomic Red Team was configured on the Windows 10 target machine.

I opened PowerShell as Administrator and prepared the environment for installing the Invoke-AtomicRedTeam framework.

The original lab documentation included a relaxed PowerShell execution policy and a broad Microsoft Defender exclusion during initial experimentation.

These settings weaken Windows protection and should not be treated as recommended deployment steps.

A safer approach is to review the selected tests, keep Defender enabled, and use a disposable VM snapshot.

![PowerShell Execution Policy](images/Set-Exectution%20Policy.png)

## Step 2: Install Atomic Red Team

The project used the Invoke-AtomicRedTeam PowerShell framework.

**Repository:** https://github.com/redcanaryco/invoke-atomicredteam

The documented installer command was:

```powershell
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
```

Then:

```powershell
Install-AtomicRedTeam -getAtomics
```

Remote installation scripts should be reviewed before execution.

![Install Atomic Red Team](images/install%20Atomic%20Red%20Team.png)

After installation, I examined the Atomic Red Team directory.

![Atomic Red Team Directory](images/Atomic%20Red%20Team%20folder.png)

The directory contained tests organized by MITRE ATT&CK technique identifiers.

![Atomic Test Files](images/Atomic%20Red%20Team%20folder%20run.png)

## Step 3: Explore MITRE ATT&CK

MITRE ATT&CK is a knowledge base documenting tactics and techniques associated with adversary behavior.

Atomic Red Team maps individual tests to ATT&CK technique identifiers.

![MITRE ATTACK Framework](images/MITRE%20ATT%26CK.png)

For example:

```text
T1136.001 - Create Account: Local Account
T1059.001 - Command and Scripting Interpreter: PowerShell
```

![MITRE Techniques](images/Testing%20differnt%20techniques%20ID.png)

## Step 4: Test T1136.001 — Local Account Creation

The first documented technique was:

**T1136.001 — Create Account: Local Account**

Attackers can abuse account creation to establish persistence or maintain unauthorized access.

The test was executed using:

```powershell
Invoke-AtomicTest T1136.001
```

After running the selected test, I opened Splunk Enterprise and searched for related activity.

```spl
index=endpoint "NewLocalUser"
```

The documented search returned **14 raw events** associated with the activity.

This confirmed that the monitoring environment was collecting relevant telemetry during the exercise.

![Splunk Atomic Red Team Investigation](images/splunk%20check%20after%20Atomic%20Red%20Team%20run.png)

## Step 5: Test T1059.001 — PowerShell

The next documented technique was:

**T1059.001 — Command and Scripting Interpreter: PowerShell**

PowerShell is a legitimate administrative tool, but it can also be misused to execute suspicious commands and scripts.

The Atomic Red Team test was executed using:

```powershell
Invoke-AtomicTest T1059.001
```

I then searched Splunk for PowerShell-related telemetry.

```spl
index=endpoint powershell
```

The investigation identified endpoint activity involving `powershell.exe` and registry-related telemetry.

One event referenced:

```text
HKLM\System\CurrentControlSet\Services\bam\State\UserSettings
```

BAM stands for Background Activity Moderator.

Windows uses BAM to track certain application activity.

The observed registry-related event showed that Sysmon was collecting additional endpoint information during the test.

However, a BAM registry event alone does not establish that malicious activity occurred. The process context and surrounding events must also be considered.

![PowerShell Atomic Red Team Investigation](images/Splunk%20check%20after%20running%20Powershell%20Atomic%20Red%20team.png)

### Why Atomic Red Team Was Useful

Atomic Red Team helped me understand how known adversary techniques can be represented by security events.

By generating controlled activity and then investigating it using Splunk, I was able to connect offensive testing with defensive monitoring.

This approach is commonly associated with purple teaming, where attack simulations are used to improve detection and response capabilities.

### Historical Security Configuration

The original project screenshots also included a broad Microsoft Defender exclusion.

That configuration should not be reproduced as a normal security practice. The screenshot is intentionally excluded from public rendering because it could encourage weakening endpoint protection without sufficient justification.

<!--
Historical lab screenshot — included in the image archive but not displayed:

![Experimental Defender Exclusion](images/Executions%20add%20C%20drive.png)
-->

---

# Splunk Detection Queries

The following SPL searches demonstrate how the collected events can be used for additional SOC monitoring and detection engineering.

These are reusable investigation examples. They are not all claims of scheduled alerts that were deployed during the original lab.

## 1. Count Failed Windows Logons

```spl
index=endpoint source="WinEventLog:Security" EventCode=4625
| stats count as FailedLogons by host
| sort -FailedLogons
```

This query counts failed Windows logon events by host.

It can help identify machines experiencing high volumes of authentication failures.

## 2. Investigate Failed NTLM Authentication

```spl
index=endpoint host=ADDC01 source="WinEventLog:Security" EventCode=4776
| rex field=_raw "Logon Account:\s+(?<Account>\S+)"
| rex field=_raw "Source Workstation:\s+(?<Workstation>[^\r\n]+)"
| rex field=_raw "Error Code:\s+(?<AuthStatus>0x[0-9A-Fa-f]+)"
| where lower(AuthStatus)="0xc000006a"
| stats count as FailedAttempts by Account Workstation
| sort -FailedAttempts
```

This query identifies accounts and source workstations generating incorrect-password validation events.

## 3. Detect Multiple Failures Followed by Success

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

This detection looks for successful NTLM credential validation preceded by five or more incorrect-password attempts from the same account and workstation within ten minutes.

The pattern is consistent with possible password guessing followed by a successful credential validation.

It should be treated as an investigation lead rather than definitive evidence of a compromised account.

## 4. Search for Sysmon Events

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| stats count by host
```

This confirms that Sysmon telemetry is arriving from the monitored Windows systems.

## 5. Extract Sysmon Event IDs

Because Sysmon data was collected in XML format, I used field extraction to identify Event IDs.

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| stats count by host SysmonEventID
| sort -count
```

This allows the event data to be grouped by Sysmon Event ID.

## 6. Investigate Process Creation

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| where SysmonEventID="1"
| table _time host _raw
| sort -_time
```

Sysmon Event ID 1 records process creation.

SOC analysts can use these events to investigate suspicious programs, command-line activity, and process relationships.

## 7. Investigate an Active Directory Account

```spl
index=endpoint source="WinEventLog:Security" "tsmith"
| table _time host EventCode _raw
| sort -_time
```

This displays Windows Security events associated with the lab account.

## 8. Verify Log Collection

```spl
index=endpoint
| stats count by host source
| sort -count
```

This provides an overview of the hosts and event sources sending telemetry to Splunk.

---

# Challenges and Troubleshooting

Throughout this project, I encountered several issues that required troubleshooting.

These challenges were valuable because they helped me better understand how virtualization, networking, Windows authentication, and SIEM systems work together.

### 1. VMware NAT Gateway Misconfiguration

**Problem:** Ubuntu initially could not communicate with external networks.

**Cause:** The VMware NAT gateway was incorrectly configured using the host adapter address.

**Solution:** I corrected the gateway to `192.168.10.2` and updated the Ubuntu default route.

**Lesson Learned:** Proper IP addressing and routing are essential for network communication.

### 2. Insufficient Storage During Splunk Installation

**Problem:** Splunk installation failed because Ubuntu ran out of disk space.

**Cause:** The Ubuntu root filesystem did not have enough storage available.

**Solution:** I expanded the VMware virtual disk and then extended the Ubuntu LVM filesystem.

**Lesson Learned:** Expanding a virtual disk and expanding the guest filesystem are separate operations.

### 3. Active Directory Domain Join Failure

**Problem:** Windows 10 was unable to join the `adlab.local` domain.

**Cause:** The workstation's DNS server was not configured to use the Domain Controller.

**Solution:** I changed the preferred DNS server to `192.168.10.7`.

**Lesson Learned:** Active Directory relies on DNS to locate domain services.

### 4. Kali Linux Could Not Ping Windows 10

**Problem:** Kali Linux could not ping Windows 10, even though Windows 10 could ping Kali.

**Cause:** Windows Firewall was restricting incoming ICMP traffic.

**Solution:** I configured a firewall rule allowing ICMP requests from the authorized Kali IP address.

**Lesson Learned:** Firewall rules can permit outbound traffic while restricting inbound traffic.

### 5. Splunk Searches Returned No Results

**Problem:** Initial SPL searches returned no expected Windows authentication events.

**Cause:** The events were stored in the `endpoint` index rather than the default `main` index.

**Solution:** I updated my SPL searches to use:

```spl
index=endpoint
```

**Lesson Learned:** Analysts must understand their data sources and index configurations before investigating events.

### 6. Sysmon Event ID Extraction

**Problem:** Sysmon events were arriving in Splunk, but the `EventCode` field was not available in the expected format.

**Cause:** The Sysmon data was ingested in XML format.

**Solution:** I used the `rex` command to extract the Event ID from the raw XML event.

**Lesson Learned:** Field extraction is an important part of understanding and investigating log data.

### 7. Crowbar and FreeRDP Compatibility

**Problem:** Crowbar generated errors and reported incorrect authentication successes.

**Cause:** Crowbar's older FreeRDP integration was incompatible with the version included in Kali Linux 2026.2.

**Solution:** I switched to NetExec, which accurately identified accepted and rejected credentials in this lab.

**Lesson Learned:** Security testing tools can behave differently across software versions, and their results should be verified using endpoint logs.

### 8. Kali Linux Mouse Cursor Issue

**Problem:** The mouse cursor disappeared inside the Kali Linux virtual machine, even though mouse clicks continued working.

**Cause:** A VMware guest cursor-rendering issue.

**Solution:** I configured Xorg to use software cursor rendering.

**Lesson Learned:** Virtual machine usability problems may involve graphics rendering rather than input hardware.

---

# Project Results

The project successfully established a working Active Directory environment with centralized security monitoring.

### Completed Components

| Component | Result |
|---|---|
| VMware virtual machines | Successfully deployed |
| Custom NAT network | Configured |
| Windows Server 2022 | Installed |
| Active Directory Domain Services | Configured |
| Domain users and OUs | Created |
| Windows 10 domain join | Successful |
| Ubuntu Server | Installed |
| Splunk Enterprise | Running |
| Splunk Universal Forwarder | Installed on both Windows systems |
| Sysmon | Installed and configured |
| Windows Security logs | Forwarding successfully |
| Sysmon telemetry | Forwarding successfully |
| Kali Linux network | Configured |
| NetExec authentication testing | Successful |
| Authentication event analysis | Completed |
| Atomic Red Team tests | Executed and investigated |

### Security Monitoring Results

The monitoring environment collected Windows Security and Sysmon telemetry from both Windows machines.

The project documented:

- Multiple failed Windows logon events.
- NTLM credential validation failures.
- Successful NTLM credential validations.
- Sysmon process creation activity.
- Registry-related events.
- File creation activity.
- Atomic Red Team-generated endpoint events.

The saved documentation includes a Splunk search showing 60 failed logon events in one filtered view and a search showing 14 raw events associated with the Atomic Red Team account-creation exercise.

These results demonstrated that the monitoring pipeline was collecting and presenting security-relevant telemetry.

The most important outcome was understanding how activities generated on endpoints can be correlated with security events collected in a centralized SIEM.

---

# Lessons Learned

### 1. Understanding Active Directory Infrastructure

This project helped me understand how Active Directory manages users, computers, authentication, and organizational structures.

I gained hands-on experience configuring Windows Server as a Domain Controller, creating Organizational Units, managing test accounts, and joining a workstation to the domain.

### 2. Importance of Network Configuration

I learned how IP addressing, subnetting, default gateways, DNS, and VMware NAT networking affect communication between systems.

Troubleshooting network connectivity helped me understand how small configuration mistakes can disrupt an entire lab environment.

### 3. Centralized Security Monitoring

Configuring Splunk Enterprise, Universal Forwarder, and Sysmon helped me understand the complete log collection process.

I learned that simply installing a SIEM is not enough. Proper inputs, receiving ports, indexes, service permissions, and event sources are necessary for collecting useful telemetry.

### 4. Investigating Windows Authentication

By generating controlled authentication activity, I learned how Windows records failed and successful logon attempts.

I practiced examining Event IDs 4624, 4625, and 4776 to investigate account activity and understand authentication results.

### 5. Learning Splunk Search Processing Language

I gained practical experience using SPL operators such as:

- `search`
- `table`
- `stats`
- `sort`
- `where`
- `eval`
- `rex`
- `streamstats`

These commands allowed me to filter, extract, organize, and investigate Windows security events.

### 6. Understanding the MITRE ATT&CK Framework

Atomic Red Team helped me understand how individual adversary techniques are categorized in MITRE ATT&CK.

By running selected techniques and reviewing the resulting events, I became more familiar with how simulated attacker behaviors can be investigated from a defensive perspective.

### 7. Troubleshooting and Problem Solving

One of the most valuable parts of the project was troubleshooting real configuration problems.

Issues involving VMware networking, Active Directory DNS, Linux storage, FreeRDP compatibility, and Splunk searches helped strengthen my ability to approach technical problems systematically.

---

# Conclusion

This project provided hands-on experience designing and implementing an Active Directory environment with centralized security monitoring.

Using VMware Workstation Pro, I deployed Windows Server 2022, Windows 10, Ubuntu Server, and Kali Linux in a virtual network.

I configured Active Directory Domain Services, created users and Organizational Units, joined Windows 10 to the domain, and established a centralized logging environment using Splunk Enterprise.

By installing Sysmon and Splunk Universal Forwarder, I was able to collect detailed Windows endpoint and authentication events.

I then performed controlled RDP authentication testing using NetExec and investigated the resulting activity in Splunk.

Finally, I used Atomic Red Team to simulate selected MITRE ATT&CK techniques and analyze the endpoint telemetry generated by those tests.

Overall, this project strengthened my knowledge of:

- Windows Server administration
- Active Directory management
- VMware networking
- Windows authentication
- Splunk Enterprise
- Sysmon monitoring
- Security event analysis
- Threat detection
- MITRE ATT&CK
- Purple-team testing
- Troubleshooting and incident investigation

The experience gave me a better understanding of how security professionals build monitoring environments, investigate suspicious activity, and use collected telemetry to support defensive security operations.

The lab can also serve as a foundation for future SOC investigations, threat-hunting exercises, and detection engineering projects.

---

# References

### Microsoft

- [Windows 10 Installation Media](https://www.microsoft.com/software-download/windows10)
- [Windows Server 2022](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022)
- [Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/)
- [Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Windows Security Auditing](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/)

### VMware

- [VMware Documentation](https://techdocs.broadcom.com/)

### Ubuntu

- [Ubuntu Server](https://ubuntu.com/server)
- [Ubuntu Server Documentation](https://documentation.ubuntu.com/server/)

### Splunk

- [Splunk Enterprise](https://www.splunk.com/)
- [Splunk Documentation](https://docs.splunk.com/)
- [Splunk Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html)
- [Splunk Search Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/)

### Security Testing

- [Kali Linux](https://www.kali.org/)
- [Olaf Hartong — Sysmon Modular](https://github.com/olafhartong/sysmon-modular)
- [NetExec Documentation](https://www.netexec.wiki/)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [Invoke-AtomicRedTeam](https://github.com/redcanaryco/invoke-atomicredteam)
- [MITRE ATT&CK](https://attack.mitre.org/)

---

## AI Transparency

> **AI-Assisted Documentation:** This README was organized and written with assistance from ChatGPT using my original project documentation, screenshots, and lab results. The configurations, testing, troubleshooting, and security investigations described in this repository are based on the hands-on activities performed during my Active Directory SOC HomeLab project. AI assistance was used to improve documentation structure, technical explanations, clarity, and readability.

---

## Security Disclaimer

All security testing documented in this repository was performed against authorized virtual machines in an isolated HomeLab environment.

The purpose of this project is educational, with an emphasis on Active Directory administration, defensive security monitoring, threat detection, and incident investigation.

No production systems or unauthorized third-party assets were targeted.

**This repository is intended as a personal cybersecurity learning and portfolio project.**
