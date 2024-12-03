+++
author = "M0D4S"
title = "Securing a Highly Available Web Application with modsecurity WAF"
date = "2023-11-17"
description = "Securing a Highly Available Web Application with modsecurity WAF"
tags = ["docker", "docker-compose","traefik", "waf", "web application firewall", "cloud native", "reverse proxy", "https termination", "load balancing", "security", "modsecurity", "owasp", "owasp modsecurity crs"]
+++

## Architecture of the solution ..
<!-- Import an image of the architecture -->
![Modsecurity with HA architecture](/images/portfolio/modsecurity/arch.png)

<!-- PS: This project is built upon another project check it here -->
<p style="border: 2px solid red; padding: 10px;">
<strong style="color: red;">PS:</strong> This project is built upon another project check it <a href="/post/traefik_lb/" target="_blank">HERE</a>.
</p>

<p>
The image above presents the architecture of the solution. The solution is composed of :
<ul>
<li>A Modsecurity WAF that is used to protect the web application and provide HTTPS termination.</li>
<li>A Traefik Load Balancer that is used to route the traffic to the web application and balance the load between its replicas</li>
<li>A web application that is used as a subject to protect: odoo (ERP)</li>
<li>A database that is used by the web application (PostGreSQL)</li>
</ul>

The WAF is configured to protect the web application from the OWASP Top 10 vulnerabilities. It is configured to block the requests that contain malicious payloads and to block the requests that are not compliant with the OWASP Modsecurity Core Rule Set (CRS). Modsecurity CRS contains by default rules that protect the web application from the OWASP Top 10 vulnerabilities. <br>

## How to implement it ?
<p>
First, we need to have the definitions of the services that we want to deploy in a docker-compose file. Next, we need to have a docker swarm cluster up and running. Then, we need to deploy the services in the cluster using the docker stack deploy command. <br>
Here is the docker-compose file that I used to deploy the services:

```yaml
version: '3.8'
services:
  modsecurity:
    image: owasp/modsecurity-crs:3.3-nginx-alpine-202310170110
    volumes:
      - ./modsecurity.conf:/etc/modsecurity/modsecurity.conf
      - ./default.conf:/etc/nginx/templates/conf.d/default.conf.template
    environment:
      - PROXY_SSL=on 
      - BACKEND=http://odoo
    ports:
      - "80:80"
      - "443:443"
    networks:
      - net1

  db:
    image: postgres
    env_file:
      - ./postgresql/.env
    volumes:
      - "./postgresql/data:/var/lib/postgresql/data"
    networks:
      - net1
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U manager"]
      interval: 5s
      timeout: 5s
      retries: 5

  odoo:
    image: odoo:latest
    depends_on:
      - db
    env_file:
      - ./odoo/.env
    volumes:
      - ./odoo/data_dir:/var/lib/odoo
      - ./odoo/conf:/etc/odoo
    deploy:
      mode: replicated
      replicas: 3
    networks:
      - net1
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.odoo.rule=Host(`odoo.m0d4s.me`)"
      - "traefik.http.services.odoo.loadbalancer.server.port=8069"

  traefik:
    image: traefik:latest
    command:
      - "--api.dashboard=true"
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
    ports:
      # - "80:80"
      - "8080:8080"
      #- "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - net1
    labels:
      - "traefik.http.routers.traefik.rule=Host(`traefik.m0d4s.me`)"
      - "traefik.http.routers.traefik.service=api@internal"
networks:
  net1:
```

**Make sure to:** <br>
<ul>
<li>Create the filesystem architecture as needed by the docker-compose file</li>
<li>Change the domain names in the docker-compose file to your own domain names OR add the current domain names to your /etc/hosts file</li>
</ul>

**The content of the env files:**
```bash
# postgresql/.env
POSTGRES_USER=manager
POSTGRES_PASSWORD=password123
POSTGRES_DB=postgres
```

```bash 
# odoo/.env
POSTGRES_USER=manager
POSTGRES_PASSWORD=password123
POSTGRES_DB=postgres
```

```bash	
# default.conf
# Nginx configuration for both HTTP and SSL

server_tokens off;

map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    listen 80 default_server;

    server_name localhost;
    set $always_redirect on;

    location / {
        client_max_body_size 20m;

        if ($always_redirect = on) {
            proxy_pass http://traefik:80;
        }

        include includes/proxy_backend.conf;
    }

    include includes/location_common.conf;
}

server {
    listen 443 ssl;

    server_name localhost;
    set $upstream http://localhost:80;

    ssl_certificate /etc/nginx/conf/server.crt;
    ssl_certificate_key /etc/nginx/conf/server.key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;
    ssl_session_tickets off;

    ssl_dhparam /etc/ssl/certs/dhparam-2048.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    ssl_stapling off;
    ssl_stapling_verify off;

    ssl_verify_client off;

    location / {
        client_max_body_size 0;

        include includes/proxy_backend.conf;
    }
}
```

```bash
# modsecurity.conf
# Basic ModSecurity Configuration

SecRuleEngine On
SecRequestBodyAccess On
SecResponseBodyAccess On

# Enable/Disable Rule Sets
IncludeOptional /etc/modsecurity/owasp-crs/*.conf
IncludeOptional /etc/modsecurity/owasp-crs/base_rules/*.conf

# Threat Intelligence Integration (Example using the modsecurity_crs_10_config.conf file)
IncludeOptional /etc/modsecurity/owasp-crs/activated_rules/modsecurity_crs_10_config.conf

# Rate Limiting for Preventing DDoS Attacks
SecRuleEngine On
SecRequestBodyLimit 13107200
SecRequestBodyInMemoryLimit 13107200

# Behavioral Analysis for Web Application Anomaly Detection
SecRuleEngine DetectionOnly
SecRequestBodyNoFilesLimit 131072

# Customize other ModSecurity settings as needed
```

## How to deploy it ?
<p>
To deploy the solution, we need to follow these steps:
<ol>
<li>Install docker and docker-compose on your machine</li>
<li>Create the files and directories as needed by the docker-compose file: docker-compose.yml, modsecurity.conf, default.conf, odoo/.env, postgresql/.env</li>
<li>Change the domain names in the docker-compose file to your own domain names OR add the current domain names to your /etc/hosts file</li>
<li>Change the permissions of the directory odoo to 777</li>
<li>Run the following command to deploy the solution:
<pre><code>docker-compose up -d</code></pre>
<li>Open the browser and go to the domain name of the odoo service. In my case, it is: <a href="https://odoo.m0d4s.me" target="_blank">https://odoo.m0d4s.me</a>, You will be prompted to create a database and to create a user. Once you do that, you will be redirected to the login page. You can login with the user that you created.</li>
</ol>
</p>

## How to destroy it ?
<p>
To destroy the solution, we need to run the following command:
<pre><code>docker stack rm modsecurity</code></pre>
</p>

## How to test it ?
<p>
To test the solution, we need to open a browser and go to the domain name of the odoo service. In my case, it is: <a href="https://odoo.m0d4s.me" target="_blank">https://odoo.m0d4s.me</a>. <br>
We can also test the solution by trying basic attacks on the web application and see if the WAF is blocking them or not. <br>
Here are some examples of attacks that we can try:
<ul>
<li>SQL Injection: <a href="https://odoo.m0d4s.me/web/login?login=admin%27%20or%201%3D1--" target="_blank">https://odoo.m0d4s.me/web/login?login=admin%27%20or%201%3D1--</a></li>

<li>XSS: <a href="https://odoo.m0d4s.me/web/login?login=%3Cscript%3Ealert(%22XSS%22)%3C/script%3E" target="_blank">https://odoo.m0d4s.me/web/login?login=%3Cscript%3Ealert(%22XSS%22)%3C/script%3E</a></li>

<li>Command Injection: <a href="https://odoo.m0d4s.me/web/login?login=admin%27%3B%20cat%20/etc/passwd%20%23" target="_blank">https://odoo.m0d4s.me/web/login?login=admin%27%3B%20cat%20/etc/passwd%20%23</a></li>
</ul>
</p>

## Conclusion
<p>
In this article, we saw how to secure a highly available web application with a WAF. We saw how to deploy the solution and how to test it. <br>
I hope you enjoyed this article and found it useful. <br>
Thank you for reading. <br>
