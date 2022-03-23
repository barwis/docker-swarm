# Prerequisities

> NOTE: If you're doing this via ssh connection, make sure you're in SU mode `sudo su`

https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/docker-swarm-mode/

Add some handy bash auto-completion for docker. Without this, you'll get annoyed that you can't autocomplete docker stack deploy <blah> -c <blah.yml> commands.

```
cd /etc/bash_completion.d/
curl -O https://raw.githubusercontent.com/docker/cli/b75596e1e4d5295ac69b9934d1bd8aff691a0de8/contrib/completion/bash/docker
```

Install some useful bash aliases on each host

```
cd ~
curl -O https://raw.githubusercontent.com/funkypenguin/geek-cookbook/master/examples/scripts/gcb-aliases.sh
echo 'source ~/gcb-aliases.sh' >> ~/.bash_profile
```

# Directory structure.

I will be keeping everything in `docker` directory placed in root folder:

```
docker
└── services
    ├── service_1
    ├── service_2
    ├── ...
```
