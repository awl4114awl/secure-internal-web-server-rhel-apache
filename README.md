# Secure Internal Web Server Deployment – Apache on RHEL (Azure)

## ⓘ Overview

This lab was carried out in The Cyber Range, an Azure-hosted enterprise environment where I replicate real-world infrastructure deployment, secure configuration, and controlled service exposure workflows.

In this scenario, I provisioned a Red Hat Enterprise Linux 9.4 virtual machine in Microsoft Azure, installed and configured Apache (httpd), deployed a static website, and securely accessed the service using SSH local port forwarding instead of exposing port 80 publicly. This approach mirrors enterprise best practices for minimizing attack surface and enforcing controlled administrative access.

---

## Environment

| Component       | Details                                     |
| --------------- | ------------------------------------------- |
| VM Name         | jc-redhat                                   |
| OS Image        | Red Hat Enterprise Linux 9.4 (Plow)         |
| Region          | East US                                     |
| VM Size         | Standard DS1 v2 (1 vCPU, 3.5 GiB RAM)       |
| Security Type   | Trusted Launch (Secure Boot + vTPM Enabled) |
| Network         | Cyber-Range-VNet / Cyber-Range-Subnet       |
| Public IP       | 20.110.38.39                                |
| Private IP      | 10.0.0.x                                    |
| Disk Encryption | Disabled                                    |
| Auto-Shutdown   | Not Enabled                                 |
| Extensions      | None                                        |

---

## Objective

* Deploy an internal Apache web server on RHEL
* Create and host a static website
* Validate service functionality locally
* Securely access the web service without exposing port 80 publicly
* Apply infrastructure security principles in a cloud environment

---

## Deployment Process

### 1. Established Secure SSH Access

After I deployed the vm, I connected to it using SSH from Windows PowerShell and validated the operating system version.

<p align="left">
  <img src="assets/Screenshot 2026-02-13 220214.png" width="600">
</p>

<p align="left">
  <img src="assets/Screenshot 2026-02-13 220234.png" width="600">
</p>

---

### 2. Installed Apache (httpd)

Using the RHEL package manager, I installed the Apache HTTP Server:

```bash
sudo dnf install httpd -y
```

<p align="left">
  <img src="assets/Screenshot 2026-02-13 223910.png" width="600">
</p>

---

### 3. Enabled and Started the Service

I enabled Apache to start at boot and verified its active state using systemctl.

```bash
sudo systemctl enable httpd
sudo systemctl start httpd
sudo systemctl status httpd
```
<p align="left">
  <img src="assets/Screenshot 2026-02-13 230816.png" width="600">
</p>

Apache confirmed it was actively listening on port 80.

---

### 4. Deployed Website Content

I created a custom `index.html` file in `/var/www/html/`, which is the default web root directory on RHEL.

<p align="left">
  <img src="assets/Screenshot 2026-02-13 231023.png" width="600">
</p>

---

### 6. Validated Locally on the Server

Before attempting remote access, I validated functionality directly from the VM using curl.

```bash
curl http://localhost
```
<p align="left">
  <img src="assets/Screenshot 2026-02-13 231114.png" width="600">
</p>

This confirmed:

* Apache was running
* Port 80 was responding
* File permissions were correctly configured
* The web content was accessible internally

---

### 7. Implemented Secure Access via SSH Tunnel

Rather than opening port 80 in Azure Network Security Groups, I established a local SSH tunnel from my workstation:

```powershell
ssh -N -L 8080:localhost:80 awl4114awl@20.110.38.39
```

This securely forwarded traffic from my local machine (port 8080) to port 80 on the remote RHEL VM.

<p align="left">
  <img src="assets/Screenshot 2026-02-13 231301.png" width="600">
</p>

I then accessed the site using:

```
http://localhost:8080
```

<p align="left">
  <img src="assets/Screenshot 2026-02-13 231421.png" width="600">
</p>

This allowed encrypted access to the internal web service without expanding the public attack surface.

---

## Security Design Considerations

* Port 80 was not exposed publicly in Azure NSG.
* Access was restricted to encrypted SSH sessions.
* Service lifecycle was managed via systemd.
* The deployment followed the principle of least exposure.

This reflects common enterprise practices for staging environments, administrative panels, and internal service testing.

---

## Fin

This project demonstrates that I can deploy and manage Linux-based web services in a cloud environment while intentionally applying security controls to limit exposure. Rather than defaulting to public access for convenience, I implemented controlled tunneling to simulate real-world enterprise administration workflows.
