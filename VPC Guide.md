
---

# AWS VPC Setup with Public and Private Subnets

This README provides step-by-step guidance to create an AWS VPC setup with the following architecture:

* A single VPC.
* Two Availability Zones (AZs): **Zone A** and **Zone B**.
* Each AZ contains:

  * **Public Subnet** (accessible from the internet).
  * **Private Subnet** (isolated from the internet).
* Public subnets connect to the internet using an **Internet Gateway (IGW)**.
* Private subnets communicate through a **Virtual Private Gateway (VPG)**.

---

## Definitions

### Virtual Private Cloud (VPC)

A logically isolated network in AWS where you can launch resources. You define the IP address range, subnets, route tables, and gateways.

### Subnets

Divisions of a VPC. Public subnets can access the internet, while private subnets cannot unless routed through NAT or VPN.

### Internet Gateway (IGW)

Enables internet access for public subnets by routing traffic destined for the internet.

### Virtual Private Gateway (VPG)

A VPN concentrator on the AWS side that allows secure communication between private subnets and external networks.

### Route Table

Defines where network traffic is directed based on destination IP addresses.

### Availability Zone (AZ)

A physically isolated zone within an AWS region, offering high availability and fault tolerance.

---

## Architecture Overview

### VPC CIDR

* `10.0.0.0/24`

### Subnets

* **Zone A**:

  * Public Subnet 1: `10.0.0.0/26`
  * Private Subnet 1: `10.0.0.128/26`
* **Zone B**:

  * Public Subnet 2: `10.0.0.64/26`
  * Private Subnet 2: `10.0.0.192/26`

### Gateways

* **Internet Gateway (IGW)**: Enables internet access for Public Subnet 1 and 2.
* **Virtual Private Gateway (VPG)**: Enables secure communication between Private Subnet 1 and 2 or external VPNs.

---

## Step-by-Step Instructions

### Step 1: Create a VPC

1. Open the AWS VPC Dashboard.
2. Click **Create VPC**.
3. Set:

   * Name: `My-VPC`
   * CIDR block: `10.0.0.0/24`
4. Create the VPC.

---

### Step 2: Create Subnets

#### Zone A:

* Public Subnet 1:

  * AZ: e.g., `us-east-1a`
  * CIDR: `10.0.0.0/26`

* Private Subnet 1:

  * AZ: `us-east-1a`
  * CIDR: `10.0.0.128/26`

#### Zone B:

* Public Subnet 2:

  * AZ: e.g., `us-east-1b`
  * CIDR: `10.0.0.64/26`

* Private Subnet 2:

  * AZ: `us-east-1b`
  * CIDR: `10.0.0.192/26`

---

### Step 3: Create and Attach Internet Gateway (IGW)

1. Navigate to **Internet Gateways**.
2. Click **Create Internet Gateway**, name it `My-IGW`.
3. Create and attach it to `My-VPC`.

---

### Step 4: Create and Attach Virtual Private Gateway (VPG)

1. Navigate to **Virtual Private Gateways**.
2. Click **Create Virtual Private Gateway**, name it `My-VPG`.
3. Use default ASN, then create and attach it to `My-VPC`.

---

### Step 5: Configure Route Tables

#### Public Route Table:

1. Create a new route table named `Public-Route-Table` for `My-VPC`.
2. Add route:

   * Destination: `0.0.0.0/0`
   * Target: `My-IGW`
3. Associate `Public-Subnet-1` and `Public-Subnet-2` to this route table.

#### Private Route Table:

1. Create a new route table named `Private-Route-Table` for `My-VPC`.
2. Enable route propagation:

   * Choose `My-VPG` under route propagation settings.
3. Associate `Private-Subnet-1` and `Private-Subnet-2` to this route table.

---

### Step 6: Launch EC2 Instances

#### Public Subnets

1. Launch EC2 in `Public-Subnet-1` and `Public-Subnet-2`:

   * AMI: Amazon Linux 2
   * Auto-assign Public IP: Enabled
   * Security Group: Allow SSH (e.g., port 22) from trusted IP

#### Private Subnets

1. Launch EC2 in `Private-Subnet-1` and `Private-Subnet-2`:

   * AMI: Amazon Linux 2 or Ubuntu
   * Auto-assign Public IP: Disabled
   * Security Group: Allow internal traffic between private subnets
   * Key Pair: Create or use existing for SSH (if VPN access exists)

---

## Testing and Verification

* Public instances should have internet access.
* Private instances should **not** have direct internet access but should communicate with each other.
* Use ping or SSH (if VPC peering or VPN exists) to test connectivity between private subnets.

---

## Useful Links

* [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/)
* [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/latest/userguide/)
* [AWS Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
* [AWS Virtual Private Gateway](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)

---
