# Service subnets

### Setup unique static subnets for every stack you deploy. This avoids IP/gateway conflicts which can otherwise occur when you're creating/removing stacks a lot. 

***


In order to avoid IP addressing conflicts as we bring swarm networks up/down, we will statically address each docker overlay network. In order to do so, in .yml (docker-compose.yml) file for each service, under network sections, set uniuqe subnet for internal network like this:

```
networks:
  internal:
    driver: overlay
    ipam:
      config:
        - subnet: 172.16.0.0/24 # unique subnet
```

# My personal subnets list
For reference purposes, here's my list

|   Network         |   Range         |
|-------------------|----------------:|
| Traefik           | _unspecified_   |
| Docker-cleanup	  | 172.16.0.0/24   |
| Emby              | 172.16.1.0/24   |
