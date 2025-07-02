# 📘 Amazon VPC (Virtual Private Cloud)

Amazon VPC enables you to launch AWS resources in a logically isolated virtual network that you define. You have full control over your virtual networking environment, including IP ranges, subnets, route tables, gateways, and security.

---

## 🔹 Key Components of VPC

| Component        | Description |
|------------------|-------------|
| **VPC**          | A virtual network dedicated to your AWS account. |
| **Subnet**       | A range of IP addresses in your VPC. Can be public or private. |
| **Route Table**  | Determines how traffic is directed within the VPC. |
| **Internet Gateway (IGW)** | Connects your VPC to the internet. Required for public subnets. |
| **NAT Gateway/Instance** | Allows private subnets to access the internet without being exposed. |
| **Security Groups** | Acts as a virtual firewall for EC2 instances to control traffic. |
| **Network ACLs** | An optional stateless firewall at the subnet level. |
| **Elastic IP**   | A static public IPv4 address you can associate with instances. |
| **VPC Peering**  | Allows private network connectivity between two VPCs. |
| **VPC Endpoints**| Enables private connection to AWS services without internet access. |

---

## 🧱 VPC Structure

- One VPC can have multiple subnets (usually across different availability zones).
- Subnets can be:
  - **Public** (access to the internet via IGW)
  - **Private** (no direct internet access)

---

## 🔑 VPC Use Cases

- Hosting a secure web application
- Isolating environments (dev, test, prod)
- Connecting on-premises networks with AWS using VPN or Direct Connect
- Enabling hybrid cloud architectures

---

## 🌐 Types of Subnets

| Type     | Description |
|----------|-------------|
| **Public Subnet** | Routes traffic through Internet Gateway |
| **Private Subnet**| No direct route to the Internet Gateway |
| **VPN-only Subnet** | Used for resources that communicate only through a VPN |

---

## 🔒 Security in VPC

- **Security Groups (SG)**: Stateful, instance-level firewalls. Only allow specified traffic.
- **Network ACLs (NACL)**: Stateless, subnet-level firewalls. Allow/deny rules for IP ranges.

---

## 🔁 VPC Peering vs Transit Gateway

| Feature         | VPC Peering           | Transit Gateway         |
|----------------|------------------------|--------------------------|
| Connectivity   | Point-to-point          | Hub-and-spoke            |
| Scalability    | Limited                 | Highly scalable          |
| Management     | Complex for many VPCs   | Centralized management   |

---

## 🔄 Routing in VPC

Routing is controlled by **Route Tables** which define how traffic is directed:

```text
Destination        Target
0.0.0.0/0          igw-xxxx (for public subnet)
10.0.0.0/16        local (default internal routing)


## image.png