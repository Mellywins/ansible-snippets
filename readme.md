# Portainer environment

## Steps to manage it
1. Install docker and docker compose to the latest versions.
2. Copy the Docker-compose in your workdir
3. Add Docker to sudoers using these commands:
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```
then logout from the user and log back in so the changes take effects.

3. run `docker-compose up`

### Deployed containers:
* Portainer
* Grafana
* Prometheus
* Node-exporter
* Nginx Reverse Proxy.

### How to monitor other servers?
1. Install node exporter on each target server.
2. Expose the exporter through a loadbalancer/reverse proxy to only certain IP addresses.
3. All the data that's fed into proemtheus will be aggregated inside the grafana dashboard.