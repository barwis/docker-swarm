# Service config template

Every service should be created in `/docker/services/service_name` directory

The most basic setup for service contains two files - main yaml config file (usually docker-compose.yml) and .env file (optionally).


## 1. .env file

Env file will contains user and group created for this specific service (security reasons). It usually looks (at least) like this:

```
# service .env file
PUID=service_name_user
GUID=docker
```

Where `service_name_user` for `emby` service will be `emby_user`

---

## 2. User

1. Create `docker` group if it doesn't exist:

```
    sudo groupadd docker
```

2. Create user for service and add it to `docker` group:

```
    sudo useradd service_name_user
    sudo usermod -a -G docker service_name_user
```

3. Set service directory ownership to newly created user (make sure to go one directory up):

```
    root@NAS:/docker/services# chown service_name_user service_name
```

4. Confirm all is set correctly 
```
root@NAS:/docker/services# ls -la
total 3
drwxr-xr-x 3 service_name_user docker 4096 Mar 23 10:12 service_name
```

## 3. Main config file

File name doesn't really matter, but I like to keep `docker-compose.yml` file, to test the image before adding it to the stack by running `docker-compose up` in the service directory.


I will use emby server as an example

```
version: "3.0"

services:
  emby:
    image: emby/embyserver
    env_file: ./emby.env                                                # [1]
    volumes:
      # - /var/data/emby/emby:/config 
      # - /srv/data/:/data
      - ./config:/config # Configuration directory
      - /home/lordex/media/shows:/mnt/shows # Media directory
      - /home/lordex/media/movies:/mnt/movies # Media directory
    deploy:
      labels:
        # traefik common
        - traefik.enable=true
        - "traefik.docker.network=traefik_public"                       # [2]

        # traefikv2
        - "traefik.http.routers.emby.rule=Host(`emby.yourdomain.com`)"  # [3]
        - "traefik.http.services.emby.loadbalancer.server.port=8096"    # [4]
    networks:
        - traefik_public                                                # [5]
    ports:
      - 8096:8096

networks:
  traefik_public:
    external: true
  internal:
    driver: overlay
    ipam:
      config:
        - subnet: 172.16.2.0/24                                         # [6]
```

---

[1]: service env vile, containing user and group  
[2]: make sure it attaches to traefik_public overlay network (traefik-specific label)  
[3]: host/url the service will be available under  
[4]: ports exposed by service added to traefik loadbalancer  
[5]: make sure it attaches to traefik_public overlay network  
[6]: unique static subnet. See [here](https://github.com/barwis/docker-swarm/blob/master/docs/appx-2-subnets.md)  



## 4. [Launch 🚀](https://github.com/barwis/docker-swarm/blob/master/docs/appx-1-cli-commands.md#launching-the-stack--and-confirming-its-running-) 

```
root@NAS:/docker/services/emby# docker stack deploy emby -c docker-compose.yml
Creating service emby_emby
```

..and confirm it's running:

```
root@NAS:/docker/services/emby# docker stack ls
NAME                   SERVICES   ORCHESTRATOR
...
emby                   1          Swarm
...
```