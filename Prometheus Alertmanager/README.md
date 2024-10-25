# Prometheus Alertmanager

### Step 1: Install Prometheus on AWS EC2.[#Check here for installation](https://github.com/Aamantamboli/Prometheus/tree/main/Prometheus%20Installation)
### Step 2: Prometheus Node Exporter on AWS EC2.[#Check here for installation](https://github.com/Aamantamboli/Prometheus/tree/main/Node%20Exporter%20Installation)
### Step 3: Prometheus Discovery Service on AWS EC2.[#Check here for installation](https://github.com/Aamantamboli/Prometheus/tree/main/Prometheus%20Service%20Discovery)
### Step 4: Connect Prometheus server and install Alertmanager.[#Check here for latest version](https://prometheus.io/download/)
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.21.0/alertmanager-0.21.0.linux-amd64.tar.gz
tar xvfz alertmanager-0.21.0.linux-amd64.tar.gz
sudo cp alertmanager-0.21.0.linux-amd64/alertmanager /usr/local/bin
sudo cp alertmanager-0.21.0.linux-amd64/amtool /usr/local/bin/
sudo mkdir /var/lib/alertmanager
```
### Step 5: Add Alertmanager’s configuration
```
sudo vim /etc/prometheus/alertmanager.yml
```
```
route:
  group_by: [Alertname]
  receiver: email-me

receivers:
- name: email-me
  email_configs:
  - to: EMAIL_YO_WANT_TO_SEND_EMAILS_TO
    from: YOUR_EMAIL_ADDRESS
    smarthost: smtp.gmail.com:587
    auth_username: YOUR_EMAIL_ADDRESS
    auth_identity: YOUR_EMAIL_ADDRESS
    auth_password: YOUR_EMAIL_PASSWORD
```
### Step 6: Configure Alertmanager as a service.
```
[Unit]
Description=Alert Manager
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/prometheus/alertmanager.yml \
  --storage.path=/var/lib/alertmanager

Restart=always

[Install]
WantedBy=multi-user.target
```
### Step 7: Configure Systemd
```
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
```
### Step 8: Generate an App Password
 #### Step i: Go to your account: https://myaccount.google.com
    For this step it is also required the following:
    1.2fa Verification is set up for your account.
    2.2fa Verification is not set up for security keys only.
    3.Your account is not through work, school, or other organization.
    4.You’ve not turned on Advanced Protection for your account.
  
#### Step ii: Search for app password.

![Screenshot 2024-10-25 171128](https://github.com/user-attachments/assets/0b289f01-14af-4de4-b7c3-5643cc7e33cc)

#### Step iii: Give app name then create it and copy that 16-digit passcode.

![Screenshot 2024-10-25 171202](https://github.com/user-attachments/assets/fb028101-8a11-4ee2-97c8-2652be84f298)

### Step 9: Create a Rule
```
sudo vim /etc/prometheus/rules.yml
```
This is just a simple alert rule. In a nutshell it alerts when an instance has been down for more than 3 minutes. 
```
groups:
- name: Down
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 3m
    labels:
      severity: 'critical'
    annotations:
      summary: "Instance  is down"
      description: " of job  has been down for more than 3 minutes."
```
### Step 10: Configure Prometheus
```
sudo chown -R prometheus:prometheus /etc/prometheus
```
### Step 11: Edit Prometheus configuration file.
```
global:
  scrape_interval: 1s
  evaluation_interval: 1s

rule_files:
 - /etc/prometheus/rules.yml

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093

scrape_configs:
  - job_name: 'node'
    ec2_sd_configs:
      - region: us-east-1
        access_key: PUT_THE_ACCESS_KEY_HERE
        secret_key: PUT_THE_SECRET_KEY_HERE
        port: 9100
```
### Step 12: Reload Systemd
```
sudo systemctl restart prometheus
```
## Try It Out
## Turn off the Node Exporter AWS EC2 Instance
## Wait for 3 minutes and check the Alertmanager URL that is installed in your prometheus-server instance
## Check your email
