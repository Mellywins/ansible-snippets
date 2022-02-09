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
1. Install a data exporter on the server. Depending on your needs, you can choose from a multiple of OSS exporters( node-exporter, cAdvisor...)
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
## Improvements Roadmap

#### Exporters
One way we can improve this is by automating the integration of exporters to the target servers. This can be achieved by using Ansible. With whom we define playbooks to install and run the exporters' containers.

To approach this with best-practices compliance, we could make our central monitoring server the **control node**. This means that it will be holding the playbooks and will have ansible installed. It'll also be holding the responsibilities of connecting to the target servers and execute the playbooks inside them.

One question might arise when thinking about this approach. *Not every server is running docker, some are just old plain Apache servers for example, so how do we solve this challenge?*

This where ansible strong-points kick in, we can delegate the responsibility if installing docker if it does not exist on the target server to **Ansible**. We will have another ansible playbook that will handle checking if docker does not exist and install it if so.

#### Proxies
It's considered a *best-practice* to run every software on a Zero-Trust Network. This means that no implicit assumptions about the network's safety just because it's a private network. Data in transit is as important as data in rest.

To solve this, exporters should not be exposed publically with a port (Here we consider the Private Network "public". remember? ZERO-TRUST). But sit behind a proxy and delegate securing the communication channels with SSL/TLS to the proxy.


However, one issue still persists. Can we automate the configuration of the running proxy on the target server? Check if it's even running a proxy? Is it apache or nginx?

This is a challenging problem and difficult to automate because there is no pre-defined standard procedure of employing proxies, every server runs its own flavor of proxy(Apache/Nginx/traefik...). So we'll deal with this point later down the road.

## Selling points
* Intervention after errors occur is risky and can have bad implications. Monitoring services allows entreprises to foresee problems and issues and give them a grace period to regulate and roll out patches/fixes. Forecasting problems is also a great way to gain clients' trust
* Monitoring is crucial to settings Piterion's performance benchmark
* Helps setting cost expectations of the running services. No more under/over-estimating server resources. You'll also know exactly what and when you'll be needing to scale a service. 
* Reduce menial tasks, thus save time and money and invest Piterion's human resources in more demanding issues.
* Configurable alert system that will let us know if a server has reached a dangerous Memory threshhold for example, giving us time to extend it before disasters occur. This is applicable to every other resource as well.
* Piterion will not only offer an easy way for entreprises to deploy a central monitoring service, but also an easy way to integrate other nodes into this services 