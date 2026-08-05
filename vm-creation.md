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

# ---------------------------------------------------
# Local DNS Records for DIS Server Farm
# ---------------------------------------------------
address=/www.uorfoe.com/10.10.40.20
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

## VM-WEB Configuration

```bash
mkdir vm-web
cd vm-web
```
Create `index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DIS Web</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f4f4f9; color: #333; text-align: center; padding: 50px; }
        .container { background: white; padding: 30px; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); max-width: 600px; margin: auto; }
        h1 { color: #0056b3; }
        .success { color: #28a745; font-weight: bold; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Department of Interdisciplinary Studies</h1>
        <h2>Internal Web Server (VM-WEB)</h2>
        <p>Welcome to Faculty of Engineering, University of Ruhuna. This is our official web site.</p>
        <p class="success">✔ Department of Electrical and Information Engineering</p>
        <p class="success">✔ Department of Civil and Environmental Engineering</p>
        <p class="success">✔ Department of Mechanical and Manufacturing Engineering</p>
        <hr>
        <p><em>University of Ruhuna - Design and Management of Data Networks Project</em></p>
    </div>
</body>
</html>
```

Create `Dockerfile`:
```bash
# Use the ultra-lightweight Nginx Alpine image
FROM nginx:alpine

# Remove the default Nginx welcome page
RUN rm -rf /usr/share/nginx/html/*

# Copy your custom project webpage into the container
COPY index.html /usr/share/nginx/html/

# Expose port 80 for HTTP traffic
EXPOSE 80

# Start Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]
```