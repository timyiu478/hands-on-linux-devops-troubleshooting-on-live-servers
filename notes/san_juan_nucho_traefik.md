## "San Juan": mucho Traefik

### Problem

Description: There is a Traefik load balancer that must be up and running. The server and the backend services are managed by Docker Compose. Running curl -s app.sadserver | head -n1 must return the host ID of one of the backend servers, running the command again must return a new host ID. The server seems to be working some times, some others fails or just times out.

The round-robin configuration should make the webserver iterate through the back-end servers.

Root (sudo) Access: True

Test: curl -s app.sadserver | head -n1 returns something like Hostname:

The "Check My Solution" button runs the script /home/admin/agent/check.sh, which you can see and execute.

### Solution

#### 1. check the docker-compose.yml

2 problems:

* Not all backend servers use the same port
    * All backends must use the same port internally (Traefik won't magically remap different ports in a single LB pool)
* Mixed container network: `web` and `internel`

```
admin@i-06b169eb07f44d33d:~$ cat app/docker-compose.yml 
services:
  traefik:
    image: traefik:v3.6.1
    restart: unless-stopped
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--log.level=DEBUG"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - web

  app01:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - web

  app02:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "81"
    networks:
      - web

  app03:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - web

  app04:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - internal

networks:
  web:
  internal:
```

#### 2. fix the docker-compose.yml

```
admin@i-06b169eb07f44d33d:~$ cat app/docker-compose.yml 
services:
  traefik:
    image: traefik:v3.6.1
    restart: unless-stopped
    command:
      - "--api.dashboard=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--log.level=DEBUG"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - web

  app01:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - web

  app02:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - web

  app03:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - web

  app04:
    image: traefik/whoami
    restart: unless-stopped
    labels:
      traefik.enable: "true"
      traefik.http.routers.app.rule: Host(`app.sadserver`)
      traefik.http.services.app.loadbalancer.server.port: "80"
    networks:
      - web

networks:
  web:
  internal:
```

#### 3. restart containers

```
admin@i-06b169eb07f44d33d:~/app$ docker compose down
[+] Running 6/6
 ✔ Container app-traefik-1  Removed                                                                                                                                         1.1s 
 ✔ Container app-app01-1    Removed                                                                                                                                         1.1s 
 ✔ Container app-app02-1    Removed                                                                                                                                         1.1s 
 ✔ Container app-app03-1    Removed                                                                                                                                         1.1s 
 ✔ Container app-app04-1    Removed                                                                                                                                         1.1s 
 ✔ Network app_web          Removed                                                                                                                                         0.3s 
admin@i-06b169eb07f44d33d:~/app$ docker compose up -d
[+] Running 6/6
 ✔ Network app_web          Created                                                                                                                                         0.1s 
 ✔ Container app-app03-1    Started                                                                                                                                         0.9s 
 ✔ Container app-app04-1    Started                                                                                                                                         0.9s 
 ✔ Container app-app01-1    Started                                                                                                                                         1.0s 
 ✔ Container app-traefik-1  Started                                                                                                                                         1.1s 
 ✔ Container app-app02-1    Started                                                                                                                                         0.9s 
admin@i-06b169eb07f44d33d:~/app$ curl -s app.sadserver | head -n1 
Hostname: 47277ef776d4
admin@i-06b169eb07f44d33d:~/app$ 
```
