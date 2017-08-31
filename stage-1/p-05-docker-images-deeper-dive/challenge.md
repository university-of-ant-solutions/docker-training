### Let’s have fun with Docker images

In this lab we will see how to create an image from a container. Even if this is not used very often it’s interesting to try it at least once. Then we will focus on the image creation using a Dockerfile. We will then see how to get the details of an image through the inspection and explore the filesystem to have a better understanding of what happens behind the hood. We will end this lab with the image API.

### Image creation from a container

Let’s start by running an interactive shell in a ubuntu container.

```
$ docker container run -ti ubuntu bash

Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
d5c6f90da05d: Pull complete
1300883d87d5: Pull complete
c220aa3cfc1b: Pull complete
2e9398f099dc: Pull complete
dc27a084064f: Pull complete
Digest: sha256:34471448724419596ca4e890496d375801de21b0e67b81a77fd6155ce001edad
Status: Downloaded newer image for ubuntu:latest
root@c51811dd1ec7:/#
```

As we’ve done in the previous lab, we will install the figlet package in this container.

```
apt-get update
apt-get install -y figlet
```

We then exit from this container

```
exit
```

Get the ID of this container using the ls command (do not forget the -a option as the non running container are not returned by the ls command).

```
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED        STATUS                       PORTS               NAMES
c51811dd1ec7        ubuntu              "bash"              2 minutes ago       Exited (127) 2 seconds ago                       hopeful_hodgkin
```

Run the following command, using the ID retreived, in order to commit the container and create an image out of it.

```
docker container commit CONTAINER_ID

Eg:
$ docker container commit c51811dd1ec7
sha256:e3392cabc309cbbf822a06a70ff53c074c97bcb4b3a941f5932f1447422f05b1

```

Once it has been commited, we can see the newly created image in the list of available images.

```
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED        SIZE
<none>              <none>              e3392cabc309        22 seconds ago      161MB
ubuntu              latest              ccc7a11d65b1        2 weeks ago        120MB
```

From the previous command, get the ID of the newly created image and tag it so it’s named **ourfiglet**.

```
docker image tag IMAGE_ID ourfiglet

Eg:
docker image tag e3392cabc309 ourfiglet
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED        SIZE
ourfiglet           latest              e3392cabc309        3 minutes ago       161MB
ubuntu              latest              ccc7a11d65b1        2 weeks ago        120MB
```

Now we will run a container based on the newly created image named ourfiglet, and specify the command to be ran such as it uses the figlet package.
As figlet is present in our ourfiglet image, the command ran returns the following output.

```
$ docker container run ourfiglet figlet hello
 _          _ _
| |__   ___| | | ___
| '_ \ / _ \ | |/ _ \
| | | |  __/ | | (_) |
|_| |_|\___|_|_|\___/
```

This example shows that we can create a container, add all the libraries and binaries in it and then commit this one in order to create an image. We can then use that image as we would do for any other images. This approach is not the recommended one as it is not very portable.

In the following we will see how images are usually created, using a Dockerfile, which is a text file that contains all the instructions to build an image.

### Image creation using a Dockerfile

We will use a simple example in this section and build a hello world application in Node.js. We will start by creating a file in which we retrieve the hostname and display it.

Copy the following content into index.js file.

```
var os = require("os");
var hostname = os.hostname();
console.log("hello from " + hostname);
```

We will dockerrize this application and start by creating a Dockerfile for this purpose. We will use alpine as the base image, add a Node.js runtime and then copy our source code. We will also specify the default command to be ran upon container creation.

Create a file named Dockerfile and copy the following content into it.

```
FROM alpine
RUN apk update && apk add nodejs
COPY . /app
WORKDIR /app
CMD ["node","index.js"]
```

Let’s build our first image out of this Dockerfile, we will name it hello:v0.1

```
$ docker image build -t hello:v0.1 .
Sending build context to Docker daemon  86.15MB
Step 1/5 : FROM alpine
latest: Pulling from library/alpine
88286f41530e: Pull complete
Digest: sha256:1072e499f3f655a032e88542330cf75b02e7bdf673278f701d7ba61629ee3ebe
Status: Downloaded newer image for alpine:latest
 ---> 7328f6f8b418
Step 2/5 : RUN apk update && apk add nodejs
 ---> Running in 8e78746ef1ca
fetch http://dl-cdn.alpinelinux.org/alpine/v3.6/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.6/community/x86_64/APKINDEX.tar.gz
v3.6.2-100-g067e3db942 [http://dl-cdn.alpinelinux.org/alpine/v3.6/main]
v3.6.2-101-g81c01d9d86 [http://dl-cdn.alpinelinux.org/alpine/v3.6/community]
OK: 8440 distinct packages available
(1/8) Installing ca-certificates (20161130-r2)
(2/8) Installing libcrypto1.0 (1.0.2k-r0)
(3/8) Installing libgcc (6.3.0-r4)
(4/8) Installing http-parser (2.7.1-r1)
(5/8) Installing libssl1.0 (1.0.2k-r0)
(6/8) Installing libstdc++ (6.3.0-r4)
(7/8) Installing libuv (1.11.0-r1)
(8/8) Installing nodejs (6.10.3-r1)
Executing busybox-1.26.2-r5.trigger
Executing ca-certificates-20161130-r2.trigger
OK: 26 MiB in 19 packages
 ---> bb5bec7842bd
Removing intermediate container 8e78746ef1ca
Step 3/5 : COPY . /app
 ---> 1189279888bf
Removing intermediate container 718e4a3ff41b
Step 4/5 : WORKDIR /app
 ---> 1c5f8f2a5389
Removing intermediate container cc75ca4ed424
Step 5/5 : CMD node index.js
 ---> Running in 12a87cc640cf
 ---> 7db2cfeda331
Removing intermediate container 12a87cc640cf
Successfully built 7db2cfeda331
Successfully tagged hello:v0.1
```

We then create a container to check it is running fine.
You should then have an output similar to the following one (the ID will be different though).

```
$ docker container run hello:v0.1
hello from 16bd7027cd97
```

There are always several ways to write a Dockerfile, we can start from a Linux distribution and then install a runtime (as we did above) or use images where this has already been done for us.

To illustrate that, we will now create a new Dockerfile but we will use the mhart/alpine-node:6.9.4 image. This is not an official image but it’s a very well known and used one.

Create a new Dockerfile named Dockerfile-v2 and make sure it has the following content.

```
FROM mhart/alpine-node:6.9.4
COPY . /app
WORKDIR /app
CMD ["node","index.js"]
```

asically, it is not that different from the previous one, it just uses a base image that embeds alpine and a Node.js runtime so we do not have to install it ourself. In this example, installing Node.js is not a big deal, but it is really helpful to use image where a runtime (or else) is already packages when using more complex environments.

We will now create a new image using this Dockerfile.

```
$ docker image build -f Dockerfile-v2 -t hello:v0.2 .
Sending build context to Docker daemon  86.15MB
Step 1/4 : FROM mhart/alpine-node:6.9.4
6.9.4: Pulling from mhart/alpine-node
b7f33cc0b48e: Pull complete
73832816494e: Pull complete
Digest: sha256:30da0d8e916c0c1980974d1eb287ba8b9f645634a58c311e3770edf80ca382e7
Status: Downloaded newer image for mhart/alpine-node:6.9.4
 ---> d448eac1cfdb
Step 2/4 : COPY . /app
 ---> 9acbe3c7c8d2
Removing intermediate container 3aeb268075bd
Step 3/4 : WORKDIR /app
 ---> 224125da82b4
Removing intermediate container e2cb276a0993
Step 4/4 : CMD node index.js
 ---> Running in ba33f3174d98
 ---> f9cc174898a4
Removing intermediate container ba33f3174d98
Successfully built f9cc174898a4
Successfully tagged hello:v0.2
```

Note: as we do not use the default name for our Dockerfile, we use the -f option to point towards the one we need to use.

We now run a container from this image.
Once again, the output will look like the following.

```
$ docker container run hello:v0.2
hello from c7318350b477
```

### ENTRYPOINT vs COMMAND

In the 2 previous Dockerfile, we used CMD to define the command to be ran when a container is launched. As we have seen, there are several ways to define the command, using ENTRYPOINT and/or CMD. We will illustrate this on a new Dockerfile, named Dockerfile-v3, that as the following content.

```
FROM alpine
ENTRYPOINT ["ping"]
CMD ["localhost"]
```

Here, we define the ping command as the ENTRYPOINT and the localhost as the CMD, the command that will be ran by default is the concatenation of ENTRYPOINT and CMD: ping localhost. This command can be seen as a wrapper around the ping utility to which we can change the address we provide as a parameter.

Let’s create an image based on this new file.

```
$ docker image build -f Dockerfile-v3 -t ping:v0.1 .
Sending build context to Docker daemon  86.16MB
Step 1/3 : FROM alpine
 ---> 7328f6f8b418
Step 2/3 : ENTRYPOINT ping
 ---> Running in 82b9bc20fb8d
 ---> 3af62d8eee4d
Removing intermediate container 82b9bc20fb8d
Step 3/3 : CMD localhost
 ---> Running in e15668f6797d
 ---> a0038429dbf7
Removing intermediate container e15668f6797d
Successfully built a0038429dbf7
Successfully tagged ping:v0.1
```

We can run this image without specifying any command:
That should give a result like the following one.

```
$ docker container run ping:v0.1
PING localhost (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=64 time=0.574 ms
64 bytes from 127.0.0.1: seq=1 ttl=64 time=0.086 ms
64 bytes from 127.0.0.1: seq=2 ttl=64 time=0.101 ms
64 bytes from 127.0.0.1: seq=3 ttl=64 time=0.092 ms
64 bytes from 127.0.0.1: seq=4 ttl=64 time=0.102 ms
^C
--- localhost ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.086/0.191/0.574 ms
```

You can also override the default CMD indicating another IP address. We will use 8.8.8.8 which is the IP of a Google’s DNS.

```
docker container run ping:v0.1 8.8.8.8
```

That should return the following.

```
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=38 time=9.235 ms
64 bytes from 8.8.8.8: seq=1 ttl=38 time=8.590 ms
64 bytes from 8.8.8.8: seq=2 ttl=38 time=8.585 ms
```

### Image Inspection

As we have already seen with containers, and as we will see with other Docker’s components (volume, network, …), the inspect command is available for the image API and it returns all the information of the image provided.

The alpine image should already be present locally, if it’s not, run the following command to pull it.

```
$ docker image pull alpine
Using default tag: latest
latest: Pulling from library/alpine
Digest: sha256:1072e499f3f655a032e88542330cf75b02e7bdf673278f701d7ba61629ee3ebe
Status: Image is up to date for alpine:latest
```

Once we are sure it is there let’s inspect it.

```
$ docker image inspect alpine
[
    {
        "Id": "sha256:7328f6f8b41890597575cbaadc884e7386ae0acc53b747401ebce5cf0d624560",
        "RepoTags": [
            "alpine:latest"
        ],
        "RepoDigests": [
            "alpine@sha256:1072e499f3f655a032e88542330cf75b02e7bdf673278f701d7ba61629ee3ebe"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2017-06-27T18:42:16.849872208Z",
        "Container": "6a3726d15fee7d345097a26ebc1b9e5b4de25dee759a921def9059a4d0cd2261",
        "ContainerConfig": {
            "Hostname": "e1ede117fb1e",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/sh\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:ac1fc1931356fa238379d061cb216c4bed2f150991298c20b166accf0604d3b1",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "17.03.1-ce",
        "Author": "",
        "Config": {
            "Hostname": "e1ede117fb1e",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:ac1fc1931356fa238379d061cb216c4bed2f150991298c20b166accf0604d3b1",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 3965955,
        "VirtualSize": 3965955,
        "GraphDriver": {
            "Data": {
                "MergedDir": "/graph/overlay2/91426ae310c34b8b84121b5149c6a36a797ce4acbb52b022f593bd56e1495c01/merged",
                "UpperDir": "/graph/overlay2/91426ae310c34b8b84121b5149c6a36a797ce4acbb52b022f593bd56e1495c01/diff",
                "WorkDir": "/graph/overlay2/91426ae310c34b8b84121b5149c6a36a797ce4acbb52b022f593bd56e1495c01/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:5bef08742407efd622d243692b79ba0055383bbce12900324f75e56f589aedb0"
            ]
        }
    }
]
```

here is a lot of information in there:

- the layers the image is composed of
- the driver used to store the layers
- the architecture / os it has been created for
- metadata of the image
- …

We will not go into all the details now but it’s interesing to see an example of the Go template notation that enables to extract the part of information we need in just a simple command.

Let’s get the list of layers (only one for alpine)

```
$ docker image inspect --format "{{ json .RootFS.Layers }}" alpine | python -m json.tool
[
    "sha256:5bef08742407efd622d243692b79ba0055383bbce12900324f75e56f589aedb0"
]

$ docker image inspect --format "{{ json .RootFS.Layers }}" alpine
["sha256:5bef08742407efd622d243692b79ba0055383bbce12900324f75e56f589aedb0"]
```

Let’s try another example to query only the Architecture information

```
$ docker image inspect --format "{{ .Architecture }}" alpine
amd64
```

This should return amd64.

Feel free to play with the Go template format and get familiar with it as it’s really handy.

### Filesystem exploration
We first stop and remove all containers from your host (you might not be able to remove images if containers are using some of the layers).

```
$ docker container stop $(docker container ls -aq)
064686ab2f99
8a1775924101
$ docker container rm $(docker container ls -aq)
064686ab2f99
8a1775924101
```

Note: containers can also be removed in a non graceful way:

```
docker container rm -f $(docker container ls -aq)
```

We also remove all the images

```
$ docker image rm $(docker image ls -q)
Untagged: ourfiglet:latest
Deleted: sha256:cc561d087cca30d3b5c2dbd88e98fbccc700150fc4636a4330dcd297531b7a95
Deleted: sha256:54abd3b5477bbf38cf1ab41796de4971ce24f952e4c13de7364b734824e1efb7
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:34471448724419596ca4e890496d375801de21b0e67b81a77fd6155ce001edad
Deleted: sha256:ccc7a11d65b1b5874b65adb4b2387034582d08d65ac1817ebc5fb9be1baa5f88
Deleted: sha256:cb5450c7bb149c39829e9ae4a83540c701196754746e547d9439d9cc59afe798
Deleted: sha256:364dc483ed8e64e16064dc1ecf3c4a8de82fe7f8ed757978f8b0f9df125d67b3
Deleted: sha256:4f10a8fd56139304ad81be75a6ac056b526236496f8c06b494566010942d8d32
Deleted: sha256:508ceb742ac26b43bdda819674a5f1d33f7b64c1708e123a33e066cb147e2841
Deleted: sha256:8aa4fcad5eeb286fe9696898d988dc85503c6392d1a2bd9023911fb0d6d27081
```

We will now have a look inside the **/graph/overlay2** folder where the image and container layers are stored.

```
$ ls /graph/overlay2
backingFsBlockDev  l
```

As we do not have any images yet, there should not be anything in this folder.

Let’s pull an nginx image
You should get something like the following where we can see that 3 layers are pulled.

```
$ docker image pull nginx
Using default tag: latest
latest: Pulling from library/nginx
94ed0c431eb5: Pull complete
9406c100a1c3: Pull complete
aa74daafd50c: Pull complete
Digest: sha256:788fa27763db6d69ad3444e8ba72f947df9e7e163bad7c1f5614f8fd27a311c3
Status: Downloaded newer image for nginx:latest
```

If we have a look to the changes that occurs in the /graph/overlay2 folder

```
$ ls /graph/overlay2
3a2d6ceff88ef5b3881b934028ceef397e49c2b6f798dd07d987f2cab8c8da52
563098ea465d9c35a19534659b3c27e551bdeaa614c2d115b6961e1466f7e52a
backingFsBlockDev
d542bb897041911966b77dfd888ac85ebdd51fe18e2bfba5a0ff8ab75e480f18
l
```

Some folders, with names that looks like hash, were created. Those are the layers which, merged together, build the image filesystem.

Let’s run a container based on nginx.

```
$ docker container run -d nginx
10a09265291da1200df6665125b2a13806e7387965da1d7238420ca0fa52f32b
```

We can now see 2 additional folders (ID, ID-init), those ones correspond to the read-write layer of the running container.

```
$ ls /graph/overlay2
1ce6e1ad78ff4559a33c9567821045aab0d2001a9f443b8fd608c13228e4c781
1ce6e1ad78ff4559a33c9567821045aab0d2001a9f443b8fd608c13228e4c781-init
3a2d6ceff88ef5b3881b934028ceef397e49c2b6f798dd07d987f2cab8c8da52
563098ea465d9c35a19534659b3c27e551bdeaa614c2d115b6961e1466f7e52a
backingFsBlockDev
d542bb897041911966b77dfd888ac85ebdd51fe18e2bfba5a0ff8ab75e480f18
l
```

Feel free to go into those folder and explore their filesystems.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
10a09265291d        nginx               "nginx -g 'daemon ..."   4 minutes ago       Up 4 minutes        80/tcp              quirky_sinoussi

$ docker stop 10a09265291d
10a09265291d

$ docker rm 10a09265291d
10a09265291d

$ ls /graph/overlay2
3a2d6ceff88ef5b3881b934028ceef397e49c2b6f798dd07d987f2cab8c8da52
563098ea465d9c35a19534659b3c27e551bdeaa614c2d115b6961e1466f7e52a
backingFsBlockDev
d542bb897041911966b77dfd888ac85ebdd51fe18e2bfba5a0ff8ab75e480f18
l
```
