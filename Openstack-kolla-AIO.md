# Deploying OpenStack with Kolla-Ansible on an 8GB RAM Laptop (AIO Mode)

Running OpenStack on a modest laptop using Kolla-Ansible in **All-in-One (AIO)** mode is possible, though performance will be constrained. This guide walks you through the setup on **Ubuntu 22.04 LTS**.

---

## System Requirements & Overview

* **Tool:** Kolla-Ansible (runs OpenStack services in Docker containers)
* **Host OS:** Ubuntu 22.04 LTS
* **Memory:** 8 GB RAM (minimum)
* **Disk Space:** 200 GB recommended
* **Deployment Mode:** All-in-One (Single Node)
* **OpenStack Version:** 2023.1 (or latest supported)

---

## Installation Steps

### 🔹 Step 1: Install Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-dev libffi-dev gcc libssl-dev python3-pip git
sudo apt install -y docker.io
sudo systemctl enable docker && sudo systemctl start docker
sudo usermod -aG docker $USER
newgrp docker  # Refresh group permissions
```

---

### 🔹 Step 2: Install Ansible & Kolla-Ansible

```bash
pip3 install -U pip
pip3 install 'ansible>=6,<8'
pip3 install kolla-ansible
```

---

### 🔹 Step 3: Setup Kolla Configuration

```bash
sudo mkdir -p /etc/kolla
sudo chown $USER:$USER /etc/kolla
cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
cp /usr/local/share/kolla-ansible/ansible/inventory/all-in-one .
```

---

### 🔹 Step 4: Generate Passwords

```bash
kolla-genpwd
```

Edit `/etc/kolla/passwords.yml` to customize `keystone_admin_password` if needed.

---

### 🔹 Step 5: Configure Global Settings

```bash
nano /etc/kolla/globals.yml
```

Recommended configuration:

```yaml
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_release: "2023.1"
kolla_internal_vip_address: "127.0.0.1"
network_interface: "eth0"  # Confirm via `ip a`
neutron_plugin_agent: "openvswitch"
enable_horizon: "yes"
enable_cinder: "no"
enable_heat: "no"
enable_sahara: "no"
```

💡 **Tip:** Disable unused services to reduce RAM usage.

---

### 🔹 Step 6: Bootstrap the Node

```bash
kolla-ansible -i all-in-one bootstrap-servers
```

---

### 🔹 Step 7: Run Prechecks & Pull Docker Images

```bash
kolla-ansible -i all-in-one prechecks
kolla-ansible -i all-in-one pull
```

This downloads \~5–10 GB of container images.

---

### 🔹 Step 8: Deploy OpenStack

```bash
kolla-ansible -i all-in-one deploy
```

Success is indicated by: `TASK [Deployment completed]`

---

### 🔹 Step 9: Setup CLI Access

```bash
source /etc/kolla/admin-openrc.sh
openstack service list
```

---

### 🔹 Step 10: Access Horizon Dashboard

```bash
echo "http://$(hostname -I | awk '{print $1}')/"
```

* **Username:** `admin`
* **Password:** Check `keystone_admin_password` in `/etc/kolla/passwords.yml`

---

## Optional Cleanup

To remove the deployment:

```bash
kolla-ansible -i all-in-one destroy --yes-i-really-really-mean-it
```

---

## Tips to Save Memory

* Disable unneeded services in `globals.yml`
* Focus on core components: Keystone, Glance, Nova, Neutron, Horizon
* Run image pull/deploy steps overnight if needed

---

With careful resource management, you can explore OpenStack on a limited laptop. Perfect for learning, prototyping, and experimentation!
