
---

# **OpenStack Deployment on EC2**

## **Prerequisites**

* **Operating System (AMI)**: Ubuntu 22.04 LTS
* **Instance Type**: `t2.large` (2 vCPUs, 8 GB RAM)
* **Storage**: Minimum 30 GB
* **Security Group**:

  * SSH (Port 22): Allow from your IP
  * HTTP/HTTPS (Ports 80, 443): Allow from anywhere for dashboard access
* **Key Pair**: Create or use an existing SSH key

---

## **1. Connect to EC2 Using PuTTY (Windows)**

### a. Download and Install PuTTY

Download from: [https://www.putty.org/](https://www.putty.org/)

### b. Convert PEM to PPK

1. Open **PuTTYgen**
2. Click **Load**, select the `.pem` file
3. Click **Save private key** to generate a `.ppk` file

### c. Connect via PuTTY

1. Open **PuTTY**
2. Enter the **public IP address** of your EC2 instance
3. Under **Connection > SSH > Auth**, select the `.ppk` file
4. Click **Open** and login with username: `ubuntu`

---

## **2. System Update and Dependency Installation**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git python3-dev python3-pip net-tools
```

---

## **3. Clone DevStack**

```bash
git clone https://opendev.org/openstack/devstack.git
cd devstack
```

---

## **4. Configure DevStack**

Create the `local.conf` configuration file:

```bash
nano local.conf
```

Paste the following content:

```ini
[[local|localrc]]
ADMIN_PASSWORD=SuperSecret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

HOST_IP=$(hostname -I | awk '{print $1}')

disable_service tempest
disable_service cinder
disable_service swift
disable_service heat

USE_PYTHON3=True
GIT_BASE=https://opendev.org
```

---

## **5. Install OpenStack**

Run the DevStack installer (takes \~30-45 mins):

```bash
./stack.sh
```

If successful, you’ll see a message indicating completion.

---

## **6. Access the OpenStack Dashboard**

Open your browser and navigate to:

```
http://<your-ec2-public-ip>/dashboard
```

Login credentials:

* **Username**: `admin`
* **Password**: `SuperSecret`

---

## **7. Verify OpenStack Services**

Source the environment:

```bash
source openrc admin admin
```

Check services:

```bash
openstack service list
```

---

## **8. Verify Each Service**

### Keystone

```bash
openstack token issue
```

### Horizon

```bash
sudo systemctl status apache2
```

If inactive:

```bash
sudo systemctl restart apache2
```

### Neutron

```bash
openstack network agent list
```

### Nova

```bash
openstack compute service list
```

### Glance

```bash
openstack image list
```

### Cinder (if enabled)

```bash
openstack volume service list
```

---

## **9. Restart a Specific Service (if needed)**

```bash
sudo systemctl restart devstack@<service-name>
```

Replace `<service-name>` with one of: `keystone`, `nova`, `neutron`, `glance`, etc.

---

## **10. Launch a Virtual Machine via Horizon**

### a. Login to Horizon

Navigate to: `http://<your-ec2-public-ip>/dashboard`
Use credentials:

* Username: `admin`
* Password: `SuperSecret`

### b. Create Key Pair

* Go to: Project → Compute → Key Pairs
* Create new key pair
* Download `.pem` file

### c. Upload OS Image

* Go to: Project → Compute → Images
* Create Image
* Use name like `Ubuntu-22.04`
* Format: `QCOW2`
* Source URL:

  ```
  https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
  ```

### d. Create a Flavor

* Go to: Admin → Compute → Flavors
* Create new flavor with:

  * vCPUs: 1
  * RAM: 2048 MB
  * Disk: 10 GB

### e. Set Up Networking

* Go to: Project → Network → Networks
* Create Network:

  * Name: `private-net`
  * Subnet: `private-subnet`
  * CIDR: `192.168.X.0/24`

---
