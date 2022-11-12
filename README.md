# My Simple Adventure with Prometheus, node_exporter, Grafana...

The purpose of this experiment was to play and have fun running Grafana and Prom in Docker on my localhost and Manually firing up node_exporter from localhost. Primarily, I wanted something to do on my new HPDEVONE. :D

## Commands to Install Docker

1. `sudo apt-get update`
1. `sudo apt-get upgrade`
1. `sudo apt-get install ca-certificates curl gnupg lsb-release`
1. `sudo mkdir -p /etc/apt/keyrings`
1. `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg`
1. `echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`
1. `sudo apt-get update` - sure why not?
1. `sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin`
1. `sudo service docker start`
1. `sudo docker run hello-world`
1. `sudo docker ps -a`
1. `sudo docker images`

## Commands to run node_exporter

1. `wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz`
1. `tar xvfz node_exporter-1.4.0.linux-amd64.tar.gz`
1. `cd node_exporter-1.4.0.linux-amd64/`
1. `./node_exporter`

## Prom Config File to Scrape Prom Itself and Node Exporter

yaml
```
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'

# scrape prometheus and scrape node_exporter
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node'

    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9100']
```

## Commands to run Prom in Docker Container and Restart on Reboot

1. `sudo docker build -t prometheus_me .`
1. `sudo docker run -d --restart always --config.file=/home/coreyperkins/src/GitHub/promgrafexample/prometheus.yml --network host prometheus_me`

## Command to run Grafana in Docker Container and Restart on Reboot

1. `sudo docker run -d --name=grafana --restart always --network host grafana/grafana`

## Configuration of service to node_exporter at startup

1. `sudo mv ./node_exporter /usr/local/bin/` --copy from your downloaded location
1. `sudo vim /etc/systemd/system/node_exporter.service`
1. Paste the following into vim then save and close

config
```
[Unit]
Description=Node Exporter
After=network.target
 
[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/bin/node_exporter
 
[Install]
WantedBy=multi-user.target
```

1. `sudo systemctl daemon-reload`
1. `sudo systemctl start node_exporter`
1. `sudo systemctl status node_exporter`
1. `sudo systemctl enable node_exporter`
1. `curl http://localhost:9100/metrics`

## Miscellaneous

1. Navigate to [Prom](http://localhost:9090)
1. Navigate to [Grafana](http://localhost:3000)

## Resources

1. [Install Prom /w Docker](https://www.techgeeknext.com/tools/docker/install-prometheus-using-docker)
1. [Install Grafana /w Docker](https://www.techgeeknext.com/tools/docker/install-grafana-using-docker)
1. [Prom Configuration](https://prometheus.io/docs/prometheus/latest/getting_started/#configuring-prometheus-to-monitor-itself)
1. [node_exporter for Linux Node to Prom](https://prometheus.io/docs/guides/node-exporter/)
1. [stress-ng](https://www.linuxshelltips.com/create-cpu-load-linux/)
1. [node_exporter Grafana Dashboard](https://grafana.com/grafana/dashboards/5174-node-exporter-full/) -- free dashboard for node_exporter for Grafana, I downloaded the json and imported into prom

