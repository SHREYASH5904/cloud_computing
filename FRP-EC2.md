# FRP Server Setup on AWS EC2 (Ubuntu)

This guide walks you through setting up an **FRP (Fast Reverse Proxy)** server on an **AWS EC2 Ubuntu instance**, including firewall setup, binary installation, configuration, and systemd integration.

---

## 📌 Prerequisites

* An **AWS EC2 instance** running **Ubuntu** (referred to as *FRP Instance*)
* **FRP binary** downloaded from [FRP GitHub Releases](https://github.com/fatedier/frp/releases)
* **Open Ports**: 7000 (FRP), 7500 (dashboard), optional HTTP/HTTPS ports (80/443)
* Basic familiarity with **Linux command line**
* **PuTTY** for Windows users (or use SSH directly on Unix-based OS)

---

## 🛠️ Step 1: Launch and Configure EC2 Instance

1. Login to [AWS Console](https://console.aws.amazon.com/).
2. Go to **EC2 → Launch Instance**.
3. Choose **Ubuntu AMI**.
4. Select instance type (e.g., `t2.micro` for free tier).
5. Configure instance and launch with an existing or new key pair.

---

## 🔐 Step 2: Update Security Group Rules

1. Go to **Security Groups** > select your instance's group.
2. Edit **Inbound Rules**:

   * Add TCP Rule: **Port 7000** from **0.0.0.0/0**
   * Add TCP Rule: **Port 7500** (for dashboard)
   * Optionally add **80** and **443** if using HTTP/HTTPS

---

## 🔗 Step 3: Connect to EC2 via SSH

### ✅ For Unix/macOS:

```bash
chmod 400 your-key.pem
ssh -i your-key.pem ubuntu@<your-ec2-public-ip>
```

### ✅ For Windows (PuTTY):

1. Use **PuTTYgen** to convert `.pem` → `.ppk`
2. In **PuTTY**:

   * Set hostname: `<your-ec2-public-ip>`
   * Go to **Connection → SSH → Auth** → load `.ppk`
   * Username: `ubuntu`

---

## 📅 Step 4: Download and Install FRP Server

```bash
sudo apt update
sudo apt install wget curl -y
wget https://github.com/fatedier/frp/releases/download/v0.47.0/frp_0.47.0_linux_amd64.tar.gz
tar -zxvf frp_0.47.0_linux_amd64.tar.gz
cd frp_0.47.0_linux_amd64
```

Move the binary and config:

```bash
sudo mkdir -p /opt/frp
sudo mv frps frps.ini /opt/frp/
sudo chmod +x /opt/frp/frps
```

---

## ⚙️ Step 5: Configure FRP Server

Edit `/opt/frp/frps.ini`:

```ini
[common]
bind_port = 7000
vhost_http_port = 80
vhost_https_port = 443
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
```

---

## 🔄 Step 6: Run FRP Server Manually (Optional)

```bash
cd /opt/frp
sudo ./frps -c ./frps.ini
```

---

## 🧹 Step 7: Create a systemd Service

Create service file:

```bash
sudo nano /etc/systemd/system/frps.service
```

Add contents:

```ini
[Unit]
Description=FRP Server
After=network.target

[Service]
ExecStart=/opt/frp/frps -c /opt/frp/frps.ini
Restart=always
User=nobody
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Reload systemd and start service:

```bash
sudo systemctl daemon-reload
sudo systemctl start frps
sudo systemctl enable frps
```

Check status:

```bash
sudo systemctl status frps
```

---

## 📊 Step 8: Access FRP Dashboard

Visit:

```
http://<your-ec2-public-ip>:7500
```

Login with:

* **Username**: `admin`
* **Password**: `admin`

---

## ✅ Success!

Your **FRP server is now running** on your EC2 instance and accessible via dashboard and port forwarding rules. You can now configure your FRP **clients** to connect to this server.

---

## 🔗 References

* Official Docs: [https://github.com/fatedier/frp](https://github.com/fatedier/frp)
