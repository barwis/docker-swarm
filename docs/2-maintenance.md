# Maintenance

This step is completely optional. The idea is to run two helper services: 
1. automated cleanup
2. automatic updates

&nbsp;

&nbsp;

## 1. Automated cleanup [↗](https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/docker-swarm-mode/#setup-automated-cleanup)

Docker swarm doesn't do any cleanup of old images, so as you experiment with various stacks, and as updated containers are released upstream, you'll soon find yourself loosing gigabytes of disk space to old, unused images.

To address this, we'll run the "meltwater/docker-cleanup" container on all of our nodes. The container will clean up unused images after 30 minutes.

Directory structure and files content:

```
docker-cleanup/
├── docker-cleanup.env
└── docker-compose.yml
```

```
# docker-cleanup.env

# add/exclude containers we want to keep:
#
# KEEP_IMAGES=container1,container2,container3 ...
#
KEEP_IMAGES=traefik
DEBUG=1
```

```
# docker-compose.yml

version: "3"

services:
  docker-cleanup:
    image: meltwater/docker-cleanup:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker:/var/lib/docker
    networks:
      - internal
    deploy:
      mode: global
    env_file: ./docker-cleanup.env

networks:
  internal:
    driver: overlay
    ipam:
      config:
        - subnet: 172.16.0.0/24
```
> &nbsp;
>
> Note: regarding network config, please refer to [my subnets list](appx-2-subnets.md)
>
> &nbsp; 



&nbsp;

### [Launch the stack 🚀](https://github.com/barwis/docker-swarm/blob/master/docs/appx-1-cli-commands.md#launching-the-stack--and-confirming-its-running-)

&nbsp;


---


## 2. Automatic updates [↗](https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/docker-swarm-mode/#setup-automatic-updates)

If your swarm runs for a long time, you might find yourself running older container images, after newer versions have been released. If you're the sort of geek who wants to live on the edge, configure shepherd to auto-update your container images regularly.

Directory structure and files content:

```
shepherd/
├── docker-compose.yml
└── shepherd.env
```

```
# shepherd.env

# Don't auto-update Plex or Emby (or Jellyfin), I might be watching a movie!
# (Customize this for the containers you _don't_ want to auto-update)
BLACKLIST_SERVICES="plex_plex emby_emby jellyfin_jellyfin"
# Run every 24 hours. Note that SLEEP_TIME appears to be in seconds.
SLEEP_TIME=86400
```


```
# docker-compose-yml

version: "3"

services:
  shepherd-app:
    image: mazzolino/shepherd
    env_file : ./shepherd.env
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    labels:
      - traefik.enable=false
    deploy:
      placement:
        constraints: [node.role == manager]
```

> Note: If you're following this tutorial, traefk is not set up yet, but we're adding traefik-related label for future reference.

&nbsp;

### [Launch the stack 🚀](https://github.com/barwis/docker-swarm/blob/master/docs/appx-1-cli-commands.md#launching-the-stack--and-confirming-its-running-)

&nbsp;
