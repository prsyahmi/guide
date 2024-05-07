## Podman basic

### Running a container
Container can be run with `podman run` command (https://docs.podman.io/en/latest/markdown/podman-run.1.html)

Example to run empty ubuntu OS:  
`podman run -d --name ubuntu ubuntu:latest`

Deleting the running container  
`podman rm --force ubuntu`

Run, but this time let us enter the os.  
`podman run -it --rm --name ubuntu ubuntu:latest`

Run, but this time let us set the entrypoint.  
`podman run -it --rm --name ubuntu --entrypoint /bin/sh ubuntu:latest`  
same as  
`podman run -it --rm --name ubuntu ubuntu:latest /bin/sh`

The entrypoint is decided by image maker, in this case it is bash shell.

##### Options
- `-d` means detached, return to your terminal instead of getting stuck with container's app
- `-it` means interactive + alloc tty, so we can use keyboard input
- `--rm` means remove container immediately after exiting the container
- `--name` if you don't specify container name, a random name will be given and this will causes inconvenience
- `--entrypoint` set executable to run

### Logs / stdout
https://docs.podman.io/en/latest/markdown/podman-logs.1.html

Running container can produce output. The output can be seen with  
`podman logs --follow name_of_container`

The `--follow` option will keep waiting and printing the stdout until the container dies.

### Listing containers
To list all running container  
`podman ps`

To list both running and not running container  
`podman ps --all`

### Persistent Volume & Host mapped drives
https://docs.podman.io/en/latest/volume.html

File inside container is ephemeral. If you restart them, all data will be lost and reverted to
container default.

However, image developer can declare persistent `VOLUME` inside their `Dockerfile` and data in this path will be persisted.  
For example: `VOLUME /container/database.sqlite`  
When running container, these persistent volume will be given random ID if the name is not specified.

To create a named persistent volume for container, add  
`-v name_of_storage:/container/database.sqlite`.

This map the path inside container `/container/database.sqlite` to a persisted storage on the host disk at default podman location.
The location on the host running rootless by default is on `~/.local/share/containers/storage/volumes/name_of_storage`.

To map the path from host to container use  
`-v /host/file/path:/container/file/path`

To make it read-only, add `:ro`.  
`-v /host/file/path:/container/file/path:ro`

To view all volume, type  
`podman volume ls`

### Networking
https://docs.podman.io/en/stable/markdown/podman-network.1.html

Podman has internal networking (running slirp4netns on rootless mode).
Each container on the same network can communicate with each other by IP or DNS.
However the default networking created by podman doesn't have DNS capability.

Creating a new network will automatically enable DNS on that network.  
```
podman network create mynetwork  
podman network inspect mynetwork

# To view all network
podman network ls
```

Assigning network to container can be specified with `--network` as below
```
podman run --name myapp1 --network mynetwork ...
podman run --name myapp2 --network mynetwork ...
```

To communicate with other network via DNS, use: `myapp1.dns.podman`, `myapp2.dns.podman`

### Exposing Port
Port inside the container can be mapped to host with `-p host_port:container_port` for example:  
Mapping port (host) 8080 <- 5000 (container)  
`-p 8080:5000`

### Entering container
To enter the running container (must be running), use `podman exec` (https://docs.podman.io/en/latest/markdown/podman-exec.1.html).

This command can execute binary/shell and to enter the shell, we must execute `/bin/bash` or `/bin/sh` inside the container:  
`podman exec -it name_of_container /bin/bash`

The `-it` is short for interactive (open stdin) and tty (allocate tty and attach to stdin like terminal do)
This can be used to execute any executables inside the container such as mysqldump, etc
