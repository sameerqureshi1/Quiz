# Setup Monitoring Stack
This guide explains how to deploy a monitoring stack on Ubuntu using Prometheus, Grafana, Node Exporter, and cAdvisor, then configure Grafana alerts and send notifications through Gmail.

 Step 1: Install Prometheus, Grafana, Node Exporter & cAdvisor 

      # Update system:
     sudo apt update && sudo apt upgrade -y
              
      #Install Prometheus:
      # Create user
      sudo useradd --no-create-home --shell /bin/false prometheus  
      # Create directories
      sudo mkdir /etc/prometheus
      sudo mkdir /var/lib/prometheus
  
      # Download Prometheus
      # If your system have amd architecture 
      wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
      OR
      # If your system have arm architecture (e.g apple M1,M2...)
      wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-arm64.tar.gz

      tar xvf prometheus-*.tar.gz # change * with version number/folder name (prometheus-2.54.1.linux-arm64) you have installed
      
      # Move binaries
      cd prometheus-2.45.0.linux-amd64
      sudo cp prometheus promtool /usr/local/bin/
      
      # Move configuration
      sudo cp -r consoles/ console_libraries/ /etc/prometheus/
      sudo cp prometheus.yml /etc/prometheus/
      sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
      
      #Install Node Exporter
      wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
      tar xvf node_exporter-*.tar.gz
      sudo cp node_exporter-*/node_exporter /usr/local/bin/
      
      #Install cAdvisor (Docker required)
      sudo apt install docker.io -y
      sudo docker run -d \
        --name=cadvisor \
        --volume=/:/rootfs:ro \
        --volume=/var/run:/var/run:ro \
        --volume=/sys:/sys:ro \
        --volume=/var/lib/docker/:/var/lib/docker:ro \
        -p 8080:8080 \
        gcr.io/cadvisor/cadvisor:latest
  
      #Install Grafana
      sudo apt-get install -y apt-transport-https software-properties-common
      sudo mkdir -p /etc/apt/keyrings/
      wget -q -O - https://apt.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
      echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
      sudo apt-get update
      sudo apt-get install grafana -y
      sudo systemctl enable --now grafana-server

 Step 2: Access Applications

- **Prometheus** → [http://localhost:9090](http://localhost:9090)  
- **Node Exporter** → [http://localhost:9100/metrics](http://localhost:9100/metrics)  
- **cAdvisor** → [http://localhost:8080](http://localhost:8080)  
- **Grafana** → [http://localhost:3000](http://localhost:3000) (default login: `admin/admin`)  


 Step 3: Add Prometheus as a Data Source in Grafana

 Open Grafana → Connections → Data sources

 Select Prometheus

 Set URL: http://localhost:9090

Click Save & Test

Step 4: Create Alert Rules

In Grafana go to  Alerting → Alert rules and add 

High CPU usage (>70% for 1 minute) 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100) > 70

<img width="1350" height="587" alt="CPU Storage 1" src="https://github.com/user-attachments/assets/c5af7581-a9e3-458b-ad3d-f302b31dd211" />
<img width="1345" height="592" alt="CPU Storage 2" src="https://github.com/user-attachments/assets/1c3d158f-239a-4929-9a29-183f3225813c" />
<img width="1365" height="590" alt="Cpu notifications" src="https://github.com/user-attachments/assets/9edf84e8-d891-4c1f-a4ed-332e15255d31" />

Low memory (<20%) (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 < 20

sum(container_last_seen{container_label_com_docker_swarm_service_name!=""}) < 1

<img width="1353" height="592" alt="Container not running" src="https://github.com/user-attachments/assets/a6bcd502-f9b5-4d09-b46e-7a454ede4cf9" />

Step 5: Enable Email Notifications
Add Contact Points

In Grafana → Alerting → Contact points

Create a new contact point → choose Email

Add Gmail Address (e.g., your_gmail@gmail.com).
<img width="1349" height="680" alt="Email Contact Point" src="https://github.com/user-attachments/assets/061bf503-6525-43a1-be97-a2df4ea1e0cb" />

Configure Grafana SMTP

Edit /etc/grafana/grafana.ini:

    [smtp]
    enabled = true
    host = smtp.gmail.com:587
    user = your_gmail@gmail.com
    password = your_app_password
    from_address = your_gmail@gmail.com
    from_name = Grafana

⚠️ For Gmail you must generate an App Password
.

Restart Grafana:

    sudo systemctl restart grafana-server

Step 6: Test Alerts

    #Test High CPU
    sudo apt install stress-ng -y
    stress-ng --cpu 4 --timeout 300
    
    #Test Low Memory
    stress-ng --vm 1 --vm-bytes 80% --timeout 300
    
    #Test Container Not Running
    sudo docker stop cadvisor
    # Restart later:
    sudo docker start cadvisor

✅ Final Result

With this setup you will have:

Prometheus, Grafana, Node Exporter & cAdvisor running on Ubuntu

Prometheus integrated as Grafana datasource

Alerts for CPU usage, memory availability, and container state

Email notifications sent to Gmail contacts

<img width="1032" height="517" alt="image" src="https://github.com/user-attachments/assets/fd495448-966b-40dd-80b2-e51dcc8cd56d" />







