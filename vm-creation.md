# Create Containers for Campus Network Design
This campus network design includes Network Automation Server, Network Monitoring, DHCP/DNS Server and Web Server.

## VM-AUTO Configuration
```bash
mkdir vm-auto
cd vm-auto
```
Create `Dockerfile` using `nano Dockerfile` command:

```Dockerfile
FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
        iputils-ping \
        openssh-client \
        nano \
        python3 \
        python3-pip \
        && rm -rf /var/lib/apt/lists/*


RUN pip3 install netmiko ansible

RUN echo "Host 10.99.*.*" >> /etc/ssh/ssh_config && \
    echo "    KexAlgorithms +diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1" >> /etc/ssh/ssh_config && \
    echo "    HostKeyAlgorithms +ssh-rsa" >> /etc/ssh/ssh_config && \
    echo "    Ciphers +aes256-cbc,aes192-cbc,aes128-cbc" >> /etc/ssh/ssh_config

CMD ["tail","-f","/dev/null"]
```
Build the image:
```bash
docker build -t vmauto-server:latest .
```


## VM-ZABBIX Configuration
```bash
mkdir vm-zabbix
cd vm-zabbix
```
Create `Dockerfile`:
```Dockerfile
# Start from the official All-In-One Zabbix Appliance (Ubuntu-based)
FROM zabbix/zabbix-appliance:ubuntu-latest

# The Zabbix image drops privileges for security.
# We must switch back to root to install your custom GNS3 tools.
USER root

# Prevent interactive prompts during installations
ENV DEBIAN_FRONTEND=noninteractive

# Install the basic networking tools required for your GNS3 topology
RUN apt-get update && apt-get install -y \
    iputils-ping \
    openssh-client \
    nano \
    && rm -rf /var/lib/apt/lists/*
```
```bash
docker build -t vmzabbix-server:latest .
```

## VM-DHCP Configuration
```bash
mkdir vm-dhcp
cd vm-dhcp
```
Create `dnsmasq.conf` using `nano dnsmasq.conf`:
```bash
# DNS Forwarding
server=8.8.8.8
server=8.8.4.4

# Listen on the container's interface
interface=eth0

# DHCP Scope for DEIE PCs (VLAN 10)
dhcp-range=set:vlan10,10.10.10.50,10.10.10.200,255.255.255.0,24h
dhcp-option=tag:vlan10,option:router,10.10.10.254

# DHCP Scope for VLAN 20
dhcp-range=set:vlan20,10.10.20.50,10.10.20.200,255.255.255.0,24h
dhcp-option=tag:vlan20,option:router,10.10.20.254

# DHCP Scope for VLAN 30
dhcp-range=set:vlan30,10.10.30.50,10.10.30.200,255.255.255.0,24h
dhcp-option=tag:vlan30,option:router,10.10.30.254

# DHCP Scope for VLAN 40
dhcp-range=set:vlan40,10.10.40.50,10.10.40.200,255.255.255.0,24h
dhcp-option=tag:vlan40,option:router,10.10.40.254

# Set DNS server pushed to clients (VM-DHCP itself)
dhcp-option=option:dns-server,10.10.40.10
```


Create `Dockerfile`:
```Dockerfile
FROM alpine:latest
RUN apk add --no-cache dnsmasq
COPY dnsmasq.conf /etc/dnsmasq.conf
EXPOSE 53/udp 53/tcp 67/udp
# Run dnsmasq in the foreground with debugging output
CMD ["dnsmasq", "-k", "-d"]

```
Build the image:
```bash
docker build -t vmdhcp-server:latest .
```