# Portainer environment

## Steps to manage it
1. Install docker and docker compose to the latest versions.
2. Copy the Docker-compose in your workdir.
3. Add Docker to sudoers using these commands:
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```
then logout from the user and log back in so the changes take effects.

3. run `docker-compose up`

## Deployed containers:
* Portainer
* Grafana
* Prometheus
* Node-exporter (Exports data about the host system: CPU, RAM, DISK, IO consumptions....)
* Nginx Reverse Proxy.


## How to make this central monitoring open for extension?
It's relatively easy to integrate new servers to this monitoring tool.
It can be reduced to these 3 steps
1. Install a data exporter on the server. Depending on your needs, you can cheese from a multiple of OSS exporters( node-exporter, cAdvisor...)
Here is an example compose file for a node-exporter that you can easily run on your server
```YAML
services:
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'
    networks:
      - portainer-network

networks:
    portainer-network
```
2. Make that exporter reachable. If you are using a reverse proxy to handle inbound traffic, Make a rewrite rule that points to the exporter container. 
> Note that it's a security best practice to not expose this exporter's port to the public, but make it reachable from a reverse proxy. Because it can output some exploitable information about your system spec.
3. Change prometheus' config file [prometheus.yml](./config/prometheus/prometheus.yml). Under the scrape_configs, add a new scrape job by assigning it a name, then the target url in the form of `host:port` or omit the port if you're routing to the exporter with a reverse proxy.
Here is an example
```YAML
  - job_name: 'Scrape from Node Exporter'
    static_configs:
      - targets: ['node_exporter:9090']
```
