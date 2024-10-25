# Install Node Exporter on AWS EC2
### Step 1: Create an AWS EC2 Instance
### Step 2: Connect EC2 instance and Install Node Exporter
### Step 3: Create a dedicated user for Node Exporter.
```
sudo useradd --no-create-home node_exporter
```
### Step 4: Download and extract the Node Exporter, then copy the executable and the service file to the systemd directory. [#Check here for latest version](https://prometheus.io/download/)
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz 
tar xzf node_exporter-1.8.2.linux-amd64.tar.gz
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/node_exporter
```
### Step 5: Edit the Node exporter service file.
```
sudo vim node_exporter-1.8.2.linux-amd64/node-exporter.service 
```
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
### Step 6: Copy the Node exporter service file to the systemd directory.
```
sudo cp node_exporter-1.8.2.linux-amd64/node-exporter.service /etc/systemd/system/
```
### Step 7: Reload the systemd manager configuration, enable the Node Exporter service to start on boot, start the Node Exporter service, and check its status.
```
sudo systemctl daemon-reload
sudo systemctl enable node-exporter
sudo systemctl start node-exporter
sudo systemctl status node-exporter
```
### Step 8: Configure Prometheus Server
```
sudo vim /etc/prometheus/prometheus.yml
```
```
global:
  scrape_interval: 15s
  external_labels:
    monitor: 'prometheus'

scrape_configs:

  - job_name: 'node_exporter'

    static_configs:

      - targets: ['<public_ip>:9100']
```
### Step 9: Restart Prometheus service.
```
sudo systemctl restart prometheus
```
