
<div align="center">

# Active Directory SOC HomeLab

### Building an Active Directory Environment with Centralized Security Monitoring, Threat Detection, and Attack Simulation

![VMware](https://img.shields.io/badge/Virtualization-VMware_Workstation-0078D4?style=for-the-badge)
![Windows](https://img.shields.io/badge/Windows-Server_2022_&_Windows_10-0078D4?style=for-the-badge)
![Splunk](https://img.shields.io/badge/SIEM-Splunk_Enterprise-F36E21?style=for-the-badge)
![Sysmon](https://img.shields.io/badge/Monitoring-Sysmon-228B22?style=for-the-badge)
![Kali](https://img.shields.io/badge/Security_Testing-Kali_Linux-557C94?style=for-the-badge)

**Active Directory | Windows Server | Splunk Enterprise | Sysmon | NetExec | Atomic Red Team | MITRE ATT&CK**

</div>

---

# Project Overview

This project focuses on building a virtualized Active Directory environment and implementing centralized security monitoring using Splunk Enterprise, Splunk Universal Forwarder, and Microsoft Sysmon.

The primary objective was to simulate a small enterprise network where I could configure and manage Active Directory, collect security logs from multiple Windows systems, simulate authorized attack techniques, and investigate suspicious activity through Splunk.

The environment was built using **VMware Workstation Pro** and consists of four virtual machines: Windows Server 2022, Windows 10, Ubuntu Server, and Kali Linux.

Windows Server 2022 serves as the Active Directory Domain Controller, while Windows 10 represents a domain-joined workstation. Ubuntu Server hosts Splunk Enterprise for centralized log collection and analysis, and Kali Linux is used to conduct controlled security testing against the Windows environment.

To enhance endpoint visibility, I installed **Microsoft Sysmon with Olaf Hartong's modular configuration** on both Windows machines. I also configured Splunk Universal Forwarder to send Windows Security, System, Application, and Sysmon events to the Splunk Enterprise server.

Once the monitoring environment was established, I performed controlled RDP authentication testing using NetExec and simulated selected MITRE ATT&CK techniques using Atomic Red Team.

The generated activity was investigated using Splunk Search Processing Language (SPL) to identify failed authentications, successful credential validations, process activity, and other security events.

### Project Objectives

- Build a virtual enterprise network using VMware Workstation.
- Install and configure Windows Server 2022 as an Active Directory Domain Controller.
- Create and manage Organizational Units and domain user accounts.
- Join a Windows 10 workstation to the Active Directory domain.
- Install Splunk Enterprise on Ubuntu Server.
- Configure Splunk Universal Forwarder on Windows endpoints.
- Implement Microsoft Sysmon for detailed endpoint monitoring.
- Centralize Windows event logs in Splunk Enterprise.
- Conduct controlled authentication testing from Kali Linux.
- Investigate failed and successful authentication events.
- Simulate selected adversary techniques with Atomic Red Team.
- Practice security monitoring, threat detection, and incident investigation using SPL.

> **Security Disclaimer:** All configurations and security tests documented in this project were performed within an authorized virtual HomeLab environment for educational purposes. No production systems or unauthorized external assets were targeted.

---

# Table of Contents

1. [Network Architecture](#1-network-architecture)
2. [Tools and Technologies](#2-tools-and-technologies)
3. [Virtual Machine Installation](#3-virtual-machine-installation)
   - [Windows 10](#31-installing-windows-10)
   - [Kali Linux](#32-installing-kali-linux)
   - [Windows Server 2022](#33-installing-windows-server-2022)
   - [Ubuntu Server](#34-installing-ubuntu-server)
4. [VMware Network Configuration](#4-vmware-network-configuration)
5. [Splunk Enterprise Installation](#5-splunk-enterprise-installation)
6. [Splunk Universal Forwarder and Sysmon](#6-splunk-universal-forwarder-and-sysmon)
7. [Active Directory Domain Services](#7-active-directory-domain-services)
8. [Active Directory Users and Organizational Units](#8-active-directory-users-and-organizational-units)
9. [Joining Windows 10 to Active Directory](#9-joining-windows-10-to-active-directory)
10. [Kali Linux Configuration](#10-kali-linux-configuration)
11. [RDP Authentication Testing with NetExec](#11-rdp-authentication-testing-with-netexec)
12. [Security Event Investigation with Splunk](#12-security-event-investigation-with-splunk)
13. [Atomic Red Team Simulation](#13-atomic-red-team-simulation)
14. [SPL Threat Detection Queries](#14-spl-threat-detection-queries)
15. [Project Results and Findings](#15-project-results-and-findings)
16. [Conclusion](#16-conclusion)
17. [References](#17-references)

---

# 1. Network Architecture

The first stage of the project involved designing a network architecture that would allow multiple virtual machines to communicate with one another while maintaining a separate laboratory environment.

The network consists of an Active Directory Domain Controller, a Windows workstation, a centralized Splunk server, and a Kali Linux security testing machine.

All systems were connected through a VMware NAT network using the `192.168.10.0/24` address space.

## 1.1 Network Diagram

![Active Directory HomeLab Architecture](images/Lab%20Logical%20Diagram.jpeg)

The diagram illustrates how the virtual machines communicate and how security telemetry is collected from the Windows endpoints.

## 1.2 Network Configuration

| Device | Operating System | IP Address | Purpose |
|---|---|---|---|
| Domain Controller | Windows Server 2022 | `192.168.10.7` | Active Directory and DNS |
| Target Workstation | Windows 10 Pro | `192.168.10.100` | Domain-joined endpoint |
| Splunk Server | Ubuntu Server | `192.168.10.10` | SIEM and log collection |
| Attacker Machine | Kali Linux | `192.168.10.250` | Controlled security testing |
| NAT Gateway | VMware NAT | `192.168.10.2` | Network gateway |
| VMware Host Adapter | Windows Host | `192.168.10.1` | Host-side virtual adapter |

### Active Directory Configuration

```text
Domain Name       : adlab.local
NetBIOS Name      : ADLAB
Domain Controller : ADDC01
Target Workstation: target-PC

Network           : 192.168.10.0/24
Subnet Mask       : 255.255.255.0
Default Gateway   : 192.168.10.2
```

## 1.3 How the Environment Works

The Domain Controller is responsible for managing Active Directory accounts, authentication requests, and DNS resolution.

Windows 10 is connected to the Active Directory domain and acts as a monitored endpoint.

Splunk Enterprise runs on Ubuntu Server and receives logs from Windows Server 2022 and Windows 10.

Kali Linux is used to simulate authorized security activity against the Windows workstation.

The activity is recorded by Windows Event Logging and Sysmon, then forwarded to Splunk for investigation.

### Security Monitoring Architecture

```text
                     KALI LINUX
                    192.168.10.250
                          |
                          | Controlled Security Testing
                          |
                          v
                   WINDOWS 10 TARGET
                    192.168.10.100
                          |
                          | Windows Security Events
                          | Sysmon Telemetry
                          |
                          v
                  SPLUNK UNIVERSAL
                      FORWARDER
                          |
                          | TCP 9997
                          |
                          v
                    SPLUNK SERVER
                    192.168.10.10
                          |
                          v
                 SECURITY MONITORING
                    & LOG ANALYSIS
```

The Domain Controller also forwards authentication and endpoint events to Splunk, providing visibility into domain authentication activity.

> **Why this matters:** Centralized security monitoring allows SOC analysts to investigate events from multiple machines in one location rather than reviewing each endpoint individually.

---

# 2. Tools and Technologies

The following technologies were used throughout the project.

| Tool / Technology | Purpose |
|---|---|
| VMware Workstation Pro | Creates and manages virtual machines |
| Windows Server 2022 | Hosts Active Directory and DNS |
| Windows 10 Pro | Domain-joined endpoint |
| Ubuntu Server | Hosts Splunk Enterprise |
| Kali Linux 2026.2 | Security testing environment |
| Active Directory Domain Services | Identity and authentication management |
| DNS Server | Domain name resolution |
| Splunk Enterprise | Centralized security log collection and analysis |
| Splunk Universal Forwarder | Forwards Windows event logs to Splunk |
| Microsoft Sysmon | Collects detailed endpoint telemetry |
| Olaf Hartong Sysmon Modular | Provides Sysmon monitoring rules |
| NetExec | Performs authorized authentication testing |
| Atomic Red Team | Simulates MITRE ATT&CK techniques |
| MITRE ATT&CK | Framework for understanding adversary behavior |
| PowerShell | Windows administration and configuration |
| SPL | Search Processing Language for security investigations |

### Project Environment

This project uses a combination of Windows and Linux systems to simulate a small enterprise environment.

The monitoring approach focuses on collecting relevant Windows security events and analyzing them through a centralized SIEM.

---

# 3. Virtual Machine Installation

Before configuring Active Directory and Splunk, I installed all the required operating systems using VMware Workstation Pro.

The virtual machines were configured with appropriate processing, memory, disk, and network resources based on their assigned roles.

---

## 3.1 Installing Windows 10

Windows 10 was used as the target workstation for domain authentication, endpoint monitoring, and security testing.

Windows 10 Pro was selected because it supports Active Directory domain membership and Remote Desktop hosting.

### Step 1 — Download Windows 10

I downloaded the Windows 10 installation media from Microsoft's official website.

**Official Download:**

https://www.microsoft.com/software-download/windows10

After downloading the Media Creation Tool, I launched the application and accepted the license agreement.

![Windows 10 Installation Media](images/Windows%2010%20Installation%20media.png)

Next, I selected **Create installation media** instead of upgrading the host computer.

![Create Installation Media](images/create%20installatin%20media.png)

I selected the recommended language and architecture settings.

![Recommended Installation Options](images/Use%20recommended%20option%20when%20upgrading.png)

Next, I selected the ISO file option.

![Windows ISO Selection](images/choose%20iso%20file.png)

**What this does:** Creating a Windows ISO provides bootable installation media that can be attached to a virtual machine without requiring a physical USB drive or DVD.

### Step 2 — Create the Windows 10 Virtual Machine

After downloading the ISO file, I opened VMware Workstation Pro and selected:

**File → New Virtual Machine**

I selected **Typical (recommended)**.

![VMware VM Creation](images/VMware%20VM%20creation%20type.png)

I then selected the downloaded Windows 10 ISO file.

![Windows 10 ISO in VMware](images/choose%20iso%20file%20in%20VMware.png)

The virtual machine was named:

```text
Windows 10 AD-Lab
```

![Virtual Machine Naming](images/VM%20naming.png)

For storage, I allocated a 50 GB virtual disk.

![VM Disk Capacity](images/VM%20Capacity.png)

After reviewing the virtual machine configuration, I selected Finish.

![VM Configuration Review](images/VM%20review.png)

### Step 3 — Install Windows 10

After starting the virtual machine, I followed the Windows installation wizard.

![Windows 10 Setup](images/Windows%2010%20installation%20setup.png)

I selected Install Now.

![Windows Installation](images/Windows%2010%20installation%20setup%20install%20now.png)

For the operating system edition, I selected Windows 10 Pro.

![Windows Edition Selection](images/choose%20version.png)

I selected **Custom: Install Windows only (advanced)**.

![Custom Windows Installation](images/choose%20custom.png)

Next, I selected the virtual disk and allowed Windows to complete installation.

![Disk Configuration](images/disk%20info.png)

![Windows 10 Installation Progress](images/Windows%2010%20installing.png)

Once installation was complete, I configured a local user account and finished the initial Windows setup.

The workstation was later renamed `target-PC` and joined to the `adlab.local` domain.

<details>
<summary><strong>Additional Windows 10 installation screenshots</strong></summary>

![Windows License Terms](images/Windows%2010%20License%20terms.png)

![Windows Product Key](images/product%20key.png)

![License Agreement](images/license%20term.png)

</details>

---

## 3.2 Installing Kali Linux

Kali Linux was installed to serve as the security testing machine.

Instead of creating the machine from an installation ISO, I used Kali Linux's pre-built VMware image.

**Official Download:**

https://www.kali.org/get-kali/#kali-virtual-machines

### Step 1 — Download the Pre-Built VM

On Kali Linux's official website, I selected the VMware virtual machine image.

![Kali Linux Prebuilt VM](images/kali%20linux%20pre-built.png)

The image was downloaded in `.7z` format.

After downloading the archive, I extracted it using 7-Zip.

### Step 2 — Import Kali into VMware

I located the extracted VMware configuration file and opened it using VMware Workstation.

![Open Kali VM](images/open%20pre-built.png)

VMware automatically loaded the preconfigured machine.

![Kali Linux VMware](images/linux%20in%20VMware.png)

**What this does:** Using a pre-built VM provides a ready-to-use Kali environment with the operating system and common security utilities already configured.

After importing Kali, I configured its network adapter to use the same VMware NAT network as the other lab machines.

---

## 3.3 Installing Windows Server 2022

Windows Server 2022 was installed to act as the Active Directory Domain Controller.

**Official Download:**

https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022

### Step 1 — Download Windows Server

I downloaded the Windows Server 2022 ISO from Microsoft's evaluation center.

![Windows Server ISO](images/Windows%20Server%20iso%20page.png)

### Step 2 — Install Windows Server

Using VMware Workstation, I created a new virtual machine and attached the downloaded Windows Server ISO.

The installation process was similar to the Windows 10 setup.

I installed a Windows Server edition with Desktop Experience to manage the environment using Server Manager and other graphical administrative tools.

![Windows Server Installation](images/Windows%20Server%20installing.png)

### Step 3 — Configure Administrator Credentials

After installation, Windows Server prompted me to configure an Administrator password.

![Windows Server Administrator](images/Admin%20User%20creation%20on%20Server.png)

After configuring the account, I signed in and verified that Server Manager opened successfully.

![Windows Server Installed](images/Server%20installation%20complete.png)

The next steps involved configuring a static IP address, renaming the server, and installing Active Directory Domain Services.

---

## 3.4 Installing Ubuntu Server

Ubuntu Server was selected to host Splunk Enterprise.

**Official Download:**

https://ubuntu.com/server

### Step 1 — Download Ubuntu Server

I downloaded the Ubuntu Server ISO and created a new VMware virtual machine.

![Ubuntu Server Download](images/Ubuntu%20Server%20page.png)

### Step 2 — Install Ubuntu

After attaching the ISO, I started the VM and selected **Try or Install Ubuntu Server**.

![Ubuntu Server Installation](images/Ubuntu%20Server%20installation%20page.png)

During installation, I configured the server hostname, account information, and password.

![Ubuntu User Configuration](images/Ubuntu%20Server%20User%20account%20creation.png)

After installation, I restarted the server and signed in.

![Ubuntu Server Login](images/Splunk%20Server%20login%20page.png)

### Step 3 — Update Ubuntu Packages

Before installing Splunk, I updated Ubuntu.

```bash
sudo apt update
sudo apt upgrade -y
```

**Command Explanation:**

- `sudo apt update` refreshes package information from configured repositories.
- `sudo apt upgrade -y` installs available upgrades and automatically confirms the upgrade prompt.

Keeping the operating system updated helps address known security vulnerabilities and software issues.

![Ubuntu Update](images/splunk%20server%20updates.png)

![Ubuntu Package Upgrade](images/splunk%20server%20packet%20updates.png)

---

# 4. VMware Network Configuration

After installing the virtual machines, the next step was configuring network connectivity.

I used VMware Workstation's Virtual Network Editor to create a private lab network using NAT.

## 4.1 Configure VMware NAT

In VMware Workstation:

1. Open **Edit**.
2. Select **Virtual Network Editor**.
3. Select `VMnet8`.
4. Configure the network as NAT.
5. Enter the subnet address.
6. Configure the NAT gateway.
7. Save the configuration.

### Network Settings

```text
VMware Network: VMnet8
Network Type: NAT
Subnet: 192.168.10.0
Subnet Mask: 255.255.255.0
CIDR: /24
NAT Gateway: 192.168.10.2
```

![VMware Network Configuration](images/Virtual%20Network%20Configuration.png)

**What NAT does:** Network Address Translation allows virtual machines to access external networks through the host while keeping them on a separate virtual subnet.

NAT does not provide complete security isolation, so firewall rules and controlled testing boundaries remain important.

## 4.2 Connect Each VM to the NAT Network

For each virtual machine, I navigated to:

**Virtual Machine Settings → Network Adapter**

I selected **Custom: Specific virtual network** and chose `VMnet8`.

![Custom VMware Network](images/Set%20VM%20to%20custom%20Virtual%20Network.png)

Using the same virtual network allows the machines to communicate directly using their assigned private IP addresses.

---

## 4.3 Configuring the Splunk Server IP Address

The Splunk Enterprise server was assigned a static IP address.

A static IP is important because the Windows Universal Forwarders must send data to a known destination.

### Network Configuration

```text
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.2
DNS Server: 8.8.8.8
```

I opened Ubuntu's Netplan configuration file:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

The configuration used the following network settings:

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

The original Netplan configuration also included interface identification using a MAC address.

![Splunk Network Configuration](images/splunk%20server%20network%20configuration.png)

### Configuration Explanation

| Configuration | Purpose |
|---|---|
| `dhcp4: false` | Disables automatic IPv4 assignment |
| `addresses` | Assigns the server's static IP |
| `/24` | Defines the network prefix |
| `nameservers` | Defines DNS resolvers |
| `routes` | Defines network routes |
| `via` | Specifies the default gateway |

After saving the file, I applied the settings:

```bash
sudo netplan generate
sudo netplan apply
```

**What these commands do:** `netplan generate` validates and generates the network backend configuration, while `netplan apply` activates the configured network settings.

### Verify the IP Address

```bash
ip a
```

![Splunk IP Verification](images/ip%20address%20splunk%20server1.png)

![Splunk IP Updated](images/ip%20address%20splunk%20server2.png)

### Verify Connectivity

```bash
ping -c 4 192.168.10.2
ping -c 4 8.8.8.8
ping -c 4 google.com
```

![Splunk Ping Verification](images/splunk%20server%20ping.png)

The successful ping results confirmed that the Splunk Server had working network connectivity and DNS resolution.

---

## 4.4 Configuring Windows Server Networking

The Domain Controller was configured with a static IPv4 address.

```text
IP Address: 192.168.10.7
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.2
DNS Server: 192.168.10.7
```

![Windows Server Static IP](images/windows%20server%20ip%20change.png)

The Windows Server was renamed to:

```text
ADDC01
```

![Rename Windows Server](images/Windows%20Server%20name%20change.png)

After configuring the network settings, I tested connectivity.

![Windows Server Connectivity](images/Windows%20server%20ping.png)

> **Important:** The Domain Controller uses its own DNS service for Active Directory name resolution. Domain-joined Windows computers must be able to resolve the Active Directory domain through the Domain Controller's DNS infrastructure.

---

## 4.5 Configuring Windows 10 Networking

The Windows 10 machine was renamed to `target-PC`.

![Rename Windows 10](images/rename%20target%20pc.png)

The network settings were configured through the Ethernet adapter properties.

![Windows 10 Ethernet Configuration](images/Ethernet%20Target%20machine.png)

### Windows 10 Network Settings

```text
Computer Name: target-PC
IP Address: 192.168.10.100
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.2
DNS Server: 192.168.10.7
```

![Windows 10 Static IP](images/target%20machine%20ip%20change.png)

The DNS server was configured to use the Domain Controller so the workstation could locate Active Directory services.

<details>
<summary><strong>Additional Windows 10 network verification</strong></summary>

![Verify Target Machine Name](images/verify%20target%20machine%20name.png)

</details>

---

# 5. Splunk Enterprise Installation

Splunk Enterprise was installed on Ubuntu Server to serve as the centralized SIEM platform.

A SIEM collects and analyzes logs from multiple systems to provide visibility into security activity.

In this environment, Splunk receives Windows Security, System, Application, and Sysmon logs from the Windows endpoints.

## 5.1 Download Splunk Enterprise

I downloaded Splunk Enterprise from the official website.

**Website:** https://www.splunk.com/

The project used the Linux `.deb` installation package for Splunk Enterprise 10.6.1.1.

![Splunk Enterprise Download](images/Splunk%20enterprise%20image%20download.png)

---

## 5.2 Install VMware Tools

Before transferring the installer, I installed VMware Tools on Ubuntu.

```bash
sudo apt install open-vm-tools -y
```

I then enabled the service:

```bash
sudo systemctl enable --now open-vm-tools
```

Verify:

```bash
systemctl status open-vm-tools
```

![VMware Tools Verification](images/open-vm-tools%20verify%20splunk%20server.png)

**What VMware Tools does:** VMware Tools improves integration between the virtual machine and the VMware host, including system information reporting, virtual machine management, and other guest integration functions.

---

## 5.3 Install OpenSSH Server

To transfer the Splunk installation package from Windows to Ubuntu, I installed OpenSSH Server.

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

![SSH Service Status](images/systemctl%20status%20ssh.png)

### Configure the SSH Firewall Rule

If UFW is active, SSH can be permitted from the Windows VMware host.

```bash
sudo ufw allow from 192.168.10.1 to any port 22 proto tcp
```

![SSH Firewall Configuration](images/Allow%20SSH%20from%20Windows%20in%20Splunk%20Server.png)

**What this does:** Allows the VMware host to establish an SSH connection to Ubuntu over TCP port 22 without opening SSH access to the entire network.

---

## 5.4 Transfer the Splunk Installer

The Splunk `.deb` installer was downloaded to the Windows host computer.

I transferred the file using SCP.

In Windows PowerShell:

```powershell
cd F:\
```

Then:

```powershell
scp .\splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb rangeuser21@192.168.10.10:/home/rangeuser21/
```

![Transferring Splunk Installation File](images/splunk%20installation%20file%20transfer%20to%20VMware.png)

### Command Explanation

- `scp` securely transfers files using SSH.
- The source is the installation file stored on the Windows host.
- `rangeuser21` is the Ubuntu account used in the lab.
- `192.168.10.10` is the destination server.
- `/home/rangeuser21/` is the destination directory.

After transferring the file, I verified that it existed on Ubuntu.

```bash
ls -lh ~/splunk*.deb
```

![Verify Splunk Transfer](images/file%20transfer%20verify.png)

The package was successfully transferred and was ready for installation.

---

## 5.5 Install Splunk Enterprise

I installed Splunk using APT.

```bash
sudo apt install ~/splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb
```

![Splunk Installation](images/install%20splunk.png)

**What this command does:** Installs Splunk Enterprise from the downloaded Debian package and places the application files under `/opt/splunk/`.

After installation, I verified the package:

```bash
dpkg -s splunk | grep -E 'Status|Version'
```

A successful package installation reports:

```text
Status: install ok installed
```

### Important Splunk Directories

| Directory | Description |
|---|---|
| `/opt/splunk/bin/` | Splunk executables |
| `/opt/splunk/etc/` | Application configuration |
| `/opt/splunk/var/log/splunk/` | Internal operational logs |
| `/opt/splunk/var/lib/splunk/` | Indexed event storage |

![Splunk Directory](images/splunk%20directory%20users%20and%20groups.png)

### Splunk Storage

Splunk requires enough available disk space to store indexed events.

During setup, the VMware virtual disk was expanded to 40 GB and Ubuntu's LVM root filesystem was extended to use the additional space.

The relevant disk information was checked using:

```bash
lsblk
df -h
```

For the verified LVM layout in this project, the storage expansion involved:

```bash
sudo growpart /dev/sda 3
sudo pvresize /dev/sda3
sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
```

These commands are specific to the disk and LVM layout used in this lab.

> **Important:** Expanding a VMware virtual disk does not automatically resize the partitions or filesystems inside Ubuntu. The guest operating system must also be configured to recognize the additional capacity.

---

## 5.6 Start Splunk Enterprise

The documented configuration used a dedicated Linux service account named `splunk`.

I started Splunk with:

```bash
sudo -u splunk /opt/splunk/bin/splunk start --accept-license
```

![Start Splunk Enterprise](images/splunk%20server%20start%20with%20the%20user%20splunk.png)

The first startup process included configuring the Splunk administrator account.

**Why use a dedicated service account?** Running a service with its own account helps separate application permissions from privileged operating system accounts.

### Enable Startup

After confirming Splunk was working, boot-start functionality was configured so the service could start automatically when Ubuntu boots.

The documented lab used the Splunk service account rather than relying on a manual startup after every reboot.

---

## 5.7 Access Splunk Web

Once Splunk was running, I accessed the web interface from my Windows host.

```text
http://192.168.10.10:8000
```

![Splunk Enterprise Dashboard](images/splunk%20server%20dashboard.png)

### Important Splunk Ports

| Port | Protocol | Purpose |
|---|---|---|
| 8000 | TCP | Splunk Web Interface |
| 9997 | TCP | Receiving forwarded data |
| 22 | TCP | SSH administration |

Port `8000` provides access to the Splunk web interface.

Port `9997` is used by Splunk Enterprise to receive forwarded events when configured as a receiving port.

At this stage, Splunk Enterprise was successfully installed and ready to receive endpoint logs.

---

# 6. Splunk Universal Forwarder and Sysmon

After configuring Splunk Enterprise, I installed Splunk Universal Forwarder and Microsoft Sysmon on Windows 10 and Windows Server 2022.

These two tools work together to provide endpoint telemetry to Splunk.

### How They Work Together

```text
                WINDOWS ENDPOINT
                      |
                      v
                 MICROSOFT SYSMON
                      |
                      | Generates detailed events
                      v
                WINDOWS EVENT LOGS
                      |
                      v
             SPLUNK UNIVERSAL FORWARDER
                      |
                      | TCP 9997
                      v
                SPLUNK ENTERPRISE
                      |
                      v
              SECURITY INVESTIGATION
```

**Sysmon generates detailed endpoint events, while Universal Forwarder collects the configured Windows event logs and sends them to Splunk.**

---

## 6.1 Download Splunk Universal Forwarder

I downloaded the Windows x64 version of Universal Forwarder from Splunk.

**Official Download:**

https://www.splunk.com/en_us/download/universal-forwarder.html

![Universal Forwarder Download](images/splunk%20universal%20forwarder%20image%20download.png)

I launched the installer on Windows 10.

![Splunk Forwarder Installation](images/splunk%20universal%20forwarder%20setup%20start.png)

During installation, I selected the on-premises Splunk Enterprise configuration and set up the receiving destination.

### Indexer Configuration

```text
Receiving Server: 192.168.10.10
Receiving Port: 9997
```

![Universal Forwarder Destination](images/splunk%20universal%20forwarder%20splunk%20server%20ip.png)

The Deployment Server field was left blank because a separate Splunk Deployment Server was not used in the lab.

<details>
<summary><strong>Additional Universal Forwarder setup</strong></summary>

![Universal Forwarder Setup](images/splunk%20universal%20forwarder%20setup%20.png)

</details>

---

## 6.2 Install Microsoft Sysmon

Microsoft Sysmon was installed to provide detailed information about endpoint activity.

**Microsoft Download:**

https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Sysmon can record events including:

- Process creation
- Network connections
- File creation
- Registry changes
- DNS queries
- Process-access activity
- Image and DLL loading

### Sysmon Configuration

For this project, I used Olaf Hartong's Sysmon Modular configuration.

**Repository:**

https://github.com/olafhartong/sysmon-modular

![Olaf Hartong Configuration](images/Olaf%20configuration%20file%20link.png)

![Sysmon Modular Configuration](images/Olaf%20configuration.png)

The modular configuration includes security-focused filtering rules to collect relevant events and reduce unnecessary logging noise.

I selected the balanced configuration compatible with the installed Sysmon version.

### Extract Sysmon

After downloading Sysmon, I extracted the ZIP archive.

![Sysmon Extracted](images/sysmon%20file%20extracted.png)

### Install Sysmon Using PowerShell

I opened PowerShell as Administrator and ran the following command from the directory containing `Sysmon64.exe` and the selected XML configuration:

```powershell
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

![Sysmon Installation](images/sysmon%20installation.png)

**What this command does:** Installs Microsoft Sysmon and applies the specified XML configuration to determine which system activities should be logged.

### Verify Sysmon

```powershell
Get-Service Sysmon64
```

Sysmon events can also be viewed through:

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
| 1 | Process Creation | Suspicious commands and executables |
| 3 | Network Connection | Suspicious network activity |
| 7 | Image Loaded | DLL and image loading |
| 10 | Process Access | Potential credential-access activity |
| 11 | File Creation | Suspicious file creation |
| 12 | Registry Object Created/Deleted | Registry monitoring |
| 13 | Registry Value Set | Persistence-related changes |
| 22 | DNS Query | Suspicious domains |
| 25 | Process Tampering | Process manipulation |

> **Why Sysmon matters:** Standard Windows Security logs provide important authentication and audit information, while Sysmon adds more detailed visibility into processes, file activity, registry activity, and network connections.

---

## 6.3 Configure Splunk Universal Forwarder Inputs

Universal Forwarder must be configured to monitor the required Windows event logs.

On Windows, I opened Notepad as Administrator.

![Notepad Administrator](images/open%20notepad%20as%20admin.png)

The configuration file was saved to:

```text
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

### inputs.conf Configuration

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

![Save inputs.conf](images/save%20the%20inputs.conf%20file.png)

### Configuration Explanation

| Configuration | Purpose |
|---|---|
| `WinEventLog://Application` | Collect application logs |
| `WinEventLog://Security` | Collect Windows Security logs |
| `WinEventLog://System` | Collect operating system logs |
| `WinEventLog://Microsoft-Windows-Sysmon/Operational` | Collect Sysmon telemetry |
| `index = endpoint` | Send events to the endpoint index |
| `disabled = 0` | Enable the input |
| `renderXml = true` | Preserve Sysmon events as XML |
| `source = XmlWinEventLog:...` | Assign the configured Sysmon source |

### Restart Universal Forwarder

After modifying `inputs.conf`, I restarted the Forwarder service.

```powershell
Restart-Service SplunkForwarder
```

**What this command does:** Restarts Splunk Universal Forwarder so it reloads its configuration and begins collecting the specified event sources.

Verify the service:

```powershell
Get-Service SplunkForwarder
```

![Universal Forwarder Service](images/splunk%20fordwarder%20service%20check.png)

<details>
<summary><strong>Additional Forwarder service verification</strong></summary>

![Forwarder Verification](images/splunk%20fordwarder%20service1.png)

![Forwarder Configuration](images/splunk%20fordwarder%20service2.png)

</details>

The same monitoring configuration was applied to Windows Server 2022.

---

## 6.4 Create the Splunk Endpoint Index

After configuring the Forwarders, I created a dedicated index named `endpoint`.

In Splunk Enterprise:

**Settings → Indexes → New Index**

The index name was:

```text
endpoint
```

![Create Splunk Index](images/create%20a%20new%20index.png)

**What an index does:** A Splunk index stores searchable event data. Organizing Windows security telemetry into a dedicated index makes it easier to manage and investigate endpoint logs.

---

## 6.5 Enable Splunk Receiving

Next, I enabled receiving on TCP port `9997`.

In Splunk Enterprise:

**Settings → Forwarding and Receiving → Configure Receiving**

![Splunk Forwarding and Receiving](images/forwarding%20and%20receiving.png)

I added the receiving port:

```text
9997
```

![Splunk Receiving Port](images/forwarding%20and%20receiving%20add%20the%20port%20number.png)

**What this does:** Allows Splunk Enterprise to receive event data sent by the Universal Forwarder instances on the Windows machines.

### Verify Forwarder Connectivity

From Windows PowerShell:

```powershell
Test-NetConnection 192.168.10.10 -Port 9997
```

A successful result includes:

```text
TcpTestSucceeded : True
```

---

## 6.6 Verify Windows Log Collection

After configuring Splunk and the Forwarders, I verified that events were arriving.

In Splunk Search & Reporting:

```spl
index=endpoint
```

I then used:

```spl
index=endpoint
| stats count by host source
```

**What this query does:** Counts events by originating host and data source, allowing me to verify which endpoints are forwarding logs.

![Splunk Host Search](images/splunk%20search%20query%20host%20check.png)

After configuring both Windows machines, Splunk displayed events from:

- `target-PC`
- `ADDC01`

![Splunk Both Hosts](images/splunk%20query%20host%20update.png)

The received log sources included:

```text
WinEventLog:Security
WinEventLog:System
WinEventLog:Application
XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

![Splunk Event Sources](images/splunk%20index%20source%20check.png)

This confirmed that the centralized log collection pipeline was working.

![Splunk Web from Windows Target](images/target%20machine%20splunk.png)

---

# 7. Active Directory Domain Services

After configuring the security monitoring infrastructure, I installed Active Directory Domain Services on Windows Server 2022.

Active Directory provides centralized identity management, authentication, authorization, and administrative control over domain resources.

## 7.1 Install Active Directory Domain Services

On Windows Server, I opened Server Manager and selected:

**Manage → Add Roles and Features**

![Add Roles and Features](images/add%20roles%20and%20features.png)

I selected **Role-based or feature-based installation**.

![Role-Based Installation](images/role-based%20or%20feature-based.png)

Next, I selected the local Windows Server.

![Destination Server](images/select%20destination%20server.png)

Under Server Roles, I selected **Active Directory Domain Services**.

![Select Server Roles](images/Select%20server%20roles.png)

![Active Directory Role](images/active%20directory%20domain%20services%20selected.png)

**What AD DS does:** Active Directory Domain Services provides centralized management of users, computers, groups, authentication, and access to domain resources.

After completing the role installation, I promoted the server to a Domain Controller.

---

## 7.2 Promote Windows Server to a Domain Controller

In Server Manager, I selected:

**Promote this server to a domain controller**

![Domain Controller Promotion](images/AD-post-deployment%20configuration.png)

Because the environment did not contain an existing domain, I selected:

**Add a new forest**

The domain name was:

```text
adlab.local
```

![Create Active Directory Forest](images/Add%20a%20new%20forest.png)

### Configure the Domain Controller

During the promotion process, I configured the Directory Services Restore Mode password.

![Directory Services Restore Mode](images/Forest%20password%20set.png)

The wizard then displayed the Active Directory database and related directory paths.

![Active Directory Database Paths](images/AD%20paths.png)

### Important Active Directory File — NTDS.dit

One important file associated with Active Directory is:

```text
NTDS.dit
```

**What NTDS.dit does:** This database stores Active Directory directory information, including sensitive account data and password-related information.

Because the database contains sensitive data, unauthorized access or attempts to extract it can represent a serious security risk.

Monitoring suspicious activity involving Active Directory services and associated database files is important in enterprise environments.

---

## 7.3 Complete Domain Controller Installation

Before installation, I reviewed the prerequisite checks.

![Active Directory Prerequisites](images/Prerequisites%20check.png)

After the checks passed, I selected Install.

![Active Directory Installation Complete](images/Active%20directory%20installation%20complete.png)

The server restarted after being promoted to a Domain Controller.

I then signed in using the domain Administrator account.

![AD Domain Administrator Login](images/ADLAB-Administrator%20logon.png)

### Final Domain Configuration

```text
Domain Name: adlab.local
NetBIOS Name: ADLAB
Domain Controller: ADDC01
IP Address: 192.168.10.7
```

At this point, Active Directory Domain Services and DNS were configured for the HomeLab.

---

# 8. Active Directory Users and Organizational Units

The next step was creating Organizational Units and domain user accounts.

Organizational Units provide a logical structure for organizing Active Directory objects.

They can also be used to apply Group Policy and delegate administrative responsibilities.

## 8.1 Open Active Directory Users and Computers

From Server Manager:

**Tools → Active Directory Users and Computers**

![Active Directory Users and Computers](images/AD%20Users%20and%20computers.png)

I reviewed the built-in Active Directory containers and security groups.

![Built-in Active Directory Groups](images/User%20and%20computers%20Builtins%20Groups.png)

---

## 8.2 Create Organizational Units

I created Organizational Units to represent departments within the simulated organization.

For example:

```text
adlab.local
|
+--- IT
|
+--- HR
```

To create an OU:

1. Open Active Directory Users and Computers.
2. Right-click the domain.
3. Select **New → Organizational Unit**.
4. Enter the OU name.
5. Click OK.

![Create IT Organizational Unit](images/IT%20OU%20Creation.png)

**What Organizational Units do:** OUs organize Active Directory objects and provide a way to apply policies or administrative permissions to specific departments or groups of systems.

---

## 8.3 Create Domain Users

After creating the Organizational Units, I created domain user accounts.

The documented environment included users such as Jenny Smith and Terry Smith for domain authentication and testing.

To create a user:

1. Right-click the selected Organizational Unit.
2. Select **New → User**.
3. Enter the user's name.
4. Configure the user logon name.
5. Assign a password.
6. Configure the account options.
7. Finish the wizard.

![Active Directory User Creation](images/User%20Creation.png)

Next, I configured the account password.

![Domain User Password](images/User%20Password%20set.png)

Test accounts were used only within the virtualized environment.

In a production environment, password policies, account lockout thresholds, least privilege, and other account protections should be enforced.

---

## 8.4 Verify Account and Group Membership

After creating the domain users, I verified they appeared in the correct Organizational Units.

![Verify Active Directory User](images/User%20group%20verify.png)

![Verify Additional Account](images/User%20group%20verify2.png)

**Why this matters:** User and group membership determines which resources an account can access. From a security perspective, excessive permissions or incorrect group assignments can increase the impact of a compromised account.

---

# 9. Joining Windows 10 to Active Directory

After configuring the Domain Controller and domain users, I joined the Windows 10 workstation to the Active Directory domain.

Joining the domain allows Windows 10 users to authenticate using domain accounts and access domain-managed resources.

## 9.1 Open Windows System Properties

On Windows 10, I opened:

**This PC → Properties → Advanced System Settings**

![Windows 10 System Properties](images/W10%20System%20properties.png)

Under the Computer Name tab, I selected Change.

---

## 9.2 Configure Domain Membership

I selected the Domain option and entered:

```text
adlab.local
```

![Windows 10 Domain Join](images/W10%20join%20domain.png)

During the initial attempt, the workstation could not locate the domain.

![Domain Join Error](images/Join%20domain%20error.png)

The workstation needed to resolve the Active Directory domain using the Domain Controller's DNS server.

I configured the Windows 10 preferred DNS server to:

```text
192.168.10.7
```

![Configure Windows DNS](images/Add%20DNS%20server%20as%20Windows%20Server%20ip.png)

### Verify DNS Configuration

```powershell
ipconfig /all
```

![Verify Windows Network Configuration](images/ipconfig%20all.png)

**What this command does:** Displays the Windows network configuration, including assigned IP addresses, default gateways, DNS servers, and adapter information.

To verify Domain Controller discovery, I also used:

```powershell
nltest /dsgetdc:adlab.local
```

**What this command does:** Attempts to locate a Domain Controller for the specified Active Directory domain.

---

## 9.3 Authenticate and Join the Domain

After configuring DNS, I attempted the domain join again.

Windows prompted for domain credentials.

![Domain Credentials](images/Credentials%20for%20Domain%20change.png)

After authenticating with an authorized administrative account, Windows successfully joined the domain.

![Successfully Joined Domain](images/Domain%20joined.png)

The workstation was restarted to complete the domain configuration.

### Final Configuration

```text
Computer Name: target-PC
Domain: adlab.local
Domain Controller: ADDC01
DNS Server: 192.168.10.7
```

The Windows 10 workstation was now part of the Active Directory environment.

> **Why this matters:** Domain joining allows centralized authentication and makes the workstation subject to applicable Active Directory policies and administrative controls.

---

# 10. Kali Linux Configuration

With the Active Directory environment operational, I configured Kali Linux for authorized security testing.

Kali Linux was assigned a static IP address on the same VMware NAT network.

## 10.1 Configure the Kali Network

In Kali Linux, I opened **Advanced Network Configuration**.

![Kali Network Settings](images/kali%20linux%20network.png)

I selected Wired Connection 1 and opened its settings.

![Kali Wired Network](images/kali%20linux%20network%20connection.png)

Under IPv4 Settings, I changed the method from Automatic (DHCP) to Manual.

### Network Configuration

```text
IP Address: 192.168.10.250
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.2
DNS Server: 192.168.10.7
```

![Kali Static IP Configuration](images/kali%20linux%20network%20configuration.png)

---

## 10.2 Verify the Kali IP Address

I opened a terminal and ran:

```bash
ip a
```

Initially, Kali was still displaying the previous address.

![Kali IP Before Reconnection](images/kali%20linux%20ip%20a1.png)

I disconnected and reconnected the network interface to apply the updated configuration.

After reconnecting, Kali displayed the correct static IP address.

![Kali Static IP Verified](images/kali%20linux%20ip%20a2.png)

---

## 10.3 Verify Network Connectivity

I tested connectivity with the Splunk Server:

```bash
ping -c 4 192.168.10.10
```

I also tested the Windows 10 target:

```bash
ping -c 4 192.168.10.100
```

![Kali Ping Results](images/kali%20linux%20ping.png)

**What these commands do:** Send four ICMP echo requests to determine whether the destination machine responds to ping.

Successful ping responses demonstrate ICMP connectivity. They do not independently prove that every application or port is accessible.

---

# 11. RDP Authentication Testing with NetExec

One of the main security exercises involved performing controlled RDP authentication testing against the Windows 10 workstation.

The purpose was to generate Windows authentication events that could be collected and investigated using Splunk.

This test was performed only against the authorized Windows 10 virtual machine using a designated lab account.

## 11.1 Enable Remote Desktop on Windows 10

Before testing authentication, Remote Desktop was enabled on the Windows 10 workstation.

I opened:

**System Properties → Remote**

Then selected:

**Allow remote connections to this computer**

Network Level Authentication was kept enabled.

![Enable Windows Remote Desktop](images/allow%20remote%20connection.png)

Next, I selected the domain user authorized to connect through Remote Desktop.

![Remote Desktop Users](images/add%20users%20in%20remote%20connection.png)

### RDP Port

Remote Desktop Protocol commonly uses:

```text
TCP Port: 3389
```

To verify the port from Kali Linux:

```bash
nmap -Pn -p 3389 192.168.10.100
```

The scan confirmed that RDP was accessible on the Windows 10 target.

**What this command does:** Checks the specified target for an accessible TCP service on port 3389 without requiring ICMP host discovery.

---

## 11.2 Install NetExec

For the authentication testing exercise, I used NetExec.

NetExec is an open-source security assessment tool that supports authentication testing across multiple network protocols.

**Official Documentation:**

https://www.netexec.wiki/

I installed NetExec using:

```bash
sudo apt update
sudo apt install netexec -y
```

![Install NetExec](images/kali%20linux%20apt%20update%20and%20install%20netexec.png)

Verify installation:

```bash
nxc --version
```

![NetExec Version](images/netexec%20version.png)

To view the available options:

```bash
nxc --help
```

![NetExec Help](images/netexec%20help.png)

**What NetExec does:** NetExec allows authorized security professionals to evaluate authentication and access behavior across supported network protocols.

---

## 11.3 Prepare the Authentication Test

I created a project directory in Kali Linux to organize the files used for the authentication exercise.

The lab used a small password list based on a limited selection of entries from Kali's RockYou wordlist.

The goal was to simulate repeated authentication attempts against a disposable test account rather than perform unrestricted testing.

### Password File Preparation

The original wordlist was available under Kali Linux's wordlist directory.

![Wordlist Preparation](images/unzip%20the%20password%20file.png)

For the controlled test, I selected a small subset of entries and created a separate file named:

```text
passwords.txt
```

![Wordlist Preview](images/Get%20first%2020%20password.png)

I also verified that the password file was accessible to the test environment.

![Password File Permissions](images/password%20file%20permissions.png)

> **Important:** Wordlists and valid passwords should not be committed to public repositories. The lab used a disposable domain account, and the exact test password is intentionally omitted from this documentation.

---

## 11.4 Execute the Controlled RDP Authentication Test

The documented test used:

```bash
nxc -t 1 --jitter 3 rdp 192.168.10.100 -d ADLAB -u tsmith -p passwords.txt
```

### Command Breakdown

| Parameter | Description |
|---|---|
| `nxc` | Launch NetExec |
| `-t 1` | Use one concurrent worker |
| `--jitter 3` | Space out connection attempts |
| `rdp` | Use the Remote Desktop protocol |
| `192.168.10.100` | Windows 10 target |
| `-d ADLAB` | Active Directory domain |
| `-u tsmith` | Lab account used for testing |
| `-p passwords.txt` | Password list used in the controlled exercise |

**What this command does:** Tests credentials from the lab password file against the designated Windows 10 RDP service and reports authentication results.

Repeated authentication testing can trigger account lockouts, so it should be limited to authorized test accounts and controlled environments.

### Authentication Results

During testing, NetExec reported failed authentication attempts using:

```text
STATUS_LOGON_FAILURE
```

This means the supplied credentials were rejected by the authentication process.

The tool also identified a valid lab credential during the authorized test.

The successful credential validation was subsequently investigated in Splunk.

> **Evidence note:** The original NetExec screenshot contains a working lab password. It should be redacted before being displayed in a public repository.

<!--
After redacting the password from the original image, enable:

![NetExec Authentication Results](images/netexec%20password%20brute%20force%20attack.png)
-->

---

## 11.5 Investigate the Authentication Activity in Splunk

After completing the test, I opened Splunk Enterprise.

I navigated to:

**Apps → Search & Reporting**

The Windows Security events were available in the `endpoint` index.

One important event associated with the exercise was:

```text
Event ID: 4625
Description: An account failed to log on
```

![Windows Event ID 4625](images/EventID%204625.png)

I searched for failed login activity associated with the test account.

```spl
index=endpoint "tsmith" EventCode=4625
```

**What this query does:** Searches for events containing the specified lab account and Windows Event ID 4625 in the endpoint index.

![Splunk Authentication Investigation](images/splunk%20query%20after%20attack.png)

The captured investigation showed **60 failed login events** in the selected search results.

This count represents the events visible in that particular filtered view.

I also examined the event details to identify the source of the authentication activity.

![Investigate Authentication Source IP](images/Splunk%20check%20ip%20of%20the%20attacker.png)

The source information helped correlate the authentication activity with the Kali Linux testing machine.

### Security Significance

Repeated failed authentication events against the same account may indicate:

- Password-guessing attempts
- Unauthorized login activity
- Misconfigured services
- Credential misuse
- Automated authentication attempts

A successful credential validation following repeated failures can be particularly important during a security investigation.

However, authentication events must be correlated with other information before determining whether malicious access occurred.

---

# 12. Security Event Investigation with Splunk

After the controlled authentication exercise, I continued analyzing events from Windows 10 and the Domain Controller.

The monitoring environment provided visibility into both endpoint activity and Active Directory authentication events.

## 12.1 Windows Event ID 4625 — Failed Logon

Event ID 4625 is generated when Windows records a failed logon.

```spl
index=endpoint source="WinEventLog:Security" EventCode=4625
| table _time host _raw
| sort -_time
```

### Query Explanation

| SPL Component | Purpose |
|---|---|
| `index=endpoint` | Select the endpoint index |
| `source="WinEventLog:Security"` | Limit results to Windows Security logs |
| `EventCode=4625` | Search for failed logons |
| `table` | Display selected fields |
| `sort -_time` | Show newest events first |

### Important Event Fields

| Field | Description |
|---|---|
| Account Name | Account involved in the authentication |
| Account Domain | Domain associated with the account |
| Logon Type | Type of authentication attempt |
| Source Network Address | Source IP when available |
| Status | Authentication failure status |
| Sub Status | Additional failure information |

These details are useful when analyzing suspicious authentication behavior.

---

## 12.2 Windows Event ID 4624 — Successful Logon

Event ID 4624 records successful Windows logons.

```spl
index=endpoint source="WinEventLog:Security" EventCode=4624
| table _time host _raw
| sort -_time
```

### Important Logon Types

| Logon Type | Description |
|---|---|
| 2 | Interactive |
| 3 | Network |
| 5 | Service |
| 10 | RemoteInteractive |
| 11 | CachedInteractive |

**Why this matters:** A successful logon event does not always represent a user signing in at the desktop. Analysts must examine the logon type and associated information to understand what occurred.

Logon Type 10 is especially relevant when investigating completed Remote Desktop logons.

---

## 12.3 Windows Event ID 4776 — NTLM Credential Validation

Event ID 4776 records NTLM credential validation activity.

The Domain Controller logged NTLM authentication results associated with the test account and Kali workstation.

Important status codes included:

| Error Code | Meaning |
|---|---|
| `0x0` | Credentials validated successfully |
| `0xC000006A` | Incorrect password |
| `0xC0000234` | Account locked out |

### Investigate NTLM Events

```spl
index=endpoint host=ADDC01 source="WinEventLog:Security" EventCode=4776
| table _time host _raw
| sort -_time
```

**What this query does:** Retrieves NTLM credential validation events collected from the Active Directory Domain Controller.

### Extract Authentication Results

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

### Command Explanation

- `rex` extracts the error code from the raw log.
- `eval` creates a new result classification.
- `case` evaluates the error code and assigns a readable status.
- `stats count by Result` counts the events in each category.

This makes it easier to summarize successful and unsuccessful credential validations.

> **Important:** A successful Event ID 4776 means the credentials were accepted by NTLM validation. It does not independently prove that a complete Remote Desktop session was established.

---

## 12.4 Sysmon Event Investigation

After verifying the Windows Security logs, I examined the Sysmon telemetry.

The Sysmon events were collected in XML format.

To search for all Sysmon events:

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
```

### Extract Sysmon Event IDs

The `EventCode` field was not always extracted in the expected format from the XML events.

I used the following query to extract the Event ID directly from the raw XML:

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| stats count by host SysmonEventID
| sort -count
```

**What this query does:** Extracts the Sysmon Event ID from XML data and counts the occurrences by host and event type.

This allows security analysts to identify which types of Sysmon events are being generated across endpoints.

### Important Sysmon Events

| Event ID | Description |
|---|---|
| 1 | Process Creation |
| 3 | Network Connection |
| 7 | Image Loaded |
| 11 | File Creation |
| 13 | Registry Value Set |
| 22 | DNS Query |

These events can help identify suspicious command execution, unusual file activity, registry modifications, and network behavior.

---

# 13. Atomic Red Team Simulation

After completing the RDP authentication exercise, I used Atomic Red Team to simulate additional security-relevant behavior.

Atomic Red Team is an open-source project maintained by Red Canary.

It contains tests associated with MITRE ATT&CK techniques and is commonly used for evaluating security monitoring and detection capabilities.

**Official Repository:**

https://github.com/redcanaryco/atomic-red-team

## 13.1 Purpose of Atomic Red Team

Atomic Red Team allows security professionals to generate controlled activity associated with known adversary techniques.

In this lab, it was used to evaluate whether Sysmon and Splunk could capture relevant endpoint events.

### Testing Workflow

```text
           ATOMIC RED TEAM
                  |
                  | Execute Selected Technique
                  |
                  v
          WINDOWS 10 TARGET
                  |
                  | Windows / Sysmon Events
                  |
                  v
          SPLUNK UNIVERSAL
              FORWARDER
                  |
                  v
          SPLUNK ENTERPRISE
                  |
                  v
           EVENT INVESTIGATION
```

> **Why this matters:** Controlled attack simulation helps defenders validate whether their monitoring tools can observe specific types of activity. This approach supports purple-team exercises and detection engineering.

---

## 13.2 Install Atomic Red Team

The tests were configured on the Windows 10 target.

I opened PowerShell as Administrator.

The original lab setup included adjusting PowerShell execution policy for the testing environment.

![PowerShell Execution Policy](images/Set-Exectution%20Policy.png)

PowerShell execution policies should be configured carefully and should not be treated as a security boundary.

### Download and Install

The documented installation process used the Invoke-AtomicRedTeam framework.

**Repository:**

https://github.com/redcanaryco/invoke-atomicredteam

The installation commands were:

```powershell
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
```

```powershell
Install-AtomicRedTeam -getAtomics
```

**What these commands do:** Retrieve and run the Invoke-AtomicRedTeam installer and download the Atomic Red Team test definitions.

Installation scripts retrieved from the internet should be reviewed before execution, and tests should be performed only in an authorized environment.

![Atomic Red Team Installation](images/install%20Atomic%20Red%20Team.png)

After installation, I inspected the Atomic Red Team directory.

![Atomic Red Team Directory](images/Atomic%20Red%20Team%20folder.png)

The directory contained technique folders organized according to MITRE ATT&CK identifiers.

![Atomic Red Team Files](images/Atomic%20Red%20Team%20folder%20run.png)

---

## 13.3 Understanding MITRE ATT&CK

MITRE ATT&CK is a knowledge base of adversary tactics and techniques based on observed behavior.

It is used by security professionals for threat detection, adversary simulation, and defensive security research.

**Framework:**

https://attack.mitre.org/

![MITRE ATTACK Framework](images/MITRE%20ATT%26CK.png)

Each ATT&CK technique has an identifier.

For example:

| Technique ID | Technique |
|---|---|
| T1136.001 | Create Account: Local Account |
| T1059.001 | Command and Scripting Interpreter: PowerShell |

![MITRE Technique Selection](images/Testing%20differnt%20techniques%20ID.png)

The project tested selected techniques to generate security events that could be investigated using Splunk.

---

## 13.4 Atomic Test — T1136.001

### Create Account: Local Account

**MITRE ATT&CK Technique:** T1136.001

This technique involves creating a local account on a Windows system.

Adversaries may abuse account creation to establish additional access or maintain persistence.

### Execute the Test

The documented test command was:

```powershell
Invoke-AtomicTest T1136.001
```

**What this command does:** Executes the Atomic Red Team test or tests configured for technique T1136.001, generating activity associated with local account creation.

The exact actions depend on the test definition and selected test number.

### Investigate in Splunk

After executing the test, I searched the endpoint index for related activity.

```spl
index=endpoint "NewLocalUser"
```

The documented search returned **14 raw events** associated with the activity.

![Atomic Red Team Local Account Test](images/splunk%20check%20after%20Atomic%20Red%20Team%20run.png)

### Security Significance

Unauthorized local account creation can be associated with persistence or access maintenance.

Potential indicators include:

- Unexpected new local accounts
- Account creation outside approved administrative activity
- Suspicious account names
- Unusual privileged process activity
- New accounts added to privileged groups

Account creation should be investigated in context to determine whether it was authorized.

---

## 13.5 Atomic Test — T1059.001

### Command and Scripting Interpreter: PowerShell

**MITRE ATT&CK Technique:** T1059.001

PowerShell is a legitimate Windows administration tool that supports automation and system configuration.

However, attackers can also use PowerShell for execution, discovery, and other malicious activities.

### Execute the Test

The documented lab test used:

```powershell
Invoke-AtomicTest T1059.001
```

**What this command does:** Executes Atomic Red Team test activity associated with the PowerShell technique.

After execution, I used Splunk to investigate the generated telemetry.

### Search PowerShell Activity

```spl
index=endpoint powershell
```

The collected logs included activity involving `powershell.exe` and registry-related events.

One observed event referenced:

```text
HKLM\System\CurrentControlSet\Services\bam\State\UserSettings
```

This path is associated with Windows Background Activity Moderator (BAM).

BAM tracks certain application execution activity.

The event demonstrated that Sysmon was recording relevant endpoint activity during the Atomic Red Team exercise.

![PowerShell Atomic Red Team Investigation](images/Splunk%20check%20after%20running%20Powershell%20Atomic%20Red%20team.png)

### Security Significance

PowerShell activity can be useful for detecting suspicious execution.

Important details to investigate include:

- PowerShell command lines
- Parent and child processes
- Executing user accounts
- Unusual script locations
- Associated file activity
- Registry modifications
- Network connections

A PowerShell process or BAM event alone does not prove malicious behavior. The complete process context and related telemetry should be reviewed.

---

# 14. SPL Threat Detection Queries

Splunk Search Processing Language (SPL) allows security analysts to search, filter, correlate, and summarize machine-generated events.

The following queries demonstrate how the collected Windows and Sysmon logs can support additional detection and investigation activities.

These queries are based on the configured HomeLab data sources and serve as reusable investigation examples.

---

## Detection 1 — Failed Windows Authentication

```spl
index=endpoint source="WinEventLog:Security" EventCode=4625
| stats count as FailedLogons by host
| sort -FailedLogons
```

**Purpose:** Counts failed Windows logon events by host.

This can help identify systems experiencing unusually high authentication failure volumes.

---

## Detection 2 — NTLM Incorrect Password Attempts

```spl
index=endpoint host=ADDC01 source="WinEventLog:Security" EventCode=4776
| rex field=_raw "Logon Account:\s+(?<Account>\S+)"
| rex field=_raw "Source Workstation:\s+(?<Workstation>[^\r\n]+)"
| rex field=_raw "Error Code:\s+(?<AuthStatus>0x[0-9A-Fa-f]+)"
| where lower(AuthStatus)="0xc000006a"
| stats count as FailedAttempts by Account Workstation
| sort -FailedAttempts
```

**Purpose:** Identifies accounts and workstations associated with rejected NTLM credentials.

Repeated authentication failures involving the same account or source system can indicate suspicious login activity.

---

## Detection 3 — Multiple Failures Followed by Success

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

**Purpose:** Identifies successful NTLM credential validation following five or more incorrect-password attempts within ten minutes for the same account and source workstation.

### Why This Detection Is Important

An attacker may attempt several passwords against a user account before discovering a valid credential.

A sequence of repeated failures followed by a successful authentication can indicate potential password-guessing activity.

However, the result should be correlated with additional Windows events, user activity, and source information before determining whether compromise occurred.

---

## Detection 4 — Sysmon Process Creation

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| where SysmonEventID="1"
| table _time host _raw
| sort -_time
```

**Purpose:** Identifies Sysmon Event ID 1 process creation events.

Process creation events can provide useful information about executed programs, command lines, and process relationships.

---

## Detection 5 — Sysmon Event Distribution

```spl
index=endpoint source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID[^>]*>(?<SysmonEventID>\d+)</EventID>"
| stats count by host SysmonEventID
| sort -count
```

**Purpose:** Counts different Sysmon event types by Windows host.

This can help identify which event categories produce the most telemetry.

---

## Detection 6 — Windows Authentication Timeline

```spl
index=endpoint source="WinEventLog:Security"
(EventCode=4624 OR EventCode=4625 OR EventCode=4771 OR EventCode=4776 OR EventCode=4740)
| table _time host EventCode _raw
| sort -_time
```

**Purpose:** Creates a timeline of relevant Windows and Active Directory authentication events.

This allows analysts to examine related logons, authentication failures, credential validations, and account lockouts.

---

## Detection 7 — Investigate a Specific Account

```spl
index=endpoint source="WinEventLog:Security" "tsmith"
| table _time host EventCode _raw
| sort -_time
```

**Purpose:** Retrieves Windows Security events referencing the specified lab account.

This can be useful when investigating unusual activity associated with a particular user.

---

## Detection 8 — Verify Endpoint Log Collection

```spl
index=endpoint
| stats count by host source
| sort -count
```

**Purpose:** Confirms that monitored endpoints continue forwarding event data to Splunk.

This is useful for validating log collection coverage.

---

# 15. Project Results and Findings

The completed HomeLab successfully demonstrated centralized security monitoring across an Active Directory environment.

Windows Server 2022 and Windows 10 were configured to forward Windows Security, System, Application, and Sysmon events to Splunk Enterprise.

The environment was also used to perform controlled authentication testing and simulate selected adversary techniques.

## 15.1 Completed Project Components

| Component | Result |
|---|---|
| VMware Workstation Environment | Completed |
| Virtual NAT Network | Configured |
| Windows Server 2022 | Installed |
| Windows 10 Workstation | Installed |
| Ubuntu Splunk Server | Installed |
| Kali Linux | Installed |
| Active Directory Domain Services | Configured |
| Active Directory Users and OUs | Created |
| Windows 10 Domain Membership | Successful |
| Splunk Enterprise | Operational |
| Splunk Universal Forwarder | Operational |
| Sysmon Endpoint Monitoring | Operational |
| Centralized Windows Log Collection | Verified |
| NetExec RDP Authentication Testing | Completed |
| Authentication Event Investigation | Completed |
| Atomic Red Team Testing | Documented |
| SPL Security Queries | Executed and Documented |

---

## 15.2 Authentication Monitoring Results

The security monitoring environment recorded failed and successful Windows authentication activity.

### Important Windows Events

| Event ID | Description | Relevance |
|---|---|---|
| 4624 | Successful Logon | Authentication monitoring |
| 4625 | Failed Logon | Password-guessing investigation |
| 4771 | Kerberos Pre-Authentication Failed | Kerberos authentication investigation |
| 4776 | NTLM Credential Validation | Domain authentication monitoring |
| 4740 | Account Locked Out | Account lockout investigation |

The documented Splunk investigation included a filtered result containing **60 failed logon events (Event ID 4625)**.

The Domain Controller also recorded NTLM credential validation events associated with the test account.

Successful and unsuccessful validation results were differentiated using their status codes.

This demonstrated that the monitoring pipeline was capable of recording authentication activity relevant to security investigations.

---

## 15.3 Sysmon Monitoring Results

Sysmon successfully generated endpoint telemetry on the Windows machines.

The collected events included:

- Process creation
- Network connections
- File creation
- Registry modifications
- Image loading
- DNS queries

The event data was forwarded through Splunk Universal Forwarder and indexed in Splunk Enterprise.

By searching the `endpoint` index, I could examine the different event categories and identify activity associated with individual Windows machines.

---

## 15.4 Atomic Red Team Results

Atomic Red Team was used to simulate selected adversary techniques.

### T1136.001 — Local Account Creation

The documented Splunk search returned 14 raw events associated with the selected account-creation activity.

### T1059.001 — PowerShell

The project generated additional endpoint telemetry involving PowerShell and registry-related activity.

These results demonstrated that the monitoring setup was collecting useful information during controlled security exercises.

---

## 15.5 Overall Security Monitoring Workflow

The completed environment allowed activity from Windows systems to be centralized and analyzed through Splunk.

```text
                SECURITY TESTING
                       |
                       v
                 WINDOWS ENDPOINT
                       |
                       v
              WINDOWS EVENT LOGGING
                    + SYSMON
                       |
                       v
              SPLUNK UNIVERSAL
                  FORWARDER
                       |
                       v
               SPLUNK ENTERPRISE
                       |
                       v
                 SPL SEARCHES
                       |
                       v
              SECURITY INVESTIGATION
                       |
                       v
               DETECTION ANALYSIS
```

The project demonstrates how endpoint telemetry, authentication data, and controlled attack simulations can be combined to support SOC investigations.

---

# 16. Conclusion

This project provided hands-on experience building and monitoring an Active Directory environment using VMware Workstation Pro, Windows Server 2022, Windows 10, Ubuntu Server, and Kali Linux.

I configured Windows Server 2022 as an Active Directory Domain Controller, created Organizational Units and domain accounts, and successfully joined a Windows 10 workstation to the domain.

I then deployed Splunk Enterprise on Ubuntu Server and installed Splunk Universal Forwarder and Sysmon on both Windows machines to establish centralized security monitoring.

With the logging environment operational, I conducted controlled RDP authentication testing using NetExec and investigated the resulting authentication events using Splunk Search Processing Language.

I also used Atomic Red Team to execute selected MITRE ATT&CK techniques and examine the endpoint activity recorded by Sysmon.

The project demonstrated the importance of centralized event collection, Windows authentication auditing, endpoint visibility, and log correlation in detecting and investigating suspicious activity.

The environment provides a foundation for additional security monitoring, threat-hunting, and detection-engineering exercises.

### Skills Demonstrated

- Active Directory Domain Services
- Windows Server Administration
- VMware Virtualization
- Network Configuration
- Windows Authentication
- Splunk Enterprise
- Splunk Universal Forwarder
- Microsoft Sysmon
- Windows Event Log Analysis
- SPL Query Development
- Threat Detection
- Security Event Investigation
- MITRE ATT&CK
- Controlled Adversary Simulation
- Purple Teaming

---

# 17. References

### Microsoft

- [Windows 10 Installation Media](https://www.microsoft.com/software-download/windows10)
- [Windows Server 2022](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022)
- [Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/)
- [Microsoft Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
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

### Security Testing and Monitoring

- [Kali Linux](https://www.kali.org/)
- [Olaf Hartong Sysmon Modular](https://github.com/olafhartong/sysmon-modular)
- [NetExec](https://www.netexec.wiki/)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [Invoke-AtomicRedTeam](https://github.com/redcanaryco/invoke-atomicredteam)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

<div align="center">

### Active Directory SOC HomeLab

**Virtualization | Active Directory | Endpoint Monitoring | SIEM | Threat Detection**

*All security testing was conducted in an authorized virtual laboratory environment for educational purposes.*

</div>
