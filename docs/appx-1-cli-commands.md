# Appendix 1: useful commands cheatsheet

# `docker stack`

## list active stacks

```
root@NAS:/# docker stack ls
NAME                   SERVICES   ORCHESTRATOR
traefik                1          Swarm
traefik-forward-auth   2          Swarm
traefikv2              1          Swarm
```

## deploy stack

```
root@NAS:/# docker stack deploy prefered_stack_name -c <path-to-docker-compose.yml>
```

> Note: I suggest you set `prefered_stack_name` as you would set `container_name` for docker-compose. Unfortunately docker stack does not support `container_name` yet (23. March 2022).

&nbsp;

## kill/remove stack

```
root@NAS:/# docker stack rm stack_name
```

&nbsp;

&nbsp;


# `docker swarm`

## initialise swarm

```
root@NAS:/#  docker swarm init
```

## list swarm nodes:

```
root@NAS:/# docker node ls
```

## get docker swarm join command:

```
root@NAS:/# docker swarm join-token <role>
```

example:
```
root@NAS:/# docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4axzto1ga0xkps84tb6nv53otq6q1nrsgrpuq6ocrlyawo2u0j-01wp71f0oyzcwhitbue12qg83 192.168.50.250:2377

```



# launching the stack ( and confirming it's running )

For every service, running stack looks exactly the same. 
Assuming that the service name is `my_service` and its directory sits under `/docker/services/` like so:
```
docker
├── services
    ├── my_service
    └── ...
```

1. Enter the service directory: `cd /docker/services/my_service/`
2. Launch the cleanup stack by running: 
```
root@NAS:/docker/services/my_service# docker stack deploy my-servivce-label -c docker-compose.yml
```

You should see something like:

```
Creating network my_service_internal
Creating service my-servivce-label_my_service
```

> Note: Creating network will be optional, depending on whether you've configuring a new network for service  


3. Make sure it's been deployed correctly by running `docker stack ls`:
```
root@NAS:/docker/services/my_service# docker stack ls
NAME                   SERVICES   ORCHESTRATOR
my-servivce-label      1          Swarm
```
&nbsp;

