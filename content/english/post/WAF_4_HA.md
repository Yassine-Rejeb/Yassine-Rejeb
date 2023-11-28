+++
author = "M0D4S"
title = "Securing a Highly Available Web Application with WAF"
date = "2023-11-17"
description = "Securing a Highly Available Web Application with WAF"
tags = ["docker", "docker swarm", "traefik", "waf", "web application firewall", "cloud native", "reverse proxy", "https termination", "load balancing", "security", "modsecurity", "owasp", "owasp modsecurity crs"]
+++

## Architecture of the solution ..
<!-- Import an image of the architecture -->
![Modsecurity with HA architecture](/images/portfolio/modsecurity/arch.png)

<!-- PS: This project is built upon another project check it here -->
<p style="border: 2px solid red; padding: 10px;">
<strong style="color: red;">PS:</strong> This project is built upon another project check it <a href="/post/traefik_lb/" target="_blank">HERE</a>.
</p>

<p>
The image above presents the architecure of the solution. The solution is composed of :
<ul>
<li>A Traefik Load Balancer that is used to route the traffic to the web application and balance the load between its replicas</li>
<li>A Modsecurity WAF that is used to protect the web application</li>
<li>A web application that is used as a subject to protect: odoo (ERP)</li>
<li>A database that is used by the web application (PostGreSQL)</li>
</ul>

The WAF is configured to protect the web application from the OWASP Top 10 vulnerabilities. The WAF is configured to block the requests that contain malicious payloads and to block the requests that are not compliant with the OWASP Modsecurity Core Rule Set (CRS).

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
      - "traefik.http.routers.odoo-https.rule=Host(`odoo-sec.m0d4s.me`)"
networks:
  net1:
```

Make sure:
<ul>
<li>Create the filesystem architecture as needed by the docker-compose file</li>
<li>Change the domain names in the docker-compose file to your own domain names OR add the current domain names to your /etc/hosts file</li>
</ul>

