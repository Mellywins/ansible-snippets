# File structure
```
├── cAdvisor.yml // Containers data exporter playbook
├── config
│   └── prometheus
│       └── prometheus.yml // prometheus config file that will hold scrape information
├── docker-install.yml // Making sure that Docker exists on the host
├── grafana-prometheus-install.yml // Dashboard+DB installation playbook
├── inventory // Holds info about the server and ssh connection
├── node-exporter.yml // System data exporter playbook
├── readme.md
├── update-prometheus-config.yml // reload prometheus config when
└── vars
    └── default.yml 
```
# Deployment
This guide will tell you how to deploy this tool to a server with ease, using Ansible.

Follow these steps:

0. clone this repository and `cd monitoring-containers`. Make sure you have ansible installed. You can find the steps in this documentation [link](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu).

1. Add the server informations in the inventory file. It should look something like this: 
    ```
    [central_monitoring_servers]
    127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file=path_to_key
    ```
    > `ansible_port` is the server's ssh port

    if you use a password, replace `ansible_ssh_private_key_file` with `ansible_password` and write the password.

2. Under `vars/default.yml` put the path to the prometheus config file `prometheus.yml`. You can find it in this repo under `monitoring-containers/config`. Feel free to edit this as you see fit.

3.  Check that ansible can be connected to the server by running the following:
    ```
    ansible -i inventory -m ping central_monitoring_servers
    ```
4.  Run the playbook `docker-install.yml` (implicit assumption that the server is ubuntu) to install docker if it doesn't exist with this command:
    ```
    ansible-playbook -i inventory docker-install.yml
    ```
5. Run the playbook `grafana-prometheus-install.yml` to install the grafana dashboard and prometheus database on the server using this command:
    ```
    ansible-playbook -i inventory grafana-prometheus-install.yml

    ```
6. Now we can start deploying exporters to the servers we want to watch. We can start by monitoring the same server we deployed the tools on by running:
    ```
    ansible-playbook -i inventory node-exporter.yml
    ```
7. If you want to visualize data about running containers on a Node. You can install cAdvisor and add it to the prometheus config. Run:
    ```
        ansible-playbook -i inventory cAdvisor.yml
    ```
    * Add the config in `prometheus.yml`  then reload prometheus by running the update-prometheus-config.yml playbook with this command:
        ```
        ansible-playbook -i inventory update-prometheus-config.yml
        ```