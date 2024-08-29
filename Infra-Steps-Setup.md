
### Steps for setting up the application, Downloading, Extracting, and Starting Prometheus, Node Exporter, Blackbox Exporter, Alertmanager, google/cadvisor, and Grafana. 

#### Prerequisites
- Ensure you have `wget` and `tar` installed on both VMs.
- Ensure you have appropriate permissions to download, extract, and run these binaries.
- Replace `<version>` with the appropriate version number you wish to download.
- **NOTE: Ensure you check the vagrantfiles for both VMs and the tools already installed by the vagrantfiles so you don't repeat the installation. ( If you already ran the vagrantfiles to setup the environments and install tools for both VMs, kindly skip already performed tasks in the vagrantfiles).**


#### **VM-2 (Application, Node Exporter, google/cadvisor)**

##### Application
1. **Clone Application Repository**
   ```bash
   git clone https://github.com/Godfrey22152/quiz_game_app.git 
   cd quiz_game_app/
   ```
2. **Build Image and Run Container**
Build your own image from the Dockerfile, run the image but ensure you provide the required details for the <.env> file:
   ```bash
   docker build -t quiz-app .
   ```
Or use already built image from Dockerhub, you can run the image as shown below:
   ```bash
   docker run -d --name=quiz-app -p 5000:5000 godfrey22152/quiz-game-app:2.0
   ```
3. **Access running Application**
Access running Application on the browser on `http://localhost:5000/` or `http://<YOUR-VM-IP>:5000/`.

##### **Node Exporter**
 - **Note: Steps 1-3 already done using the VM_2_Vagrantfile.**

1. **Download Node Exporter**
   ```bash
   wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
   ```

2. **Extract Node Exporter**
   ```bash
   tar xvfz node_exporter-1.8.1.linux-amd64.tar.gz
   ```

3. **Move the binary to a directory in your PATH**
   ```bash
   sudo mv node_exporter/node_exporter /usr/local/bin/
   ```
   
4. **Create a systemd service file for Node Exporter**
   ```bash
   sudo nano /etc/systemd/system/node_exporter.service
   ```

   **Add the following content to the systemd service file `/etc/systemd/system/node_exporter.service`:**
   ```bash
      [Unit]
      Description=Node Exporter
      Wants=network-online.target
      After=network-online.target

      [Service]
      User=node_exporter
      ExecStart=/usr/local/bin/node_exporter

      [Install]
      WantedBy=default.target
   ```

5. **Create a user for Node Exporter**
   ```bash
   sudo useradd -rs /bin/false node_exporter
   ```

6. **Reload systemd and start Node Exporter**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start node_exporter
   sudo systemctl enable node_exporter
   ```
   
7. **Access Node Exporter**
Access running node exporter on the browser on `http://localhost:9100/` or `http://<YOUR-VM-IP>:9100/`.


##### **Google/cadvisor**
1. **Running cAdvisor in a Docker Container**
cAdvisor (Container Advisor) provides container users an understanding of the resource usage and performance characteristics of their running containers. It is a running daemon that collects, aggregates, processes, and exports information about running containers. Specifically, for each container it keeps resource isolation parameters, historical resource usage, histograms of complete historical resource usage and network statistics. This data is exported by container and machine-wide. You can run a single cAdvisor to monitor the whole machine. Simply run:

```bash
# use the latest release version from https://github.com/google/cadvisor/releases
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:latest 
```

2. **Access Running cAdvisor on your browser**
cAdvisor is now running (in the background) on `http://localhost:8080` or `http://<Your-VM-IP>:8080`. 


#### **VM-1 (Prometheus, Alertmanager, Blackbox Exporter, Grafana)**

##### **Prometheus**     
1. **Download Prometheus**
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.53.1/prometheus-2.53.1.linux-amd64.tar.gz
   ```
      
2. **Extract Prometheus**
   ```bash
   tar xvfz prometheus-2.53.1.linux-amd64.tar.gz
   ```
   
3. **Remove the downloaded tarball to clean up disk space**
   ```bash
   sudo rm prometheus-2.53.1.linux-amd64.tar.gz
   ```
   
4. **Rename the extracted directory to 'prometheus' for easier access**
   ```bash
   sudo mv prometheus-2.53.1.linux-amd64/ prometheus
   ```

5. **Create a directory for logging active queries and grant the Prometheus process permissions to write to this 
   directory.**
   ```bash
   sudo mkdir -p /home/vagrant/prometheus/data
   sudo chown -R vagrant:vagrant /home/vagrant/prometheus/data
   ```

6. **Start Prometheus**
   ```bash
   cd prometheus
   ./prometheus --config.file=prometheus.yml &
   ```
      
##### **Alertmanager**
1. **Download Alertmanager**
   ```bash
   wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
   ```

2. **Extract Alertmanager**
   ```bash
   tar xvfz alertmanager-0.27.0.linux-amd64.tar.gz
   ```

3. **Remove the downloaded tarball to clean up disk space**
   ```bash
   sudo rm alertmanager-0.27.0.linux-amd64.tar.gz
   ```
   
4. **Rename the extracted directory to 'alertmanager' for easier access**
   ```bash
   sudo mv alertmanager-0.27.0.linux-amd64/ alertmanager
   ```
    
5. **Create data directory for alertmanager and grant it permissions to write to it.**
   ```bash
   sudo mkdir -p alertmanager/data
   sudo chown -R vagrant:vagrant alertmanager/data
   ```   

6. **Start Alertmanager**
   ```bash
   cd alertmanager
   ./alertmanager --config.file=alertmanager.yml &
   ```

##### **Blackbox Exporter**
1. **Download Blackbox Exporter**
   ```bash
   wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux- 
   amd64.tar.gz
   ```

2. **Extract Blackbox Exporter**
   ```bash
   tar xvfz blackbox_exporter-0.25.0.linux-amd64.tar.gz
   ```

3. **Remove the downloaded tarball to clean up disk space**
   ```bash 
   sudo rm blackbox_exporter-0.25.0.linux-amd64.tar.gz
   ```

4. **Rename the extracted directory to 'blackbox_exporter' for easier access**
   ```bash
   sudo mv blackbox_exporter-0.25.0.linux-amd64/ blackbox_exporter
   ```
   
5. **Start Blackbox Exporter**
   ```bash
   cd blackbox_exporter
   ./blackbox_exporter &
   ```

##### **Grafana**
1. **Download Grafana**
   ```bash
   wget https://dl.grafana.com/oss/release/grafana-11.1.3.linux-amd64.tar.gz
   ```

2. **Extract Grafana**
   ```bash 
   tar -zxvf grafana-11.1.3.linux-amd64.tar.gz
   ```

3. **Remove the downloaded tarball to clean up disk space**
   ```bash
   sudo rm grafana-11.1.3.linux-amd64.tar.gz
   ```
   
4. **Start Grafana**
   ```bash
   cd grafana-v11.1.3/
   ./bin/grafana-server &
   ```
5. **Access Grafana in a Browser**
Once Grafana is running, you can access the web UI by opening a web browser and navigating to:
   ```bash
   http://localhost:3000
   ```
If you're running Grafana on a remote server or a Vagrant machine, replace localhost with the server's IP address. Log in to Grafana. The default username is `admin`, and the default password is also `admin`.

### Notes:
- The `&` at the end of each command ensures the process runs in the background.
- Ensure that you have configured the `prometheus.yml` and `alertmanager.yml` configuration files correctly before starting the services.
- You can use this command `curl -X POST http://<Prometheus_VM_IP>:9090/-/reload`, and `curl -X POST http://<ALERTMANAGER-VM-IP>:9093/-/reload` to trigger Prometheus and alertmanager to reload its configuration files without restarting the service.
- Adjust the firewall and security settings to allow the necessary ports (typically 9090 for Prometheus, 9093 for Alertmanager, 9115 for Blackbox Exporter, 9100 for Node Exporter, and 3000 for grafana) to be accessible.

---

# Prometheus and Alertmanager Configuration

## Prometheus Configuration (`prometheus.yml`)

### Global Configuration
```yaml
global:
  scrape_interval: 15s                # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s            # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
```

### Alertmanager Configuration
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - '192.168.71.98:9093'          # Alertmanager endpoint
```

### Rule Files
```yaml
rule_files:
   - "alert_rules.yml"                # Path to alert rules file
  # - "second_rules.yml"              # Additional rule files can be added here
```

### Scrape Configuration
#### Prometheus Itself
```yaml
scrape_configs:
  - job_name: "prometheus"            # Job name for Prometheus

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]   # Target to scrape (Prometheus itself)
```

#### Node Exporter
```yaml
  - job_name: "node_exporter"         # Job name for node exporter

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["192.168.119.133:9100"]  # Target node exporter endpoint
```

#### cAdvisor
```yaml
  - job_name: 'cadvisor'
      static_configs:
        - targets:
            - 192.168.119.133:8080 
```

#### Blackbox Exporter
```yaml
  - job_name: 'blackbox'              # Job name for blackbox exporter
    metrics_path: /probe              # Path for blackbox probe
    params:
      module: [http_2xx]              # Module to look for HTTP 200 response
    static_configs:
      - targets:
        - http://prometheus.io        # HTTP target
        - https://prometheus.io       # HTTPS target
        - http://192.168.119.133:5000/  # HTTP target with port 5000
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # Blackbox exporter address
```

## Alert Rules Configuration (`alert_rules.yml`)

### Alert Rules Group
```yaml
groups:
- name: alert_rules                   # Name of the alert rules group
  rules:
    - alert: InstanceDown
      expr: up == 0                   # Expression to detect instance down
      for: 1m
      labels:
        severity: "critical"
      annotations:
        summary: "Endpoint {{ $labels.instance }} down"
        description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."

    - alert: WebsiteDown
      expr: probe_success == 0        # Expression to detect website down
      for: 1m
      labels:
        severity: critical
      annotations:
        description: The website at {{ $labels.instance }} is down.
        summary: Website down

    - alert: HostOutOfMemory
      expr: node_memory_MemAvailable / node_memory_MemTotal * 100 < 25  # Expression to detect low memory
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Host out of memory (instance {{ $labels.instance }})"
        description: "Node memory is filling up (< 25% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HostOutOfDiskSpace
      expr: (node_filesystem_avail{mountpoint="/"} * 100) / node_filesystem_size{mountpoint="/"} < 50  # Expression to detect low disk space
      for: 1s
      labels:
        severity: warning
      annotations:
        summary: "Host out of disk space (instance {{ $labels.instance }})"
        description: "Disk is almost full (< 50% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HostHighCpuLoad
      expr: (sum by (instance) (irate(node_cpu{job="node_exporter_metrics",mode="idle"}[5m]))) > 80  # Expression to detect high CPU load
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Host high CPU load (instance {{ $labels.instance }})"
        description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: ServiceUnavailable
      expr: up{job="node_exporter"} == 0  # Expression to detect service unavailability
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Service Unavailable (instance {{ $labels.instance }})"
        description: "The service {{ $labels.job }} is not available\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HighMemoryUsage
      expr: (node_memory_Active / node_memory_MemTotal) * 100 > 90  # Expression to detect high memory usage
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: "High Memory Usage (instance {{ $labels.instance }})"
        description: "Memory usage is > 90%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: FileSystemFull
      expr: (node_filesystem_avail / node_filesystem_size) * 100 < 10  # Expression to detect file system almost full
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "File System Almost Full (instance {{ $labels.instance }})"
        description: "File system has < 10% free space\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

## Alertmanager Configuration (`alertmanager.yml`)

### Routing Configuration
```yaml
route:
  group_by: ['alertname']             # Group by alert name
  group_wait: 30s                     # Wait time before sending the first notification
  group_interval: 5m                  # Interval between notifications
  repeat_interval: 1h                 # Interval to resend notifications
  receiver: 'email-notifications'     # Default receiver

receivers:
- name: 'email-notifications'             # Receiver name
  email_configs:
  - to: your_email@gmail.com              # Email recipient
    from: monitoringProject@gmail.com     # Email sender
    smarthost: smtp.gmail.com:587         # SMTP server
    auth_username: your_email             # SMTP auth username    
    auth_identity: your_email             # SMTP auth identity    
    auth_password: "wxgd dzcs ehkb ivkc"  # SMTP auth password
    send_resolved: true                   # Send notifications for resolved alerts
```

### Inhibition Rules
```yaml
inhibit_rules:
  - source_match:
      severity: 'critical'            # Source alert severity
    target_match:
      severity: 'warning'             # Target alert severity
    equal: ['alertname', 'dev', 'instance']  # Fields to match
```

## Steps to Add Prometheus Data Source and Import Dashboard
### Step 1: Access Grafana
- Open your web browser and go to the Grafana web interface. The default URL is `http://localhost:3000`.
- Log in to Grafana. The default username is `admin`, and the default password is also `admin`.

### Step 2: Add Prometheus as a Data Source
- Once logged in, go to the Configuration (gear icon) on the left-hand side.
- Click on Data Sources.
- Click the Add data source button.
- In the list of available data sources, select Prometheus.

### Step 3: Configure Prometheus Data Source
- Name: Enter a name for the data source (e.g., "Prometheus").
- URL: Enter the URL of your Prometheus server. If Prometheus is running locally and on the default port, this will be `http://localhost:9090`.
- Access: Choose how Grafana will access the Prometheus server:
- Server (default): Grafana will connect directly to the Prometheus server.
- Browser: The connection will be made from the user’s browser.
- Scrape Interval: This is optional. You can specify how frequently Grafana should query Prometheus.
- HTTP Method: Typically, you can leave this as GET.
- Custom HTTP Headers: If you need to add any custom headers for the connection, you can configure them here.
- With Credentials: Enable this if your Prometheus server requires authentication.
- Basic Auth Details: If your Prometheus server is secured with basic authentication, enable Basic Auth and provide the username and password.

### Step 4: Test and Save
- Scroll down to the bottom of the configuration page.
- Click the Save & Test button. Grafana will test the connection to the Prometheus server.
- If successful, you’ll see a confirmation message. If not, check the error message and adjust your configuration.

### Step 5: Use Prometheus in Dashboards 
To import a Prometheus dashboard into Grafana, you typically use a pre-built dashboard from Grafana's dashboard repository. Each dashboard has a unique ID (or UID) that you can use to import it. 

- **Import Node Exporter Full dashboard:**
  **To import this dashboard:**
  1. In Grafana, go to the Dashboards section and click Import.
  2. In the "Import via grafana.com" field, enter 1860 and click Load.
  3. After loading, select the Prometheus data source you configured earlier.
  4. Click Import to add the dashboard to your Grafana instance.
This will give you a comprehensive set of metrics related to system performance, gathered by Prometheus from Node Exporter.

- **Import Blackbox Exporter dashboard:**
  **To import this dashboard:**
  1. In Grafana, navigate to Dashboards and click Import.
  2. Enter 7587 in the "Import via grafana.com" field and click Load.
  3. Choose your Prometheus data source.
  4. Click Import to add the dashboard to your Grafana instance.
This dashboard will help you visualize the metrics collected by the Blackbox Exporter, which is used for probing endpoints over various protocols like HTTP, HTTPS, DNS, TCP, and ICMP.

- **Import Docker and system monitoring dashboard:**

  For monitoring containers with cAdvisor, you can use the "Docker and system monitoring" dashboard, which provides insights into the containerized applications. The UID for this popular dashboard is 893.
  
  **To import it into Grafana:**
  1. In Grafana, go to Dashboards and click Import.
  2. Enter 893 in the "Import via grafana.com" field and click Load.
  3. Select your Prometheus data source. 
  4. Click Import to add the dashboard to your Grafana instance.
This dashboard provides detailed metrics on your containers, such as CPU, memory, network, and disk usage, as well as information about the host system.

---

This documentation provides a detailed explanation of the Prometheus and Alertmanager configuration files. It covers global settings, alert rules, email notifications, and inhibition rules.
