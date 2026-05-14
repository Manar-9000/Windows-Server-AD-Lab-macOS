# Windows-Server-AD-Lab-macOS
A complete virtualization of a Windows Server 2022 environment and Windows 11 client, demonstrating Active Directory configuration, DNS management, and virtual networking troubleshooting on Intel-based hardware.

# Project Overview
This project documents the deployment of a virtualized Windows Server environment on macOS using VMware Fusion. The goal is to build a functional Active Directory (AD) environment to practice systems administration, identity management, and networking.

## Phase 1: Installation & Initial Configuration
### Hypervisor Setup
Resource Allocation: Each VM is allocated 2 CPU cores and 4GB of RAM to ensure smooth performance on a 16GB host.
Installation Method: Used a manual installation approach rather than "Easy Install" to prevent licensing and configuration errors during the Windows setup wizard.
VMware Tools: Installed VMware Tools on both guest operating systems to enable clipboard sharing, smooth cursor movement, and optimized display drivers.

![Prerequisite Check](Images/Prerequisites.png)

### Networking & Connectivity
Network Type: Configured both VMs to use NAT (Share with my Mac) to provide internet access while maintaining an isolated virtual network.

### Troubleshooting Connectivity:
Issue: Initial ping requests from the Windows 11 client to the Server timed out.
Root Cause: Windows Defender Firewall was blocking ICMP traffic by default.
Resolution: Temporarily disabled the firewall on the Server to verify Layer 3 connectivity and ensure seamless domain joining in later steps.

### Static IP & DNS Configuration
To ensure the stability of the Domain Controller, the Server's IP address was "frozen" with a static configuration.

### Network Configuration
| Component | IP Address | Purpose |
| :--- | :--- | :--- |
| **Domain Controller** | `192.168.211.10` | Primary AD DS / DNS Server |
| **Default Gateway** | `192.168.211.2` | Virtual Router (VMware NAT) |
| **DNS (Server)** | `127.0.0.1` | Local Loopback for AD services |
| **DNS (Client)** | `192.168.211.10` | Points Client to DC for name resolution |

---

## Phase 2: Role Installation & Conflict Resolution
### Role Selection
The goal was to install the core "brains" of the network. I selected:
1. **AD DS (Active Directory Domain Services):** The central identity database.
2. **DNS Server:** Essential for resolving domain names to IP addresses.
3. **AD CS (Certificate Services):** Initially installed for digital identity management.

### Troubleshooting the "Order of Operations"
During the promotion process, a prerequisite check failed because **AD CS** was installed as a "Standalone" authority before the domain existed.
* **Issue:** Conflict between Standalone AD CS and Root Domain Controller promotion.
* **Resolution:** 1. Utilized the "Remove Roles and Features" wizard to uninstall AD CS.
    2. Successfully passed the AD DS prerequisite check.
    3. Re-installation of AD CS is deferred until after the domain is established (as an Enterprise CA).

  ### The Problem
The prerequisite check failed because a "Certificate Server" was already installed on the local machine.
![ADCS Conflict Error](Images/error-adcs.png)

### The Resolution
I utilized the "Remove Roles and Features Wizard" to uninstall the AD CS role. 
![Removing ADCS](Images/removing-adcs.png)

Once removed, the prerequisites check passed successfully, allowing the installation to proceed.
![Success Check](Images/success-check.png)

---

## Phase 3: Domain Controller Promotion
### Initializing the Forest
The server was promoted from a standalone workgroup machine to a **Domain Controller**.
* **Deployment:** Created a new Forest named `manar.local`.
* **Functional Level:** Set to **Windows Server 2016** to ensure a balance between modern features and legacy compatibility.
* **DSRM Security:** Configured a secure Directory Services Restore Mode password for emergency database maintenance.

![Domain Configuration](Images/domain-config.png)
![DSRM Password Setup](Images/dsrm-password.png)

### Results
Upon successful promotion and reboot, the server now authenticates users against the `MANAR` domain. 

**Verification:** The login screen now displays `MANAR\Administrator`, confirming that the Active Directory database is active and managing local security.

![Final Login Verification](Images/success-login.png)

---

## Technical Concept: The "Hierarchy"
* **The Domain Controller (This VM):** The central management office holding the keys and employee ledger.
* **Member Servers (Future VMs):** Specialized worker servers (File, Web, SQL) that join the domain to follow the DC's rules.
* **Users & Groups:** Logical objects created within the DC to manage human access to network resources.
