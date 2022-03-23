## Docker swarm 
1. Initialise swarm

```
[root@nas.local ~]# docker swarm init
Swarm initialized: current node (b54vls3wf8xztwfz79nlkivt8) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-2orjbzjzjvm1bbo736xxmxzwaf4rffxwi0tu3zopal4xk4mja0-bsud7xnvhv4cicwi7l6c9s6l0 \
    202.170.164.47:2377\

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

[root@nas.local ~]#
```

> Note: You will obviously see different id/token/ID - duh...

&nbsp;  

2. Run `docker node ls` to confirm that you have a 1-node swarm:

```
[root@nas.local ~]# docker node ls
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
hdbio4pg0snkkb75iogw3imrv *   NAS        Ready     Active         Leader           20.10.12
[root@nas.local ~]#
```

> **IMPORTANT:** if you ever forget the command to join to this particular swarm, you get it by typing:
> ```
> [root@nas.local ~]# docker swarm join-token manager
> ```
> &nbsp;

---

Also see: [CLI commands cheatsheet](appx-1-cli-commands.md)
