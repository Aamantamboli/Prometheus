# Install Prometheus on AWS EC2
### Step 1: Create an AWS EC2 Instance
### Step 2: Connect EC2 instance and Install Prometheus
### Step 3: Create a Prometheus user without a home directory, and set up directories for configuration and data storage with the following commands.
```
sudo useradd --no-create-home prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
### Step 4: Download and extract Prometheus, then copy the binaries and console files to their respective locations with the following commands. [#Check here for latest version](https://prometheus.io/download/)
```
wget https://github.com/prometheus/prometheus/releases/download/v2.55.0-rc.1/prometheus-2.55.0-rc.1.linux-amd64.tar.gz 
tar -xvf prometheus-2.55.0-rc.1.linux-amd64.tar.gz
sudo cp prometheus-2.55.0-rc.1.linux-amd64/prometheus /usr/local/bin
sudo cp prometheus-2.55.0-rc.1.linux-amd64/promtool /usr/local/bin
sudo cp -r prometheus-2.55.0-rc.1.linux-amd64/consoles /etc/prometheus/
sudo cp -r prometheus-2.55.0-rc.1.linux-amd64/console_libraries /etc/prometheus
sudo cp prometheus-2.55.0-rc.1.linux-amd64/prometheus.yml /etc/prometheus/
```
### Step 5: Change the ownership of the Prometheus configuration and binary files to the prometheus user and group, including all files in the consoles and console_libraries directories, with the following commands.
```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus
```
### Step 6: Edit the Prometheus service file.
```
vim prometheus-2.55.0-rc.1.linux-amd64/prometheus.service 
```
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle \
    --log.level=info

[Install]
WantedBy=multi-user.target
```
### Step 7: Copy the Prometheus service file to the systemd directory.
```
sudo cp prometheus-2.55.0-rc.1.linux-amd64/prometheus.service /etc/systemd/system/
```
### Step 8: Initially and as a proof of concept we can configure Prometheus to monitor itself. All what we need to do is create or replace the content.
```
sudo vim /etc/prometheus/prometheus.yml
```
```
global:
  scrape_interval: 15s
  external_labels:
    monitor: 'prometheus'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```
### Step 9: Reload the systemd manager configuration, enable the Prometheus service to start on boot, start the Prometheus service, and check its status.
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```
### Step 10: Hit public ip of your instance and you will get the output.

![image](https://github.com/user-attachments/assets/572361a9-fc41-46ac-859e-d75ad71a6960)
