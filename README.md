# Cloud-VPC-Repository

# Project 1: AWS VPC Network Design (Web–App Architecture)

## Overview
This project demonstrates a basic but production-style AWS VPC network architecture
that separates **public-facing web services** from **private application services**.

The main objective was not only to deploy resources, but to **design, validate,
and troubleshoot network communication paths** inside a VPC, following
real-world cloud engineering practices.

---

## Architecture Summary

- Public Web Server exposed to the internet
- Private Application Server isolated from external access
- Bastion-style access using the Web server
- Outbound-only internet access from Private Subnet via NAT Gateway
- Security Group references for role-based access control

---

## Architecture Diagram

Internet
|
[ Internet Gateway ]
|
[ Public Subnet (10.0.1.0/24) ]
|
[ Web EC2 ]
|
(VPC local routing)
|
[ App EC2 ]
|
[ NAT Gateway + Elastic IP ]
|
Internet

---

## Network Configuration

### VPC
- CIDR: `10.0.0.0/16`
- DNS Resolution: Enabled
- DNS Hostnames: Enabled

### Subnets
| Name | CIDR | Purpose |
|-----|------|--------|
| Public Subnet | 10.0.1.0/24 | Web / Bastion |
| Private Subnet | 10.0.2.0/24 | Application |

---

## Routing Design

### Public Route Table
- `10.0.0.0/16 → local`
- `0.0.0.0/0 → Internet Gateway`

### Private Route Table
- `10.0.0.0/16 → local`
- `0.0.0.0/0 → NAT Gateway`

---

## Security Group Design

### SG-Web
- Inbound
  - HTTP (80) from `0.0.0.0/0`
  - SSH (22) from My IP
- Outbound
  - Allow all

### SG-App
- Inbound
  - HTTP (80) from **SG-Web**
  - SSH (22) from **SG-Web**
- Outbound
  - Allow all

> Security Group references were used instead of CIDR blocks to enforce
role-based access control and improve scalability.

---

## Implementation Steps

### 1. Web Server (Public Subnet)
- EC2 launched with Public IP enabled
- Apache (`httpd`) installed and started
- External access verified via browser and curl

### 2. App Server (Private Subnet)
- EC2 launched without Public IP
- Apache (`httpd`) installed via Bastion access
- Direct external access intentionally blocked

---

## Testing & Validation

| Test Case | Result |
|---------|--------|
| External → Web | ✅ Success |
| Web → App (Private IP) | ✅ Success |
| External → App | ❌ Blocked (Expected) |
| App → Internet (via NAT) | ✅ Success |

### Example Test Commands
```bash
# Web → App communication
curl http://<app-private-ip>

# App → Internet via NAT
curl https://www.google.com

---

##Troubleshooting Experience (Key Learning)
During testing, the App server could not access the internet.
Initial suspicion included Security Groups and DNS configuration.

**Root Cause**
-NAT Gateway was not created.
-Private Route Table had no outbound route

**Resolution**
-Created NAT Gateway in Public Subnet
-Attached Elastic IP
-Updated Private Route Table to route 0.0.0.0/0 via NAT
-This reinforced the importance of route tables over intuition when diagnosing cloud networking issues.

**Key Takeaways**
-Public Subnet does not automatically mean public access
-Private Subnet instances require NAT for outbound internet connectivity
-Security Group references are superior to CIDR-based rules for internal services
-SSH is for management; service-level testing should use application protocols
-Most cloud networking issues are caused by routing, not instances

**Technologies Used**
AWS EC2
AWS VPC
Internet Gateway
NAT Gateway
Elastic IP
Security Groups
Apache HTTP Server (httpd)
Amazon Linux
Next Improvements
Add Application Load Balancer in front of Web tier
Replace Bastion SSH with AWS Systems Manager (SSM)
Convert architecture into Terraform (IaC)
One-Line Summary (Interview Ready)

Designed and troubleshot a secure AWS VPC architecture with public web access,
private application isolation, and NAT-based outbound connectivity.
