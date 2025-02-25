# Prometheus & Grafana without helm

Installing and setting up 𝐏𝐫𝐨𝐦𝐞𝐭𝐡𝐞𝐮𝐬 and 𝐆𝐫𝐚𝐟𝐚𝐧𝐚 to monitor both 𝐆𝐨𝐨𝐠𝐥𝐞 𝐂𝐥𝐨𝐮𝐝 𝐏𝐥𝐚𝐭𝐟𝐨𝐫𝐦 (𝐆𝐂𝐏) 𝐕𝐢𝐫𝐭𝐮𝐚𝐥 𝐌𝐚𝐜𝐡𝐢𝐧𝐞𝐬 and 𝐃𝐨𝐜𝐤𝐞𝐫 𝐜𝐨𝐧𝐭𝐚𝐢𝐧𝐞𝐫𝐬 using 𝐍𝐨𝐝𝐞 𝐄𝐱𝐩𝐨𝐫𝐭𝐞𝐫 and 𝐜𝐀𝐝𝐯𝐢𝐬𝐨𝐫.


![1_1AQ_No5TEWUwZf6nCLHUfg](https://github.com/user-attachments/assets/4759c1ff-0166-43ff-abc6-3671e5da10d2)
- Here’s a complete guide to install and set up `Prometheus and Grafana` on an Ubuntu VM from scratch.
# Prometheus setup:
# Step 1: Update Your System
- First, make sure your system packages are up-to-date:
```
sudo apt update
sudo apt upgrade -y
```
# Step 2: Install Prometheus
- a. Download and `Install Prometheus`
1. Download the latest Prometheus package:
```
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
```
- Replace `v2.43.0` with the latest version if necessary.
2. Extract the tarball:
```
tar xvf prometheus-2.43.0.linux-amd64.tar.gz
```
3. Move the binaries to `/usr/local/bin`:
```
sudo mv prometheus-2.43.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.43.0.linux-amd64/promtool /usr/local/bin/
```
4. Move the configuration files and create necessary directories:
```
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo mv prometheus-2.43.0.linux-amd64/prometheus.yml /etc/prometheus/
sudo mv prometheus-2.43.0.linux-amd64/consoles /etc/prometheus/
sudo mv prometheus-2.43.0.linux-amd64/console_libraries /etc/prometheus/
```
# b. Create a Prometheus User and Group
1. Create a `user` and `group` for `Prometheus`:
```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo groupadd prometheus
```
2. Set ownership of the directories:
```
sudo chown -R prometheus:prometheus /usr/local/bin/prometheus
sudo chown -R prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```
# c. Create a Prometheus Systemd Service File
1. Create and edit the systemd service file for Prometheus:
```
sudo vim /etc/systemd/system/prometheus.service
```
2. Add the following configuration:
```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/

[Install]
WantedBy=multi-user.target
```
# 1. [Unit] Section
This section provides metadata and dependencies for the service.
- `Description=Prometheus:` A short description of the service. In this case, it's simply labeled "Prometheus".
- `Documentation=https://prometheus.io/docs/introduction/overview/:` A link to the official Prometheus documentation for more information.
- `After=network.target:` This ensures that Prometheus starts only after the network has been initialized. This is important because Prometheus often relies on network connectivity to scrape metrics from other services.
# 2. [Service] Section:
This section defines how the service should be started and run.
- `User=prometheus:` Specifies that the service should run under the `prometheus user` account. This is a security best practice to ensure the service runs with limited privileges.
- Group=prometheus: Specifies that the service should also run under the `prometheus group`. It helps maintain control over file permissions.
- `Type=simple:` Defines the type of the systemd `service`. In this case, simple means that the service is expected to run in the foreground and does not fork or spawn child processes.
- `ExecStart=/usr/local/bin/prometheus \:` Specifies the exact command that systemd will run to start Prometheus.
- `/usr/local/bin/prometheus:` The path to the Prometheus binary.
- `--config.file /etc/prometheus/prometheus.yml:` The path to Prometheus' main configuration file, prometheus.yml, which defines the scrape targets and other settings.
- `--storage.tsdb.path /var/lib/prometheus/:` The path where Prometheus will store its time-series data (metrics). Prometheus uses a time-series database (TSDB) to store this data.
# d. Start and Enable Prometheus Service
1. Reload systemd to recognize the new service:
```
sudo systemctl daemon-reload
```
2. Start the Prometheus service:
```
sudo systemctl start prometheus
```
3. Enable Prometheus to start on boot:
```
sudo systemctl enable prometheus
```
4. Verify the service status:
```
sudo systemctl status prometheus
```

- Now `connect to prometheus server` using `instancepublicip:9090`

- -----------------------------------------------------------------------------------------------------------------
# Grafana setup:
# Step 3: Install Grafana
# a. Add the Grafana APT Repository
1. Add the `Grafana repository` to your system:
```
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```
2. Add the Grafana GPG key:
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```
3. Update the package list:
```
sudo apt update
```
# b. Install Grafana
1. Install Grafana:
```
sudo apt install grafana -y
```
2. Start and enable Grafana:
```
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```
3. Check Grafana service status:
```
sudo systemctl status grafana-server
```
4. Open a browser and go to `http://<your-VM-IP>:3000` to access the Grafana web interface.
5. Log in using the default credentials:
- Username: `admin`
- Password: `admin` (you will be prompted to change it on first login)

- -------------------------------------------------------------------------------------------------------------------

# Step 4: Configure Grafana to Use Prometheus
- 1. In Grafana, navigate to `Configuration` > `Data Sources`.
- 2. Click `Add data source`, and select Prometheus.
- 3. Set the URL to `http://localhost:9090` (or the address of your Prometheus server).
- 4. Click `Save & Test` to ensure Grafana can access Prometheus.

- --------------------------------------------------------------------------------------------------------------------

# Step 5: Create a Dashboard in Grafana
- 1. Click on `Create` > `Dashboard` in Grafana.
- 2. Add a new panel, and choose `Prometheus` as the data source.
- 3. Use Prometheus queries to visualize metrics on the dashboard.

- ---------------------------------------------------------------------------------------------------------------------

# Now ssh into vm where your `Docker Containers are running`:
# Step 1: Install `Node Exporter on the Docker VM`
- Node Exporter is used to expose hardware and OS-related metrics for Prometheus. To install it:
1. `Log in to the VM where your Docker containers are running.`
2. Download and install Node Exporter:
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz
sudo mv node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin/
```
3. `Create a systemd service file for Node Exporter:`
```
sudo vim /etc/systemd/system/node_exporter.service
```
- Add the following content:
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
```
4. `Start and enable Node Exporter:`
```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```
- You should be able to access the metrics at `http://<Docker_VM_IP>:9100/metrics`.

- --------------------------------------------------------------------------------------------------------------------

# Step 2: Install cAdvisor for Docker Container Metrics
- `cAdvisor (Container Advisor)` is a tool from Google that monitors resource usage and performance characteristics of running containers. It’s useful for tracking Docker container metrics like CPU, memory, and network usage.
- 1. `Run cAdvisor on your Docker VM`:
  You can run cAdvisor as a Docker container:
```
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/cgroup:/cgroup:ro \
  --publish=8080:8080 \
  --detach=true \
  google/cadvisor:latest
```
- This will expose cAdvisor metrics at `http://<Docker_VM_IP>:8080/metrics`

- ---------------------------------------------------------------------------------------------------------------------

# Step 3: Configure Prometheus to Scrape Node Exporter and cAdvisor
- Now that both Node Exporter and cAdvisor are running on your Docker VM, `you need to configure Prometheus (on your Prometheus VM) to scrape their metrics`.
# Edit the Prometheus configuration file on your Prometheus VM
- (/etc/prometheus/prometheus.yml):
```
sudo vim /etc/prometheus/prometheus.yml
```

2. Add the following scrape configurations to the `scrape_configs` section:
```
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<Docker_VM_IP>:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['<Docker_VM_IP>:8080']
```
- Replace `<Docker_VM_IP>` with the `actual IP address of the VM where your Docker containers are running`.

3. Restart Prometheus to apply the configuration:
```
sudo systemctl restart prometheus
```

4. `Verify Prometheus is scraping the new targets`:
Open the Prometheus UI (`http://<Prometheus_VM_IP>:9090/targets`) and ensure that the `node_exporter` and `cadvisor` jobs are listed and up.

- ---------------------------------------------------------------------------------------------------------------------

# Step 4: Visualize Docker Container Metrics in Grafana
- Now that `Prometheus is scraping metrics from your Docker VM`, you can set up `Grafana to visualize those metrics`.
1. Log in to Grafana (`http://<Prometheus_VM_IP>:3000`).
2. Add `Prometheus` as a `Data Source` (if you haven’t already):
- Navigate to `Configuration` > `Data Sources`.
- Click `Add data source`.
- Select `Prometheus`.
- Set the URL to `http://localhost:9090`.
- Click `Save & Test`.

3. Import `Docker` and `Node Exporter Dashboards`:
- In `Grafana`, go to `Create` > `Import`.
- Enter the Dashboard ID of popular prebuilt dashboards:
  `Node Exporter Full: ID: 1860`
  `Docker Monitoring with cAdvisor: ID: 15798, 14282`
- Click `Load` and select Prometheus as the data source.
- Click `Import`.

4. `Explore the Dashboard`: You should now see Docker container metrics (CPU, memory, network usage, etc.) in Grafana.


- ---------------------------------------------------------------------------------------------------------------------

# if you want to `scrape data from 2 different VM's`:
`open Prometheus VM`:
- Edit the `Prometheus configuration file on your Prometheus VM`:
```
sudo vim /etc/prometheus/prometheus.yml
```
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter_vm1'
    static_configs:
      - targets: ['Docker-vm1-publicip:9100']

  - job_name: 'cadvisor_vm1'
    static_configs:
      - targets: ['Docker-vm1-publicip:8080']

  - job_name: 'node_exporter_vm2'
    static_configs:
      - targets: ['Docker-vm2-publicip:9100']

  - job_name: 'cadvisor_vm2'
    static_configs:
      - targets: ['Docker-vm2-publicip:8080']
```
`sudo systemctl restart prometheus`
`sudo systemctl status prometheus`

- --------------------------------------------------------------------------------------------------------------------

# some Docker Containers:
- `nginx container`
```
docker run -d --name nginx-server -p 80:80 nginx
```
- `MySQL Database`
```
docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 mysql
```
- `MongoDB Database`
```
docker run -d --name mongodb -p 27017:27017 mongo
```
- `Redis (In-memory Data Store)`
```
docker run -d --name redis-server -p 6379:6379 redis
```
