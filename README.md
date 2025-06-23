# 🛠️ Step-by-Step Manual: NGINX Plus Installation with Dashboard in Failover Mode (Red Hat 9)

This guide covers the installation, configuration, and validation of the NGINX Plus dashboard in active/passive failover mode using Ansible across two Red Hat 9 servers.

---

## 1. Prerequisites

Two Red Hat 9 servers:
- `nginx-primary.ghc.local` (IP: x.x.x.x)
- `nginx-secondary.ghc.local` (IP: x.x.x.x)

Requirements:
- SSH access using username and password
- NGINX Plus repository and license files (`nginx-repo.key`, `nginx-repo.crt`, optional `license.jwt`)
- Valid SSL certificate files:
  - `/etc/nginx/ssl/STAR.GHC.COM.crt`
  - `/etc/nginx/ssl/ghc.com.key`

---

## 2. Ansible Structure

Directory: `/ghc/ansible/`

```
├── inventory.ini
├── nginx-install.yml
├── nginx-uninstall.yml
├── nginx-conf.yml
├── fix-dashboard-api.yml
├── nginx-dashboard-conf-open.yml
└── files/
    ├── nginx-repo.key
    ├── nginx-repo.crt
    ├── ghc.com.key
    └── STAR.GHC.COM.crt
```

---

## 3. Uninstall Default NGINX Webserver

```bash
ansible-playbook -i inventory.ini nginx-uninstall.yml
```

---

## 4. Install NGINX Plus

```bash
ansible-playbook -i inventory.ini nginx-install.yml
```

---

## 5. NGINX Main Configuration

```bash
ansible-playbook -i inventory.ini nginx-conf.yml
```

---

## 6. Dashboard Configuration (API + HTML)

### 6.1 Open Dashboard Version for Validation

```bash
ansible-playbook -i inventory.ini nginx-dashboard-conf-open.yml
```

Validate:
```bash
curl -vk https://nginx-primary.ghc.local/dashboard.html
```

### 6.2 Fix Dashboard API Block (if needed)

```bash
ansible-playbook -i inventory.ini fix-dashboard-api.yml
```

---

## 7. Final Validation

Access from browser:
- `https://nginx-primary.ghc.local/dashboard.html`
- `https://nginx-secondary.ghc.local/dashboard.html`

Dashboard should show:
- Connections
- Workers
- Traffic
- PID state and uptime

---

## 8. (Optional) Harden Dashboard Access

Replace `allow all;` with trusted IPs only:
```nginx
allow 127.0.0.1;
allow x.x.x.x;
allow x.x.x.x;
deny all;
```
Reload NGINX:
```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

## 9. Configure Active/Passive Failover with Keepalived

### Objective

Configure real active/passive failover between `nginx-primary.ghc.com` and `nginx-secondary.ghc.com` using floating IP `x.x.x.x`.

### Step-by-Step

#### 9.1 Install Keepalived
```bash
sudo dnf install -y keepalived
```

#### 9.2 Create `/etc/keepalived/keepalived.conf`

**Primary Node:**
```bash
state MASTER
interface eth1
virtual_router_id 51
priority 150
...
virtual_ipaddress {
    x.x.x.x
}
```

**Secondary Node:**
```bash
state BACKUP
priority 100
...
virtual_ipaddress {
    x.x.x.x
}
```

Include:
```bash
garp_master_delay 1
garp_master_repeat 3
```

#### 9.3 Enable & Start Keepalived
```bash
sudo systemctl enable --now keepalived
sudo systemctl status keepalived
```

#### 9.4 Validate Floating IP Assignment
```bash
ip a show eth1
ip addr | grep x.x.x.x
```

#### 9.5 Simulate Failover
```bash
sudo systemctl stop keepalived
```
Validate failover to secondary node.

---

## 🧪 External Validation

```bash
curl -vk https://x.x.x.x/dashboard.html
```

---

## ✅ Summary of Completed Tasks

- Full NGINX Plus installation and dashboard setup over HTTPS
- Dashboard API exposed and functional
- Configuration fully automated with Ansible
- Active/passive HA using Keepalived and floating IP
- TLS enabled with wildcard certificate (*.ghc.com)
- Failover validated by stopping primary node and observing IP transfer

---

## 📬 Contact

For support or questions:  
📧 **jmartinez@arkhadia.net**  
📱 **+507 6363-6738**  
🌐 **@genialcorpholding**
