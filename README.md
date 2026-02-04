# NENO Active Directory Lab Documentation

## 1.Project Overview

This project is a hands-on **Active Directory (AD) lab** built entirely from scratch, designed to simulate a real-world enterprise environment and evolve over time.

The lab starts with a fully **on-premises Active Directory deployment**, including multiple virtual machines that represent users from different departments. These machines will be joined to the domain and managed using standard AD practices such as group management, organizational units (OUs), and Group Policy Objects (GPOs).

In addition to the domain controllers, the lab will include:

- A **File Server** hosted on its own virtual machine
- A **Print Server** hosted on a separate virtual machine
- Multiple **client PCs** to simulate end users across departments

As the on-premises environment becomes well-structured and stable, the lab will transition into a **hybrid architecture**:

- Active Directory will be **replicated to Azure**, resulting in two synchronized Domain Controllers
- A **site-to-site VPN** will connect the on-premises network to Azure
- The File Server and Print Server will be migrated to **Azure Virtual Machines**

Once the hybrid AD setup is in place, the lab will expand further to include cloud identity and productivity services:

- **Azure Entra ID (Azure AD)**
- **Microsoft 365 / Office 365**
- Hands-on practice with **Exchange**, **Intune**, and other Microsoft cloud services

### Goals of the Lab

- Simulate a realistic enterprise IT environment
- Practice Active Directory design, deployment, and management
- Gain experience with hybrid identity and cloud migration
- Explore modern Microsoft tools and technologies in a safe lab setting
- Learn, experiment, break things, fix them, and most importantly — **have fun doing it**

![alt text](<Screenshot 2026-02-04 203145.png>)

## 2.Active Directory Installation and Initial Configuration

This section documents the deployment of **Active Directory Domain Services (AD DS)** and the initial directory structure implemented in the lab.

### Server Deployment

A **Windows Server 2022** virtual machine was created using a Windows Server 2022 ISO image and deployed on **VirtualBox**.

After completing the operating system installation and base configuration, the following roles and features were installed:

- Active Directory Domain Services (AD DS)
- DNS Server

The server was then **promoted to a Domain Controller**, creating a new forest and domain.

### Domain Configuration

- **Domain Name:** `ad.neno.info`
- **DNS:** Installed and integrated with Active Directory

This server currently serves as:

- Primary Domain Controller
- DNS Server for the domain

### Organizational Unit (OU) Structure

To maintain a scalable, secure, and enterprise-style directory design, a custom OU hierarchy was created under a top-level organizational unit.

#### Top-Level OU

- **NENO**

All company-related objects are placed under this OU to simplify administration and Group Policy management.

### Detailed OU Layout

![alt text](<Screenshot 2026-02-02 171008.png>)

### OU Design Rationale

#### Admins

- Contains privileged administrative accounts
- **IT Admins** OU is used to separate elevated accounts from standard user accounts
- Enables tighter security controls and targeted Group Policies

#### Users

- Organized by department:
  - Finance
  - HR
- Simplifies user management and department-based policy application

#### Computers

- Separated by role:
  - **Workstations:** End-user client machines
  - **Servers:** Infrastructure and application servers

#### Groups

- Split by group type:
  - **Security Groups:** Used for permissions and access control
  - **Distribution Groups:** Used for communication and email distribution

#### Service Accounts

- Dedicated OU for service and application accounts
- Allows stricter security policies and easier auditing

### Users, Computers, and Asset Management

For the initial phase of the lab:

- Approximately **5 user accounts** are created
- Includes **1 IT administrator**

Client machines:

- Around **5 workstations**
- Named using a consistent convention (e.g. `PC-01`, `PC-02`, etc.)

To improve traceability and asset control:

- Each computer object includes a description referencing the assigned user and department
- Each user account includes a description referencing their assigned workstation

This approach reflects real-world enterprise best practices for clarity, accountability, and long-term manageability.

## Domain-Joined Workstations Deployment

This section documents the deployment and configuration of the client machines used to simulate end users across different departments.

### Workstation Creation

A total of **5 client PCs** were created as virtual machines.  
Each workstation was deployed using the same base operating system image to ensure consistency across the environment.

The workstations were configured as **standard user PCs**, representing endpoints used by employees in different departments.

### Naming Convention

Each workstation follows a standardized naming convention to support clarity, asset tracking, and best practices:

- Format: `PC-XX`
- Examples:
  - `PC-01`
  - `PC-02`
  - `PC-03`

This naming scheme simplifies administration, troubleshooting, and reporting within Active Directory.

### Network Configuration

Before joining the domain, each workstation was configured with the following network settings:

- Network connectivity verified
- **Primary DNS server set to the Domain Controller’s IP address**
- Internet access tested (where applicable)

Pointing client machines directly to the Domain Controller for DNS ensures proper domain name resolution and successful domain join operations.

### Domain Join Process

Once networking was correctly configured, each workstation was:

1. Joined to the `ad.neno.info` domain
2. Restarted to apply domain membership
3. Verified in Active Directory

After joining the domain:

- Workstations were moved into the appropriate OU:
  - `NENO > Computers > Workstations`
- Computer objects were reviewed to confirm correct placement and naming

### User and Workstation Association

To improve visibility and asset management:

- Each workstation is associated with a specific user
- Each user is assigned a primary workstation

This association is documented using the **Description** field in Active Directory:

- Computer objects reference the assigned user and department
- User objects reference the assigned workstation

This mirrors real-world enterprise practices and makes it easier to manage users, devices, and permissions as the environment grows.

![alt text](<Screenshot 2026-02-04 203632.png>)

## 3. File Server Deployment and Configuration

In this step, a dedicated **File Server** was deployed to provide centralized storage for users, departments, and public resources, as well as support for **user home folders**. This setup simulates a typical enterprise file server environment.

### Server Deployment

A **Windows Server 2022** virtual machine was created and configured with:

- Network connectivity verified within the `ad.neno.info` domain  
- **Primary DNS** set to the Domain Controller  
- Server joined to the Active Directory domain as a **member server**  

The server object was placed in the following OU:

- `NENO > Computers > Servers`

### Roles and Features

The following roles were installed on the server:

- **File Server**  
- **File Server Resource Manager (optional)** for quota management and reporting  

### Disk Partitioning and Folder Structure

A dedicated disk partition was created for shared resources:

- **Partition:** `N:` (labeled `Shares`)  
- **Folder hierarchy**:

N:
├── Users
├── Departments
│ ├── Finance
│ └── HR
└── Public


This structure separates **user-specific data**, **departmental data**, and **public shared resources**.

### Share and NTFS Permissions

Shared folders were configured as follows:

- **Users folder:**  
  - Share Permissions: `Authenticated Users – Full Control`  
  - NTFS Permissions: Individual users have **Full Control** over their own folder  

- **Departments folders (Finance, HR):**  
  - Share Permissions: Department security groups have **Change** access  
  - NTFS Permissions: Restricted to corresponding department users  

- **Public folder:**  
  - Share Permissions: `Authenticated Users – Read/Write`  
  - NTFS Permissions: All domain users can read and write  

This configuration ensures proper access control using **NTFS permissions** while allowing network access through share permissions.

### Group Policy for Drive Mapping

GPOs were created on **Domain Controller 1** to automatically map network drives:

- **Departmental drives:**  
  - Finance users → `N:\Departments\Finance`  
  - HR users → `N:\Departments\HR`  

- **User home folders:**  
  - Configured in Active Directory user properties  
  - Automatically mapped as `N:\Users\<username>`  

- **Public share:**  
  - Mapped for all users to `N:\Public`  

This allows users to access authorized folders automatically on login.

### Verification

After configuration:

- Users successfully logged in and accessed mapped drives  
- Permissions were verified to ensure proper access control  
- Drives were mapped according to department and user-specific policies  

This completes the deployment and configuration of the **on-premises File Server**.

![alt text](<Screenshot 2026-02-04 205234.png>)

## 4.
