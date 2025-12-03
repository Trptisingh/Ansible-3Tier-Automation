# **Ansible 3-Tier AWS Automation – End-to-End Infrastructure Provisioning**

This repository contains a complete automation solution for deploying a **3-Tier production-style application** on **AWS EC2** using **Ansible**.
The setup includes dedicated **Web**, **Application**, and **Database** tiers, each deployed and configured through **Ansible roles** following industry best practices.

---

# 🏢 **Problem Statement (Industry-Level)**

A growing EdTech company is preparing to launch its new student-learning platform.
To support their microservices adoption, they require:

* A **scalable and secure 3-Tier architecture**
* A fully automated **repeatable infrastructure deployment**
* Zero manual configuration across servers
* Capability to deploy updates with a single command

The platform must consist of:

### **1️⃣ Database Tier (MySQL)**

Stores student records, session data, progress tracking, and application metadata.

### **2️⃣ Application Tier (Flask API)**

Processes business logic such as:

* Student login
* Course metadata
* Quiz and assessment processing
* API responses to the web tier

### **3️⃣ Web Tier (Nginx)**

Exposes:

* User-facing web page
* Static content
* Routing to backend application

The DevOps team must automate this entire system using **Ansible**, ensuring:

* Idempotent deployments
* Modular role design
* Secure SSH key distribution
* Zero-touch provisioning
* Cloud-agnostic Ansible standards

This repository implements the **production-grade automation** for this system.

---

# 📁 **Repository Structure**

> Matches your exact folder layout — **hosts file directly in project root**, not inside inventory/.

```
ansible-3tier/
│
├── hosts                  # Ansible inventory (web, app, db groups)
├── site.yml               # Main Ansible orchestration playbook
│
└── roles/
    ├── dbserver/
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── files/
    │   │   └── schema.sql
    │   └── handlers/
    │       └── main.yml
    │
    ├── appserver/
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── templates/
    │   │   └── flaskapp.service.j2
    │   ├── files/
    │   │   └── flask_app/
    │   │       ├── app.py
    │   │       └── requirements.txt
    │   └── handlers/
    │       └── main.yml
    │
    └── webserver/
        ├── tasks/
        │   └── main.yml
        ├── templates/
        │   └── index.html.j2
        └── handlers/
            └── main.yml
```

---

# 🧱 **Solution Overview**

The automation provisions:

## **1️⃣ Database Tier — MySQL (dbserver)**

* Installs MySQL server
* Creates database: `app_db`
* Creates DB user: `app_user`
* Grants remote privileges
* Imports schema
* Enables & starts MySQL service

---

## **2️⃣ Application Tier — Python Flask API (appserver)**

* Installs Python3, pip, venv, git
* Deploys Flask source code
* Creates virtual environment
* Installs requirements
* Configures systemd service: `flaskapp.service`
* Starts Flask on port **5000**

---

## **3️⃣ Web Tier — Nginx (webserver)**

* Installs Nginx
* Deploys Jinja2-powered index page
* Displays dynamic hostname
* Routes traffic to backend appserver
* Enables & restarts nginx

---

# 🏗 **High-Level Architecture**

```
                ┌───────────────────────────┐
                │         Internet           │
                └───────────────┬───────────┘
                                │
                                v
                    ┌───────────────────────┐
                    │     Web Server        │
                    │       (Nginx)         │
                    └─────────────┬─────────┘
                                  │
                                  v
                      ┌─────────────────────┐
                      │    App Server       │
                      │   (Flask API)       │
                      └───────────┬─────────┘
                                  │
                                  v
                        ┌───────────────────┐
                        │     DB Server     │
                        │     (MySQL)        │
                        └────────────────────┘
```

---

# ✅ **Usage – How to Use This Project (Step-by-Step Execution Guide)**

This section explains **exactly how to deploy the full 3-tier architecture** using the code already present in this repository.
All YAML files, roles, handlers, templates, and folder structures are already included — **you only follow the steps**.

---

# ⚙️ **1. Prerequisites**

### **A. AWS Setup**

Before using this project, ensure the following EC2 instances are created in AWS:

| Instance Name   | Purpose          | OS           | Type     |
| --------------- | ---------------- | ------------ | -------- |
| ansible-control | Ansible Master   | Ubuntu 22.04 | t2.micro |
| webserver1      | Web Tier (Nginx) | Ubuntu 22.04 | t2.micro |
| appserver1      | App Tier (Flask) | Ubuntu 22.04 | t2.micro |
| dbserver1       | DB Tier (MySQL)  | Ubuntu 22.04 | t2.micro |

Requirements:

* All instances must have **public IP**
* Use same **default security group**
* Download AWS SSH key → `ansible.pem`

---

# ⚙️ **2. Prepare Control Node (Ansible Master)**

SSH into **ansible-control** and install Ansible:

```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

---

# ⚙️ **3. Upload Your PEM File From Windows → Control Node**

From your Windows system:

```
scp -i C:\Users\lenovo\Downloads\ansible.pem C:\Users\lenovo\Downloads\ansible.pem ubuntu@<CONTROL_NODE_PUBLIC_IP>:/home/ubuntu/
```

Fix PEM permissions on control node:

```
chmod 400 ansible.pem
```

---

# ⚙️ **4. Validate SSH From Control Node → All Worker Nodes**

```
ssh -i ansible.pem ubuntu@<WEB_PUBLIC_IP>
ssh -i ansible.pem ubuntu@<APP_PUBLIC_IP>
ssh -i ansible.pem ubuntu@<DB_PUBLIC_IP>
```

If SSH works using the PEM → proceed.

---

# ⚙️ **5. Generate Ansible SSH Key (For Passwordless SSH)**

```
ssh-keygen
```

This generates:

```
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

---

# ⚙️ **6. Copy Ansible SSH Key to All Worker Nodes**

A playbook is already included (`copy-key.yml`).
Run it:

```
ansible-playbook -i hosts copy-key.yml
```

If successful, test:

```
ssh ubuntu@<WEB_PUBLIC_IP>
ssh ubuntu@<APP_PUBLIC_IP>
ssh ubuntu@<DB_PUBLIC_IP>
```

→ No password or PEM required.

---

# ⚙️ **7. Verify Project Directory Structure**

SSH into control node and clone/download your repo:

```
git clone <YOUR_REPO_URL>
cd ansible-3tier
```

This repo already contains:

* `site.yml`
* `hosts` inventory
* `roles/` for dbserver, appserver, webserver
* All tasks, templates, handlers

No need to create anything manually.

---

# ⚙️ **8. Update Inventory File with Your Server IPs**

Open the `hosts` file in the root directory:

```
nano hosts
```

Add:

```
[web]
<WEB_PUBLIC_IP>

[app]
<APP_PUBLIC_IP>

[db]
<DB_PUBLIC_IP>
```

Save.

---

# ⚙️ **9. Run the Complete 3-Tier Deployment**

From the project root:

```
ansible-playbook -i hosts site.yml
```

This will:

### DB Tier

* Install MySQL
* Create DB & user
* Import schema

### App Tier

* Deploy Flask code
* Create virtualenv
* Install requirements
* Start systemd service

### Web Tier

* Install Nginx
* Deploy index.html
* Start service

---

# ⚙️ **10. Validate the Deployment**

### **A. Web Tier**

Open browser:

```
http://<WEB_PUBLIC_IP>
```

You should see the web page.

---

### **B. App Tier**

SSH into app server:

```
ssh ubuntu@<APP_PUBLIC_IP>
```

Check systemd status:

```
sudo systemctl status flaskapp
```

Check the app:

```
curl http://<APP_PUBLIC_IP>:5000
```

---

### **C. DB Tier**

```
ssh ubuntu@<DB_PUBLIC_IP>
```

Check database:

```
sudo mysql -e "SHOW DATABASES;"
sudo mysql -e "USE app_db; SHOW TABLES;"
```

---


# 📌 **Key Features**

* Complete end-to-end automation
* Zero manual server setup
* Role-based modular structure
* SSH automation (passwordless)
* Templated systemd & HTML configs
* AWS-ready EC2 deployment model
* Real production-style 3-tier design

---
