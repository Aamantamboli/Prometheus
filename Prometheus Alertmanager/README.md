Here’s the complete README.md file with all the steps from the setup process for Prometheus, Node Exporter, and Alertmanager:

# Prometheus, Node Exporter, and Alertmanager Setup

This guide provides instructions to set up Prometheus, Node Exporter, and Alertmanager on a Linux server.

## Prerequisites

- A Linux server with sudo privileges.
- Basic knowledge of command line operations.
- A Google account with 2-Step Verification (2FA) enabled.

### Step 1: Create Prometheus User and Directories

```bash
sudo useradd --no-create-home prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

### Step 2: Download and Install Prometheus [#Check here for latest version of prometheus](https://prometheus.io/download/)

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.55.0/prometheus-2.55.0.linux-amd64.tar.gz
tar xvfz prometheus-2.55.0.linux-amd64.tar.gz
sudo cp prometheus-2.55.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.55.0.linux-amd64/promtool /usr/local/bin/
sudo cp -r prometheus-2.55.0.linux-amd64/consoles/ /etc/prometheus/
sudo cp -r prometheus-2.55.0.linux-amd64/console_libraries/ /etc/prometheus/
```

### Step 3: Create Prometheus Systemd Service

```bash
sudo vim /etc/systemd/system/prometheus.service
```

Add the following configuration:

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

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

### Step 4: Set Permissions

```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

### Step 5: Start Prometheus Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

### Step 6: Install Node Exporter [#Check here for latest version of node exporter](https://prometheus.io/download/)

```bash
sudo useradd --no-create-home node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xzf node_exporter-1.8.2.linux-amd64.tar.gz
sudo cp node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/node_exporter
```

### Step 7: Create Node Exporter Systemd Service

```bash
sudo vim /etc/systemd/system/node-exporter.service
```

Add the following configuration:

```ini
[Unit]
Description=Prometheus Node Exporter Service
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

### Step 8: Start Node Exporter Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable node-exporter
sudo systemctl start node-exporter
sudo systemctl status node-exporter
```

### Step 9: Download and Install Alertmanager [#Check here for latest version for alertmanager](https://prometheus.io/download/)

```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar xvfz alertmanager-0.27.0.linux-amd64.tar.gz
sudo cp alertmanager-0.27.0.linux-amd64/alertmanager /usr/local/bin/
sudo cp alertmanager-0.27.0.linux-amd64/amtool /usr/local/bin/
sudo mkdir /var/lib/alertmanager
```

Here’s the steps for generating an app password for use with Prometheus Alertmanager:

## Generating an App Password for Prometheus Alertmanager

This guide explains how to generate an app password for your Google account to be used with Prometheus Alertmanager when 2-Step Verification (2FA) is enabled.

### Step 10: Go to Your Account

Visit [Google Account Management](https://myaccount.google.com) and ensure the following:

1. 2FA Verification is set up for your account.
2. 2FA Verification is not set up for security keys only.
3. Your account is not through work, school, or other organization.
4. You have not turned on Advanced Protection for your account.

### Step 11: Search for App Password

1. In the Google Account settings, search for "app password."

   ![Screenshot 2024-10-25 171128](https://github.com/user-attachments/assets/29b600da-1ba2-4ded-bfe4-10ce0e7835a8)

### Step 12: Generate App Password

1. Select "Generate App Password."
2. Give your app a name (e.g., "Prometheus Alertmanager") and create it.
3. Copy the generated 16-digit passcode.

   ![Screenshot 2024-10-25 171202](https://github.com/user-attachments/assets/49f8079a-f12b-494b-8102-7f94d5b25dd0)

## Conclusion

You have successfully generated an app password for your Google account. Use this 16-digit passcode in your Prometheus Alertmanager configuration to authenticate with your email account.

### Step 13: Configure Alertmanager

```bash
sudo vim /etc/prometheus/alertmanager.yml
```

Add the following configuration:

```yaml
# Group alerts by the Alertname label
route:
  group_by: [Alertname]  # This determines how alerts are grouped for notifications
  receiver: email-me     # Specify the receiver for the alerts

# Define the receivers section
receivers:
- name: email-me        # Name of the receiver configuration
  email_configs:        # Configuration for sending email
  - to: EMAIL_YO_WANT_TO_SEND_EMAILS_TO  # Recipient email address
    from: YOUR_EMAIL_ADDRESS                # Sender email address (should match the SMTP account)
    smarthost: smtp.gmail.com:587          # SMTP server and port (for Gmail)
    auth_username: YOUR_EMAIL_ADDRESS       # SMTP authentication username (your email address)
    auth_identity: YOUR_EMAIL_ADDRESS       # Identity for authentication (usually same as username)
    auth_password: YOUR_EMAIL_PASSWORD      # SMTP authentication password (use an app-specific password if 2FA is enabled)
```

### Step 14: Create Alertmanager Systemd Service

```bash
sudo vim /etc/systemd/system/alertmanager.service
```

Add the following configuration:

```ini
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

### Step 15: Start Alertmanager Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
sudo systemctl status alertmanager
```

### Step 16: Create Alert Rules

```bash
sudo vim /etc/prometheus/rules.yml
```

Add the following configuration:

```yaml
groups:
- name: Down  # Name of the alerting group
  rules:
  - alert: InstanceDown  # Name of the alert
    expr: up == 0  # Expression to trigger the alert when the instance is down
    for: 3m  # Duration for which the condition must be true before firing the alert
    labels:
      severity: 'critical'  # Severity level of the alert
    annotations:
      summary: "Instance {{ $labels.instance }} is down"  # Brief summary of the alert
      description: "Instance {{ $labels.instance }} has been down for more than 3 minutes."  # Detailed description of the alert
```
### Step 17: Create Access Key

1. Log in to your AWS Management Console.
2. Navigate to the IAM (Identity and Access Management) service.
3. Select "Users" from the sidebar.
4. Choose your username.
5. Click on the "Security credentials" tab.
6. Under "Access keys," click on "Create access key."
7. Save the Access

### Step 18: Configure Prometheus to Use Alert Rules

```bash
sudo vim /etc/prometheus/prometheus.yml
```

Add or update the following configuration:

```yaml
global:
  scrape_interval: 1s  # How often to scrape targets (default is 1 minute)
  evaluation_interval: 1s  # How often to evaluate rules (default is 1 minute)

rule_files:
 - /etc/prometheus/rules.yml  # Path to the alerting rules file

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093  # Address of the Alertmanager instance

scrape_configs:
  - job_name: 'node'  # Name of the job for scraping metrics
    ec2_sd_configs:  # Configuration for EC2 service discovery
      - region: ap-south-1  # AWS region where EC2 instances are located
        access_key: PUT_THE_ACCESS_KEY_HERE  # Your AWS access key
        secret_key: PUT_THE_SECRET_KEY_HERE  # Your AWS secret key
        port: 9100  # Port where Node Exporter is running
```

### Step 19: Set Permissions for Prometheus Configuration

```bash
sudo chown -R prometheus:prometheus /etc/prometheus
```

### Step 20: Restart Prometheus Service

```bash
sudo systemctl restart prometheus
```

## Security Group Configuration

Ensure that your server's security group allows inbound traffic on the following ports:

- **Prometheus**: Port `9090` (for web interface)
- **Alertmanager**: Port `9093` (for web interface)
- **Node Exporter**: Port `9100` (for metrics)

To configure your security group, add rules to allow inbound traffic on these ports from your desired IP ranges or CIDR blocks.

## Conclusion

You have successfully set up Prometheus, Node Exporter, and Alertmanager. Make sure to replace placeholders like email addresses and AWS credentials with your actual values. You can access Prometheus at `http://your-server-ip:9090` and Alertmanager at `http://your-server-ip:9093`.
