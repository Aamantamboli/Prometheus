# Configure Prometheus Service Discovery
### Step 1: Create access key 
### Step 2: Configure Prometheus server
### Step 3: Edit the Prometheus service file.
```
sudo vim /etc/prometheus/prometheus.yml
```
```
global:
  scrape_interval: 1s
  evaluation_interval: 1s

scrape_configs:
  - job_name: 'node'
    ec2_sd_configs:
      - region: us-east-1
        access_key: PUT_THE_ACCESS_KEY_HERE
        secret_key: PUT_THE_SECRET_KEY_HERE
        port: 9100
```
### Step 4: Restart Prometheus service.
```
sudo systemctl restart prometheus
```
