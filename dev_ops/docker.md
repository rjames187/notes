# Docker

An image is a definition. A container is a running instance of an image.

## Run command

`docker run -d -p hostport:containerport namespace/name:tag`

Note:
- the d flag runs in detached mode (doesn't block terminal)
- the p flag forwards a container port to a host port
- can add `-e var=val` to set an environment variable

## Stop commands

`docker stop` - issues SIGTERM signal

`docker kill` - issues SIGKILL signal (stronger)

## Volumes

A way to persist state between container restarts by giving a container access to the host file system

`docker volume create volume-name`

`docker volume ls` - to see all volumes

`docker volume inspect volume-name` - to see where the volume is located

Can add `-v hostvolume:containerpath` to run command for a volume

## Execute

`docker exec CONTAINER_ID COMMAND` to run a command inside of a container

`docker exec -it CONTAINER_ID /bin/sh` - to get a shell session inside of a container

## Bridge Networks

Bridge networks allow containers to communicate with eachother within an isolated system

`docker network create network-name`

`docker network ls`

Add `--network network-name` flag when running
Add `--name <name>` to give a container an address in a network

## Logs

`docker logs [OPTIONS] CONTAINER_ID` to see logs

Options:
- -f for a continuous stream
- 











