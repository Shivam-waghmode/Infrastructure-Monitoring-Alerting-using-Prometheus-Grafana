# 🚀 Real-Time Infrastructure Monitoring & Alerting

## 📌 Project Overview
This project implements a real-time monitoring and alerting system using Prometheus and Grafana.

It replaces manual SSH-based health checks with automated monitoring of:
- CPU usage
- Memory utilization
- Disk usage

Alerts are triggered automatically when system thresholds are exceeded.

---

## 🎯 Objective
- Collect metrics from multiple EC2 servers  
- Centralize monitoring using Prometheus  
- Visualize data using Grafana dashboards  
- Trigger alerts when CPU usage exceeds 70%  

---

## 🏗️ Architecture

App Servers (Node Exporter) ---> Prometheus ---> Grafana ---> Alerts

---

## ⚙️ Technologies Used
- AWS EC2  
- Prometheus  
- Grafana  
- Node Exporter  

---

# 🚀 Monitoring Setup with Prometheus & Grafana

## 🔹 1. Monitoring Server (Prometheus + Grafana)

- Launch **1 EC2 instance**
- Install **Prometheus** and **Grafana**
- #### Install Prometheus
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz
tar -xvf prometheus-linux-amd64.tar.gz
sudo mv prometheus-linux-amd64 prometheus
cd prometheus/
./prometheus --config.file=prometheus.yml
```
- Access Prometheus **http://EC2-IP:9090**
- #### Install Grafana
```bash
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install -y grafana
```
- Access Grafana **http://EC2-IP:3000**
- Default login **admin / admin**

  ## 🔹 2. Launch 2 Application Server 

- Launch **2 EC2 instance**
- Install **Node Exporter**
```bash
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-1.8.2.linux-amd64.tar.gz
tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
sudo mv node_exporter-1.8.2.linux-amd64.tar.gz nodeapp
cd nodeapp/
./node_exporter
nohup ./node_exporter > node_exporter.log 2>&1 &
```
- Add Port in Security Group- **9100**
- Verify on browser- **http://APP_SERVER_IP:9100/metrics**

 ## 🔹 3. Configure Prometheus scrape targets on Monitoring Server
 - nano prometheus.yml
  ```bash
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "<APP_SERVER_1_IP>:9100"
          - "<APP_SERVER_2_IP>:9100"
 ```
-  **Start Prometheus**
 ```bash
sudo systemctl restart prometheus
 ```
- Verify on browser- **http://MONITER_SERVER_IP:9090/Targets**

## 🔗 Connect Grafana to Prometheus
1. Go to Settings → **Data Sources**
2. Click Add Data Source
3. Select **Prometheus**
4. URL:
   ```bash
   http://localhost:9090
    ```
5. Click **Save & Test**
   
## 📥 Import Node Exporter Dashboard
1. Go to **+ → Import**
2. Enter **Dashboard ID:**
 ```bash
   1860
 ```
3. Select **Prometheus**
4. Click **Import**

## 🛠️ Custom Dashboard Panels
### Step 1: Create New Dashboard

1. Click **+ → Dashboard**
2. Click **Add New Panel**

---

## 🔹 Create CPU Usage Panel

### Query:
```promql
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```
## 🔹 Create Memory Utilization Panel

### Query:
```promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```
## 🔹 Create Disk Usage Panel

### Query:
```promql
(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100
```
## 🚨 Alerting Setup (Prometheus + Grafana + SMTP Email)

This section explains how to configure alerting using:
- **Prometheus alert.rules.yml**
- **Grafana alert rules**
- **SMTP Email notifications (Gmail)**

---

## 🧩 Alerting Architecture
Node Exporter → Prometheus → Grafana → Email (SMTP)

- Prometheus evaluates alert conditions  
- Grafana sends notifications via SMTP  

---

## PART 1: Prometheus Alert Rule (alert.rules.yml)

## 📁 Step 1: Create Alert File

```bash
sudo nano /etc/prometheus/alert.rules.yml
```
## 📝 Step 2: Add Alert Rule (Text File at Alert-RULE.yml)

## ⚙️ Step 3: Link Rule in Prometheus
    Edit config:
     sudo nano /etc/prometheus/prometheus.yml
  
 ##    ADD:
 ```yaml
    rule_files:
       -"alert.rules.yml"
  ```
 ## 🔄 Step 4: Restart Prometheus
       sudo systemctl restart prometheus
      
  ## ✅ Step 5: Verify
       http://<SERVER_IP>:9090
       
  ##    Go to:
        Status → Rules.
     
## PART 2: Grafana SMTP Email Setup

## 📁 Step 1: Configure SMTP
```bash
sudo nano /etc/grafana/grafana.ini
```
Update:
```bash
[smtp]
enabled = true
host = smtp.gmail.com:587
user = your-email@gmail.com
password = your-app-password
from_address = your-email@gmail.com
from_name = Grafana Alerts
skip_verify = true
```
## 🔐 Gmail Requirements
- **Enable 2-Step Verification**
- **Generate App Password**
- **Use App Password (not Gmail password)**

## 🔄 Restart Grafana
```bash
sudo systemctl restart grafana-server
```

## PART 3: Grafana Alert Rule
 ## 📊 Step 1: Create Alert Rule
Open Grafana:
```bash
http://Monitor_SERVER_IP:3000
```
## Go to:
   - Alerting → Alert Rules
   - Click New Alert Rule

## 📈 Step 2: Define Query
 ```promql
 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)
```

## 🚨 Step 3: Set Condition
   - Condition: IS ABOVE 70
   - For: 1 minute
   - Evaluate every: 1 minute

## 🏷️ Step 4: Rule Details
  - Name: HighCPUUsage
  - Folder: General
  - Group: system_alerts

## 📝 Step 5: Labels & Annotations
```promql
labels:
  severity: warning
annotations:
  summary: "High CPU usage detected"
  description: "CPU usage is above 70% for 1 minute"
```
## PART 4: Email Notification Setup
## 📩 Step 1: Create Contact Point
Go to:
 - **Alerting → Contact Points**
 - Click New **Contact Point**
Configuration:
 - Name: **Email-Alerts**
 - Type: **Email**
 - Email: **your-email@gmail.com**
## 👉 Click Save

## 🔗 Step 2: Link Alert to Email
Open your alert rule
In Notifications, select:
 - **Email-Alerts**

## 🔹 PART 5: Test Alert
## 🔧 Install stress tool
```bash
sudo apt install stress
```
## 🔥 Generate CPU Load
```bash
stress --cpu 2 --timeout 120
```
## 📬 Expected Output
   - CPU > 70%
   - Alert triggers after 1 minute
   - Email sent to your inbox
   
   ---
