# Traefik v2

The main course of this tutorial. 

Table of contents:
1. Overlay network
2. Traefik v2  
    1. traefik config file
    2. .env file
    3. main yaml file
3. Launch
&nbsp;

---

Directory structure:

```
docker
└── services
    ├── overlay-network
    │   └── docker-compose.yml
    └── traefik
        ├── data
        │   ├── acme.json
        │   └── traefik.log
        ├── traefik.toml
        ├── traefik.env
        └── docker-compose.yml
```

&nbsp;



## 1. Overlay network [↗](https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/traefik/#prepare-the-docker-service-config)


We'll want an overlay network, independent of our traefik stack, so that we can attach/detach all our other stacks (including traefik) to the overlay network. This way, we can undeploy/redepoly the traefik stack without having to bring down every other stack first


```
# overlay-network/docker-compose.yml

version: "3.2"

# What is this?
#
# This stack exists solely to deploy the traefik_public overlay network, so that
# other stacks (including traefik-app) can attach to it

services:
  scratch:
    image: scratch
    deploy:
      replicas: 0
    networks:
      - public

networks:
  public:
    driver: overlay
    attachable: true
    ipam:
      config:
        - subnet: 172.16.200.0/24
```

---

&nbsp;


## 2. Traefik v2  [↗](https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/traefik/)

Go to `services/traefik` directory:

### 1. Create `traefik.toml` config file (to keep the main yaml file nice and tidy)


```
# traefik/traefik.toml

[global]
  checkNewVersion = true

# Enable the Dashboard
[api]
  dashboard = true

# Write out Traefik logs
[log]
  level = "INFO"
  filePath = "/traefik.log" # path to the file will be mapped in the main yaml config file

[entryPoints.http]
  address = ":80"
  # Redirect to HTTPS (why wouldn't you?)
  [entryPoints.http.http.redirections.entryPoint]
    to = "https"
    scheme = "https"

[entryPoints.https]
  address = ":443"
  [entryPoints.https.http.tls]
    certResolver = "main"

# Let's Encrypt
[certificatesResolvers.main.acme]
  email = "your@email.com"
  storage = "acme.json"  # path to the file will be mapped in the main yaml config file
  # uncomment to use staging CA for testing
  # caServer = "https://acme-staging-v02.api.letsencrypt.org/directory"
  [certificatesResolvers.main.acme.dnsChallenge]
    provider = "cloudflare"
  # Uncomment to use HTTP validation, like a caveman!
  # [certificatesResolvers.main.acme.httpChallenge]
  #  entryPoint = "http"

# Docker Traefik provider
[providers.docker]
  endpoint = "unix:///var/run/docker.sock"
  swarmMode = true
  watch = true
```

> Note: Make sure to update your let's encrypt email


&nbsp;


### 2. Create `traefik.env` file


.env file will contain environment variables required by the provider you chose in the LetsEncrypt DNS Challenge section of traefik.toml. Full configuration options can be found in the [Traefik documentation](https://doc.traefik.io/traefik/https/acme/#providers). Route53 and CloudFlare examples are below.

```
# traefik/traefik.env

# CloudFlare example
CLOUDFLARE_EMAIL=your@email.com
CLOUDFLARE_API_KEY=your_secret_key

# Route53 example
# AWS_ACCESS_KEY_ID=<your-aws-key>
# AWS_SECRET_ACCESS_KEY=<your-aws-secret>

```

> Note: Make sure to update your email

&nbsp;

### 3. Create  `docker-compose.yml` file.

This is the main yaml config file for traefik service.



```
# traefik/docker-compose.yml
version: "3.2"

services:
  app:
    image: traefik:v2.4
    env_file: ./traefik.env
    # Note below that we use host mode to avoid source nat being applied to our ingress HTTP/HTTPS sessions
    # Without host mode, all inbound sessions would have the source IP of the swarm nodes, rather than the
    # original source IP, which would impact logging. If you don't care about this, you can expose ports the 
    # "minimal" way instead
    ports:
      - target: 80
        published: 80
        protocol: tcp
        mode: host
      - target: 443
        published: 443
        protocol: tcp
        mode: host
      - target: 8080
        published: 8080
        protocol: tcp
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./:/etc/traefik
      - ./traefik.toml:/traefik.toml    # traefik config file
      - ./data/traefik.log:/traefik.log # log file
      - ./data/acme.json:/acme.json     # cert storage file
    networks:
      - traefik_public
    # Global mode makes an instance of traefik listen on _every_ node, so that regardless of which
    # node the request arrives on, it'll be forwarded to the correct backend service.
    deploy:
      mode: global
      labels:
        - "traefik.docker.network=traefik_public"
        - "traefik.http.routers.api.rule=Host(`traefik.yourdomain.com`)"
        - "traefik.http.routers.api.entrypoints=https"
        - "traefik.http.routers.api.tls.domains[0].main=yourdomain.com"
        - "traefik.http.routers.api.tls.domains[0].sans=*.yourdomain.com"        
        - "traefik.http.routers.api.tls=true"
        - "traefik.http.routers.api.tls.certresolver=main"
        - "traefik.http.routers.api.service=api@internal"
        - "traefik.http.services.dummy.loadbalancer.server.port=9999"

        # uncomment this to enable forward authentication on the traefik api/dashboard
        - "traefik.http.routers.api.middlewares=forward-auth"      
      placement:
        constraints: [node.role == manager]

networks:
  traefik_public:
    external: true

```

> Note: Make sure to change `yourdomain.com` to your domain

&nbsp;

---

&nbsp;

## 3. [Launch 🚀](https://github.com/barwis/docker-swarm/blob/master/docs/appx-1-cli-commands.md#launching-the-stack--and-confirming-its-running-) 

&nbsp;


1. Launch overlay network. 

Make sure you set `traefik` as stack name (network name is based on service name and we'll need this network to attach to when adding other services ). If you have a good reason to give the stack a different name, make sure to update network name accordingly.

```
root@NAS:/docker/services/overlay-network# docker stack deploy traefik -c docker-compose.yml
Creating network traefik_public
Creating service traefik_scratch
[root@kvm ~]#
```

&nbsp;

2. Deploy traefik stack:
```
root@NAS:/docker/services/traefik# docker stack deploy traefikv2 -c docker-compose.yml
Creating service traefikv2_app
```

3. confirm both are running:

```
root@NAS:/docker/services/traefik# docker stack ls
NAME                   SERVICES   ORCHESTRATOR
traefik                1          Swarm
traefikv2              1          Swarm
```

4. After a minute or two, go to https://traefik.yourdomain.com/ and you should see a nice traefik dashboard.

