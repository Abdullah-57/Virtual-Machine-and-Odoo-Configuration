# Odoo 18 Installation and Configuration

This repository provides a comprehensive guide for installing and configuring Odoo 18 on an Ubuntu Server 24.04.2 LTS virtual machine using VirtualBox. It includes step-by-step instructions for setting up the server, installing Odoo, configuring advanced features, and troubleshooting common issues.

---

## 🌐 Project Overview

**Objective**: Set up Odoo 18, a business management suite, with a focus on installation, configuration, and customization for small to enterprise-scale deployments. The project covers hardware and software requirements, Ubuntu Server setup, Odoo installation, advanced configurations like custom discounts and reports, and testing of sales modules.

---

## 🛠️ Hardware and Software Requirements

### Hardware Requirements

- **Small Installations (Up to 5 Users)**:
  - CPU: Dual-core processor
  - RAM: 2 GB
  - Storage: 10 GB
- **Medium Installations (Up to 20 Users)**:
  - CPU: Quad-core processor
  - RAM: 8 GB
  - Storage: 20 GB
- **Large Installations (Up to 100 Users)**:
  - CPU: 8-core processor
  - RAM: 16 GB
  - Storage: 50 GB
- **Enterprise Installations (Over 100 Users)**:
  - Application Server: 8-core CPU, 32 GB RAM
  - Database Server: 8-core CPU, 32 GB RAM, 100 GB+ storage
  - Recommendation: Use load balancing and high-availability configurations.

### Software Requirements

🔹 **Operating System**: Ubuntu 14.04 LTS or higher (24.04.2 LTS recommended)\
🔹 **Database**: PostgreSQL 9.6 or higher\
🔹 **Python**: Python 3.6 or higher\
🔹 **Additional Packages**: Python dependencies, system libraries (e.g., `libxml2-dev`, `libpq-dev`)

---

## 📦 Installation Steps

### 1. Ubuntu Server Setup

- **Download and Install Ubuntu Server**:
  - Use Ubuntu Server 24.04.2 LTS ISO.
  - Configure VirtualBox VM with 4 GB RAM, 2 CPU cores, and 40 GB disk space.
  - Set username and password for the Ubuntu Server.
- **Update and Upgrade Packages**:

  ```bash
  sudo apt-get update
  sudo apt-get upgrade
  ```

### 2. Odoo 18 Installation

- **Install Dependencies**:

  ```bash
  sudo apt-get install -y python3-pip python3-dev libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev build-essential libssl-dev libffi-dev libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev liblcms2-dev libblas-dev libatlas-base-dev
  ```
- **Install PostgreSQL and Create User**:

  ```bash
  sudo apt-get install -y postgresql
  sudo su - postgres
  createuser --createdb --username postgres --no-createrole --superuser --pwprompt odoo18
  exit
  ```
- **Install Git and Clone Odoo 18**:

  ```bash
  sudo adduser --system --home=/opt/odoo18 --group odoo18
  sudo apt-get install -y git
  sudo su - odoo18 -s /bin/bash
  git clone https://www.github.com/odoo/odoo --depth 1 --branch master --single-branch .
  exit
  ```
- **Set Up Python Virtual Environment**:

  ```bash
  sudo apt install -y python3-venv
  sudo python3 -m venv /opt/odoo18/venv
  sudo -s
  cd /opt/odoo18/
  source venv/bin/activate
  pip install -r requirements.txt
  ```
- **Configure Odoo**:
  - Copy and edit the configuration file:

    ```bash
    sudo cp /opt/odoo18/debian/odoo.conf /etc/odoo18.conf
    sudo nano /etc/odoo18.conf
    ```

    Add:

    ```ini
    [options]
    admin_passwd = admin
    db_host = localhost
    db_port = 5432
    db_user = odoo18
    db_password = 123456
    addons_path = /opt/odoo18/addons
    default_productivity_apps = True
    logfile = /var/log/odoo/odoo18.log
    ```
  - Set permissions:

    ```bash
    sudo chown odoo18: /etc/odoo18.conf
    sudo chmod 640 /etc/odoo18.conf
    sudo mkdir /var/log/odoo
    sudo chown odoo18:root /var/log/odoo
    ```
- **Create and Start Odoo Service**:

  ```bash
  sudo nano /etc/systemd/system/odoo18.service
  ```

  Add:

  ```ini
  [Unit]
  Description=Odoo18
  After=network.target postgresql.service
  Documentation=http://www.odoo.com
  [Service]
  Type=simple
  User=odoo18
  Group=odoo18
  ExecStart=/opt/odoo18/venv/bin/python3.12 /opt/odoo18/odoo-bin -c /etc/odoo18.conf
  Restart=always
  KillMode=mixed
  TimeoutStopSec=30
  [Install]
  WantedBy=multi-user.target
  ```

  Set permissions and start:

  ```bash
  sudo chmod 755 /etc/systemd/system/odoo18.service
  sudo chown root: /etc/systemd/system/odoo18.service
  sudo systemctl start odoo18.service
  ```
- **Verify Server IP**:

  ```bash
  hostname -I
  # or
  ip a
  ```

### 3. Initial Configuration

- **Create Database**: Access Odoo interface using credentials from `odoo18.conf`.
- **Log In**: Use the same credentials to log in.
- **Explore Dashboard**: Install and explore apps like Sales.
- **Update Settings**: Change password, create users, and configure company details (currency, language, timezone).

### 4. Advanced Configuration

Enable developer mode in General Settings to perform:

- **Discounts and Promotions**: Enable discounts, online signatures, pricelists, and create custom discount offers.
- **Custom Pricing Formula**: Define pricing rules for products.
- **Automated Actions**: Add a Python script for approval workflows:

  ```python
  approval_threshold = 5000
  if record.amount_total > approval_threshold:
      record.write({'state': 'sent'})
      manager = env['res.users'].search([('groups_id', '=', env.ref('sales_team.group_sale_manager').id)], limit=1)
      if manager:
          record.write({'user_id': manager.id})
      else:
          record.write({'state': 'sale'})
  ```
- **Custom Reports**: Modify report templates:

  ```xml
  <t t-name="sale.report_saleorder_document">
      <t t-call="web.external_layout">
          <div class="page">
              <h2>Quotation / Order: <t t-esc="doc.name"/></h2>
              <p>Customer: <t t-esc="doc.partner_id.name"/></p>
              <p>Total Amount: <t t-esc="doc.amount_total"/></p>
              <p>Discount: <t t-esc="sum(line.discount for line in doc.order_line)"/>%</p>
              <p>Pricelist: <t t-esc="doc.pricelist_id.name"/></p>
              <t t-if="doc.promotion_ids">
                  <p>Promotions:</p>
                  <ul>
                      <t t-foreach="doc.promotion_ids" t-as="promo">
                          <li><t t-esc="promo.name"/></li>
                      </t>
                  </ul>
              </t>
              <t t-if="doc.signature">
                  <p>Customer Signature:</p>
                  <img t-att-src="'data:image/png;base64,%s' % doc.signature"/>
              </t>
          </div>
      </t>
  </t>
  ```

### 5. Testing

- Create a customer and a discounted order.
- Generate an invoice for product sales.
- View graph reports for sales analytics.

---

## 🔍 Troubleshooting

### VirtualBox Issues

- **Black Screen**: Disable 3D acceleration or toggle EFI boot.
- **Network Issues**: Ensure host internet connectivity; switch between NAT and Bridged adapter.
- **CPU Error**: Use 64-bit Ubuntu and enable hardware virtualization in BIOS/UEFI.
- **Slow Performance**: Allocate more RAM/CPU or close host applications.
- **Guest Additions Failure**:

  ```bash
  sudo apt install -y build-essential dkms linux-headers-$(uname -r)
  ```

### Odoo Issues

- **Service Won’t Start**:

  ```bash
  sudo tail -f /var/log/odoo18/odoo18.log
  ```
- **Database Connection**:

  ```bash
  sudo systemctl status postgresql
  sudo -u postgres psql -c "\du"
  ```
- **Permission Issues**:

  ```bash
  ls -la /opt/odoo18/
  ls -la /var/log/odoo18/
  sudo chown -R odoo18:odoo18 /opt/odoo18/
  sudo chown -R odoo18:odoo18 /var/log/odoo18/
  ```

---

## 📁 Project Structure

```plaintext
.
├── odoo18/                    # Odoo 18 installation directory
├── etc/odoo18.conf            # Odoo configuration file
├── var/log/odoo/              # Odoo log directory
└── etc/systemd/system/odoo18.service  # Systemd service file
```

---

## ⚖️ License
This project is for **academic and personal skill development purposes only**.  
Reuse is allowed for **learning and research** with proper credit.

---
