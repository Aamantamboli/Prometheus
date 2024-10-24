# <div align="center">Prometheus-Graffana</div> 

<p align="center">
  <img src="https://github.com/user-attachments/assets/a4b8e18b-e787-424b-a5c4-b0f7c5939945"/> <img src="https://github.com/user-attachments/assets/cf131f80-e956-4a29-82d8-8f2a0b67cded"/>
</p>

## Prometheus
### Step 1: Create two instance.

![Screenshot 2024-10-22 100034](https://github.com/user-attachments/assets/f15f4432-b584-4c0e-b7da-44dfcc7e3249)

### Step 2: Create a Prometheus user without a home directory, and set up directories for configuration and data storage with the following commands.
```
sudo useradd --no-create-home prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
### Step 3: Download and extract Prometheus, then copy the binaries and console files to their respective locations with the following commands. [#Check here for latest version](https://prometheus.io/download/)
```
wget https://github.com/prometheus/prometheus/releases/download/v2.55.0-rc.1/prometheus-2.55.0-rc.1.linux-amd64.tar.gz 
tar -xvf prometheus-2.55.0-rc.1.linux-amd64.tar.gz
sudo cp prometheus-2.55.0-rc.1.linux-amd64/prometheus /usr/local/bin
sudo cp prometheus-2.55.0-rc.1.linux-amd64/promtool /usr/local/bin
sudo cp -r prometheus-2.55.0-rc.1.linux-amd64/consoles /etc/prometheus/
sudo cp -r prometheus-2.55.0-rc.1.linux-amd64/console_libraries /etc/prometheus
sudo cp prometheus-2.55.0-rc.1.linux-amd64/prometheus.yml /etc/prometheus/
```
### Step 4: Change the ownership of the Prometheus configuration and binary files to the `prometheus` user and group, including all files in the consoles and console_libraries directories, with the following commands.
```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus
```
### Step 5: Edit the Prometheus service file.
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
sudo cp prometheus-2.55.0-rc.1.linux-amd64/prometheus.service /etc/systemd/system/prometheus.service
```
### Step 8: Reload the systemd manager configuration, enable the Prometheus service to start on boot, start the Prometheus service, and check its status.
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```
### Step 9: Edit the Prometheus configuration file.
```
...
- job_name: "remote_collector"
  scrape_interval: 10s
  static_configs:
    - targets: ["remote_addr:9100"] #give webserver ip
```
### Step 10: Connect webserver instance and create a dedicated user for Node Exporter.
```
sudo useradd --no-create-home node_exporter
```
### Step 11: Download and extract the Node Exporter, then copy the executable and the service file to the systemd directory. [#Check here for latest version](https://prometheus.io/download/)
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz 
tar xzf node_exporter-1.8.2.linux-amd64.tar.gz
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/node_exporter
```
### Step 12: Edit the service file.
```
vim node_exporter-1.8.2.linux-amd64/node-exporter.service 
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
### Step 13: Copy the file 
```
sudo cp node_exporter-1.8.2.linux-amd64/node-exporter.service /etc/systemd/system/prometheus.service
```
### Step 14: Reload the systemd manager configuration, enable the Node Exporter service to start on boot, start the Node Exporter service, and check its status.
```
sudo systemctl daemon-reload
sudo systemctl enable node-exporter
sudo systemctl start node-exporter
sudo systemctl status node-exporter
```
### Step 15: Hit public ip in browser. 

![Screenshot 2024-10-22 110704](https://github.com/user-attachments/assets/382f9072-a2c4-43f7-83f1-d983a25aaa20)

## Graffana [#Check here for latest version](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/)
### Step 1: Install the prerequisite packages.
```
sudo apt-get install -y apt-transport-https software-properties-common wget
```
### Step 2: Import the GPG key.
```
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```
### Step 3: To add a repository for stable releases, run the following command.
```
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
### Step 4: Run the following command to update the list of available packages.
```
# Updates the list of available packages
sudo apt-get update
```
### Step 5: To install Grafana OSS, run the following command.
```
# Installs the latest OSS release:
sudo apt-get install grafana
```
