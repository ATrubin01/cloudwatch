# Prometheus, Node Exporter, and Grafana Setup
## Overview
This document outlines the steps to set up Prometheus for monitoring, Node Exporter for system metrics collection, and Grafana for visualization on three EC2 instances.

## 1. Install Prometheus
Download Prometheus:

```
wget https://github.com/prometheus/prometheus/releases/download/v2.1.0/prometheus-2.1.0.linux-amd64.tar.gz
```
Extract the Archive:

```
tar -xf prometheus-2.1.0.linux-amd64.tar.gz
```
Move Binaries to /usr/local/bin:

```
sudo mv prometheus-2.1.0.linux-amd64/prometheus prometheus-2.1.0.linux-amd64/promtool /usr/local/bin
```
Create Directories:

```
sudo mkdir /etc/prometheus /var/lib/prometheus
```
Move Configuration Files:

```
sudo mv prometheus-2.1.0.linux-amd64/consoles prometheus-2.1.0.linux-amd64/console_libraries /etc/prometheus
```
Remove Leftover Files:

```
rm -r prometheus-2.1.0.linux-amd64*
```
Configure Prometheus:

Edit /etc/prometheus/prometheus.yml to include Node Exporter as a target:

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<node-exporter-ip>:9100']
```
Change Ownership:

```
sudo useradd -rs /bin/false prometheus
sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus
```
Create Systemd Unit File for Prometheus:

```
sudo vim /etc/systemd/system/prometheus.service
```
Add the following contents:

```
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
Reload systemd and Start Prometheus:

```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```
Verify Prometheus UI:

Visit http://<prometheus-ip>:9090 in browser.

## 2. Setup Node Exporter

## Install Node Exporter
1. Download Node Exporter:

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
```
Extract the Archive:

```
tar -xvf node_exporter-1.1.2.linux-amd64.tar.gz
```
Move Binary to /usr/local/bin:

```
sudo mv node_exporter-1.1.2.linux-amd64/node_exporter /usr/local/bin/
```
Remove Residual Files:

```
rm -rf node_exporter-1.1.2.linux-amd64/*
```
Create User and Service File:

```
sudo useradd -rs /bin/false node_exporter
sudo nano /etc/systemd/system/node_exporter.service
```
Add the following contents:

```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
Reload systemd, Enable, and Start Node Exporter:

```
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```
Verify Node Exporter:

Visit http://<node-exporter-ip>:9100/metrics in browser.

## 3. Setup Grafana
## Install Grafana

Add Grafana YUM Repository:

```
sudo vim /etc/apt/sources.list.d/grafana.list
```
Add the following lines:

```
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
Install Grafana:

```
sudo apt-get install -y grafana
```
Reload systemd, Start Grafana, and Enable at Boot:

```
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service
sudo systemctl status grafana-server
```
Access Grafana:

Visit http://<grafana-ip>:3000 in browser. Default credentials are:

Username: admin
Password: admin
## 4. Configure Grafana and Prometheus
Add Prometheus Data Source in Grafana:

Go to Configuration -> Data Sources in Grafana.
Add Prometheus and set the URL to http://<prometheus-ip>:9090.
Create Dashboards:


<img width="1015" alt="Screenshot 2024-08-18 at 2 33 52 PM" src="https://github.com/user-attachments/assets/8664b9c0-e0d6-4f84-9262-9d30ef2dd8bd">


# CloudWatch Agent

## IAM Role and Policies
  - Created an EC2 instance role with the following policies:
    - AmazonEC2RoleforSSM
    - CloudWatchAgentServerPolicy

## 1. Install amazon-cloudwatch-agent
```
sudo apt-get install amazon-cloudwatch-agent -y
```
## 2. Launch cloudwatch wizard
```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```
## 3. Go through the questions
## 4. Go inside the amazon-cloudwatch-agent/bin folder
```
cd /opt/aws/amazon-cloudwatch-agent/bin
```
## 5. Start Cloudwatch agent
```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a start -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
```
![Screenshot 2024-08-18 at 4 39 56 PM](https://github.com/user-attachments/assets/3949f717-4dfe-4616-8594-36beaf438779)

