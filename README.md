# Setup Monitoring System
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
