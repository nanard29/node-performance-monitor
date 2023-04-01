# Node Performance Dashboard
User Guide  to monitor Node performances (hardware performance , bandwidth used, etc) using Grafana Dashboard.

![image](https://user-images.githubusercontent.com/97830502/229294098-c2136f26-e370-4322-a0cb-27f05b543571.png)


Grafana Official documentation is located here: https://grafana.com/docs/grafana-cloud/quickstart/noagent_linuxnode/


# Grafana

Get the latest node_exporter package
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
```
Extract
```
tar xvfz node_exporter-1.5.0.linux-amd64.tar.gz
```
Move into folder
```
cd node_exporter-1.5.0.linux-amd64/
```
Add executable permissions
```
chmod +x node_exporter
```
Move it in /usr/local/bin, rename it grafana-agent, then start it
```
mv node_exporter /usr/local/bin/grafana-agent
grafana-agent
```
Check in an another terminal if everything if working properly
```
curl http://localhost:9100/metrics
```
If you have results, you can close this terminal, and Ctrl-C the other one to stop it. Then we will create a service (in case there is an error, everything will restart correctly)

Create an user for Grafana
```
sudo useradd --no-create-home --shell /bin/false grafana-agent
```
Create service
```
sudo tee <<EOF >/dev/null /etc/systemd/system/grafana-agent.service
[Unit]
Description=Grafana Agent

[Service]
User=grafana-agent
ExecStart=/usr/local/bin/grafana-agent
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
To enable the service to run automatically on every reboot, and start it:
```
sudo systemctl enable grafana-agent.service
sudo systemctl start grafana-agent.service
```
Check  if everything if working properly
```
curl http://localhost:9100/metrics
```
# Configure a dashboard

Import a pre-build dashboard
We choose Linux Hosts Metrics | Base. Note the ID of the dashboard: 10180. We will use this ID in the next step.

In Grafana, click Dashboards in the left-side menu to go to the Dashboards page.

Click New and select Import in the dropdown. Enter the ID number of the dashboard we selected into the box and click Load.

![image](https://user-images.githubusercontent.com/97830502/229294001-1862bedb-1443-4653-b8b2-561dfde346b5.png)

![image](https://user-images.githubusercontent.com/97830502/229294045-f3aa9544-0be7-4cf6-adc2-b0b1166de947.png)


# Install Prometheus on the node

Download Prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
```
Extract it
```
tar xvf prometheus-2.43.0.linux-amd64.tar.gz
```
Move into folder
```
cd prometheus-2.43.0.linux-amd64
```
To confirm your username and URL, first navigate to the Cloud Portal, then from the Prometheus box, click Send Metrics.

Create a Prometheus configuration file named prometheus.yml in the same directory as the Prometheus binary with the following content. Do not forget to change the login and password in the instruction below.
```
global:
  scrape_interval: 60s

scrape_configs:
  - job_name: node
    static_configs:
      - targets: ['localhost:9100']
			

remote_write:
  - url: '<Your Metrics instance remote_write endpoint>'
    basic_auth:
      username: 'your grafana username'
      password: 'your Grafana API key'
```
![image](https://user-images.githubusercontent.com/97830502/229293856-839d7eb5-4f22-4d66-8c7b-302a94c10581.png)


You can find the /api/prom/push URL, username, and password for your metrics endpoint by clicking on Details in the Prometheus card of the Cloud Portal.

Save the file.

Run the Prometheus binary, instructing Prometheus to use the configuration file we just created.
```
./prometheus --config.file=./prometheus.yml
```


# Results Node Performances Metrics
Check  the Grafana dashboard

![image](https://user-images.githubusercontent.com/97830502/229293437-f9bfa002-864b-4dd6-a494-b60e895c1c85.png)

