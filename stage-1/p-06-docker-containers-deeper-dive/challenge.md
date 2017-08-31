Let’s play with Docker containers !

### What you will do

In this first lab, you’ll put into practice the base commands to manage Docker containers. That means you will start to play and to have fun with containers :)

Important note: in the labs, when you see a command inside some grey rectangle, you just need to click on it so it is executed in the terminal (but if you want you can also write it manually into the terminal). Let’s see the first example below to get the version of the Docker Engine running on the platform.

```
docker version
```

You should get an output like the following that shows Docker Engine (Server) and Client are running version 17.03.0.

```
Client:
 Version:      17.06.1-ce-rc1
 API version:  1.30
 Go version:   go1.8.3
 Git commit:   77b4dce
 Built:        Fri Jul 14 07:32:23 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.06.1-ce-rc1
 API version:  1.30 (minimum version 1.12)
 Go version:   go1.8.3
 Git commit:   77b4dce
 Built:        Fri Jul 14 07:33:35 2017
 OS/Arch:      linux/amd64
 Experimental: true
 ```

### Launch containers

#### Running a container in foreground

Let’s go direct to the point and launch a container based on alpine, and use the options to run it interactive mode.

```
docker container run -ti alpine
```

Seems there is a problem there… you should have an error message like the following one.

```
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
0a8490d0dfd3: Pull complete
Digest: sha256:dfbd4a3a8ebca874ebd2474f044a0b33600d4523d03b0df76e5c5986cb
02d7e8
Status: Downloaded newer image for alpine:latest
docker: Error response from daemon: No command specified.
See 'docker run --help'.
```

The No command specified error means that you ran a container that does not have any default command and you did not specify one neither.

Let’s fix that and run a new container providing the sh command.

```
docker container run -ti alpine sh
```

Once created, the container should run the sh command. You should then now be in a shell running in an alpine linux. Let’s check that.

```
cat /etc/issue
```

Part of the output should be like

```
Welcome to Alpine Linux 3.5
```

You can now exit the container.

```
exit
```

### Running a container in background

Very often, containers are ran in background. They can expose services like HTTP API, databases, …

Let’s now use the mongo official image (for those who might not be very familiar with it, MongoDB is a very popular NoSQL database) and run a container in background (using the -d option).

Note: we have also used the –name option to assign a name to the container.

The command should pull the mongo image and return the ID of the newly created container.

```
$ docker container run -d --name mongo mongo:3.2
Unable to find image 'mongo:3.2' locally
3.2: Pulling from library/mongo
5233d9aed181: Pull complete
5bbfc055e8fb: Pull complete
03e4cc4b6057: Pull complete
8319d631fd37: Pull complete
797ca64b920a: Pull complete
4f57a996ba49: Pull complete
5778b19a1103: Pull complete
a763733f623a: Pull complete
0101d9086c98: Pull complete
8b0a7b12275b: Pull complete
bfe8dd06ccf2: Pull complete
Digest: sha256:b2c7025b69223fca43a2c7d60c30b2bffac4df20314f11d2b46f4d8d4eaf29e9
Status: Downloaded newer image for mongo:3.2
d4e640adcde1673b00e2e9a09d247b28afd5948866712f49b2c2e7426b37340f
```

An interesting thing we can do is to use the name of the container or the ID returned by the previous command and jump into the running container. For this, we will use the exec command and the -ti options (to get an interactive tty).

```
$ docker container exec -ti mongo bash
root@d4e640adcde1:/#
```

We are now in the container, that can be really handy for debugging purposes sometimes. Let’s check the running processes.

The output should be pretty much like the following one. As we can see, the process with PID 1 is mongod which is the MongoDB deamon, this is the command ran by default when a container is instantiated from a mongo image.

```
root@d4e640adcde1:/# ps ax
  PID TTY      STAT   TIME COMMAND
    1 ?        Ssl    0:00 mongod
   33 pts/0    Ss     0:00 bash
   39 pts/0    R+     0:00 ps ax
```

You can now exit the mongodb container:

```
exit
```

### Inspection of a container

A container is a quite complex thing under the hood, the container API provides the inspect command to get all its details.

Let’s launch a container based on nginx in background and name it www.

```
$ docker container run --name www -d nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
94ed0c431eb5: Pull complete
9406c100a1c3: Pull complete
aa74daafd50c: Pull complete
Digest: sha256:788fa27763db6d69ad3444e8ba72f947df9e7e163bad7c1f5614f8fd27a311c3
Status: Downloaded newer image for nginx:latest
8e8298ce53c59834c9adad1c93c3e904a7c946c5fad78c5b913868717d09ae23
```

Use the inspect command against this container.

```
$ docker container inspect www
[
    {
        "Id": "8e8298ce53c59834c9adad1c93c3e904a7c946c5fad78c5b913868717d09ae23",
        "Created": "2017-08-31T17:09:03.959716689Z",
        "Path": "nginx",
        "Args": [
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 1263,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2017-08-31T17:09:04.653773769Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:b8efb18f159bd948486f18bd8940b56fd2298b438229f5bd2bcf4cedcf037448",
        "ResolvConfPath": "/graph/containers/8e8298ce53c59834c9adad1c93c3e904a7c946c5fad78c5b913868717d09ae23/resolv.conf",
        "HostnamePath": "/graph/containers/8e8298ce53c59834c9adad1c93c3e904a7c946c5fad78c5b913868717d09ae23/hostname",
        "HostsPath": "/graph/containers/8e8298ce53c59834c9adad1c93c3e904a7c946c5fad78c5b913868717d09ae23/hosts",
        "LogPath": "/graph/containers/8e8298ce53c59834c9adad1c93c3e904a7c946c5fad78c5b913868717d09ae23/8e8298ce53c59834c9adad1c93c3e904a7c946c5fad78c5b913868717d09ae23-json.log",
        "Name": "/www",
        "RestartCount": 0,
        "Driver": "overlay2",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": -1,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/graph/overlay2/bae8050c853deb1890fedbae845004e90501e9a2dd84a59538721e79a61202e3-init/diff:/graph/overlay2/f2c199400a27eb037ea07fd68e1efff447000cc270fb3f9aa678cd45c7cc0012/diff:/graph/overlay2/189977dcf941f4c339b3c5403134d9afe5c9de02339369b60c11edba6d7f0470/diff:/graph/overlay2/0085ae4d54dd13fc10272c7fd93f77fe16798b797d4e2bf5eacd5901817caa1d/diff",
                "MergedDir": "/graph/overlay2/bae8050c853deb1890fedbae845004e90501e9a2dd84a59538721e79a61202e3/merged",
                "UpperDir": "/graph/overlay2/bae8050c853deb1890fedbae845004e90501e9a2dd84a59538721e79a61202e3/diff",
                "WorkDir": "/graph/overlay2/bae8050c853deb1890fedbae845004e90501e9a2dd84a59538721e79a61202e3/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "8e8298ce53c5",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.13.3-1~stretch",
                "NJS_VERSION=1.13.3.0.1.11-1~stretch"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "ArgsEscaped": true,
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {},
            "StopSignal": "SIGTERM"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "928535bb52c9bc24196eff9d5cc924d5f1db463bb26df89542506f0b22c4063b",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/928535bb52c9",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "356fac0beef9fac6b2ad77e8e5d78e3252b82fc5399de2d3d69ec07a705e84cc",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.3",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:03",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "296296fcd19cb83d7c850343850f57a79869bf4ae6360e95106c91a609d1e8bb",
                    "EndpointID": "356fac0beef9fac6b2ad77e8e5d78e3252b82fc5399de2d3d69ec07a705e84cc",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:03",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

There is a lot of information here, so we will use the Go template notation to get only the information we are interested in: the hostname and the API address of the container.

The Hostname key is under the Config one, and can be retrieved with the following command.
You should see the below sha (not the exact value but something similar):

```
$ docker container inspect --format "{{ .Config.Hostname }}" www
8e8298ce53c5
```

The IPAdress key is under the NetworkSettings and can be retrieved with
You should see the below IP address (value may vary):

```
$ docker container inspect --format "{{ .NetworkSettings.IPAddress }}" www
172.17.0.3
```

Select some other elements of the whole json structure returned by the inspect command and try to get them using the Go template format.

### Explore the other commands of the container’s API

All the commands linked to the container can be listed with

```
$ docker container --help

Usage:  docker container COMMAND

Manage containers

Options:
      --help   Print usage

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  inspect     Display detailed information on one or more containers
  kill        Kill one or more running containers
  logs        Fetch the logs of a container
  ls          List containers
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  prune       Remove all stopped containers
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  run         Run a command in a new container
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker container COMMAND --help' for more information on a command.
```

We have already seen some of them and will see some other ones in the following but feel free to test them by yourself and to experiment with some fun container commands and features.

### Understand the container layer

The container layer is the layer created when a container is run. This is the layer in which the changes applied are stored. This layer is deleted when the container is removed and thus cannot be used for persistent storage.

We will start by running a container in interactive mode based on the ubuntu image.

```
docker container run -ti ubuntu
```

Note: you can notice here that we do not have any error message as this was the case when we ran our first alpine container. The reason for this is because ubuntu does have a default command bash that is specified. The bash command with the -ti option enables us to get into an interactive shell within this container.

figlet is a package that takes a text as input and displays the same text in an Ascii-art format. By default this package is not installed in the ubuntu image, but let’s check that.

```
figlet
```

You should get something like

```
bash: figlet: command not found
```

We will update the packages and install figlet then

```
apt-get update -y
apt-get install figlet
```

Make sure figlet is correctly installed

```
figlet Holla
```

You should get a nicely formated output

```
 _   _       _ _
| | | | ___ | | | __ _
| |_| |/ _ \| | |/ _` |
|  _  | (_) | | | (_| |
|_| |_|\___/|_|_|\__,_|
```

You can now exit the container

```
exit
```

We will now run a new container using the ubuntu image.

```
docker container run -ti ubuntu
```

Is figlet package still there ? Let’s figure this out.

```
figlet
```

You should get an error message like the following.

```
bash: figlet: command not found
```

Can you explain why ?

In fact, this new ubuntu container is different from the previous one, the one in which figlet was installed. Both containers have their own container’s layer. Remember, the container’s layer is the read-write layer created when a container is run, it’s the place where changes done within the container are saved.

Let’s exit from this container.

```
exit
```

We can list all the running container.

```
docker container ls
```

And then all the container existing on the host.

```
docker container ls -a
```

From this list, get the id of the container in which we installed the figlet package and restart the container using the ‘start’ command.

```
docker container start CONTAINER_ID
```

Run an interactive shell in this container. We will use the exec command to do so.

```
docker container exec -ti CONTAINER_ID bash
```

Verify figlet is present in this container.

```
figlet still there !
```

If you get the funny output, everything is fine.

We can now exit the container once again.

```
exit
```

### Cleanup
We will now remove all the containers from the machine. There should not be any container in the running state though. Let’s check that.

```
docker container ls -a
```

If we had the -q option to the previous command, we get only the ID of the container.

```
docker container ls -aq
```

This is really handy when we need to remove several containers at the same time as we can feed the rm command with this list of ids.

```
docker container rm -f $(docker container ls -aq)
```

There should not be any more container on the host.

```
docker container ls -a
```

### What we seen in this lab

We have started to play with containers and to understand the container layer, the read-write layer that is added to each container that is ran. We also started to play with the container API and the commands used the most (run, exec, ls, rm, inspect).

True or false: Once a container stops it is removed from the system? ( ) True (x) False

Which command helps you access the commandline on a running container?

docker container exec
