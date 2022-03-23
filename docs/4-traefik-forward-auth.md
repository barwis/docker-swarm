# Traefik Forward Auth [↗](https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/traefik-forward-auth/)

Everything is explained in the linked article. To keep it short, I'll be using Google for authentication. Please refer to [this](https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/traefik-forward-auth/google/)

```
docker
└── services
    └── traefik-forward-auth/
        ├── traefik-forward-auth.env
        └── traefik-forward-auth.yml
```

```
# traefik-forward-auth.env

PROVIDERS_GOOGLE_CLIENT_ID=<google_client_id>
PROVIDERS_GOOGLE_CLIENT_SECRET=<google_client_secret>
SECRET=<random_string_make_something_up>
# comment out AUTH_HOST if you'd rather use individual redirect_uris (slightly less complicated but more work)
AUTH_HOST=auth.yourdomain.com
COOKIE_DOMAINS=yourdomain.com
WHITELIST=your@gmail.com, your.second@gmail.com

```

```
# traefik-forward-auth.yml

version: "3.2"

services:
  traefik-forward-auth:
    image: thomseddon/traefik-forward-auth:2.1.0
    env_file: ./traefik-forward-auth.env
    networks:
      - traefik_public
    deploy:
      labels: # you only need these if you're using an auth host
        # traefik
        - traefik.enable=true

        # traefikv2
        - "traefik.docker.network=traefik_public"
        - "traefik.http.routers.auth.rule=Host(`auth.yourdomain.com`)"
        - "traefik.http.routers.auth.entrypoints=https"
        - "traefik.http.routers.auth.tls=true"
        - "traefik.http.routers.auth.tls.domains[0].main=yourdomain.com"
        - "traefik.http.routers.auth.tls.domains[0].sans=*.yourdomain.com"        
        - "traefik.http.routers.auth.tls.certresolver=main"
        - "traefik.http.routers.auth.service=auth@docker"
        - "traefik.http.services.auth.loadbalancer.server.port=4181"
        - "traefik.http.middlewares.forward-auth.forwardauth.address=http://traefik-forward-auth:4181"
        - "traefik.http.middlewares.forward-auth.forwardauth.trustForwardHeader=true"
        - "traefik.http.middlewares.forward-auth.forwardauth.authResponseHeaders=X-Forwarded-User"
        - "traefik.http.routers.auth.middlewares=forward-auth"
  # This simply validates that traefik forward authentication is working
  whoami:
    image: containous/whoami
    networks:
      - traefik_public
    deploy:
      labels:
        # traefik
        - traefik.enable=true
        # traefikv2
        - "traefik.docker.network=traefik_public"
        - "traefik.http.routers.whoami.rule=Host(`whoami.yourdomain.com`)"
        - "traefik.http.routers.whoami.entrypoints=https"
        - "traefik.http.services.whoami.loadbalancer.server.port=80"
        - "traefik.http.routers.whoami.middlewares=forward-auth" # this line enforces traefik-forward-auth  


networks:
  traefik_public:
    external: true
```
&nbsp;

---

&nbsp;


### [Launch the stack 🚀](https://github.com/barwis/docker-swarm/blob/master/docs/appx-1-cli-commands.md#launching-the-stack--and-confirming-its-running-)

&nbsp;

To confirm it's working, go to `https://whoami.yourdomain.com`, you should be asked to log in with your google account, After that you should see something like:

```
Hostname: (redacted)
IP: 127.0.0.1
IP: 172.16.200.13
IP: 172.26.0.7
RemoteAddr: (redacted)
GET / HTTP/1.1
Host: whoami.yourdomain.com
User-Agent: (redacted)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate, br
Accept-Language: en-GB,en-US;q=0.9,en;q=0.8,pl;q=0.7
Cookie: (redacted)
Sec-Ch-Ua: " Not A;Brand";v="99", "Chromium";v="99", "Google Chrome";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "macOS"
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Upgrade-Insecure-Requests: 1
X-Forwarded-For: (redacted)
X-Forwarded-Host: whoami.yourdomain.com
X-Forwarded-Port: 443
X-Forwarded-Proto: https
X-Forwarded-Server: (redacted)
X-Forwarded-User: your@gmail.com
X-Real-Ip: 192.168.50.1
```

From now on, for every service you want to hide behind auth wall, you need to add this deploy label in the main yaml file:

```        - "traefik.http.routers.whoami.middlewares=forward-auth" ```