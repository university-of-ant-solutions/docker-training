### 1.0 Running your first container

Now that you have everything setup, it’s time to get our hands dirty. In this section, you are going to run an [Alpine Linux](http://www.alpinelinux.org/) container (a lightweight linux distribution) on your system and get a taste of the `docker container run` command.

To get started, let’s run the following in our terminal:

```
docker image pull alpine
```

The `pull` command fetches the alpine image from the **Docker registry** and saves it in our system. In this case the registry is [Docker Store](https://store.docker.com/). You can change the registry, but that’s a different lab.

You can use the `docker image` command to see a list of all images on your system.

```
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              latest              7328f6f8b418        5 weeks ago         3.97MB
```

### 1.1 Docker Container Run
Great! Let’s now run a Docker **container** based on this image. To do that you are going to use the `docker container run` command.

```
$ docker container run alpine ls -l
total 8
drwxr-xr-x    2 root     root          4096 Jun 25 17:52 bin
drwxr-xr-x    5 root     root           340 Aug  5 09:13 dev
drwxr-xr-x    1 root     root            66 Aug  5 09:13 etc
drwxr-xr-x    2 root     root             6 Jun 25 17:52 home
drwxr-xr-x    5 root     root           234 Jun 25 17:52 lib
drwxr-xr-x    5 root     root            44 Jun 25 17:52 media
drwxr-xr-x    2 root     root             6 Jun 25 17:52 mnt
...
...
```

What happened? Behind the scenes, a lot of stuff happened. When you call `run`, the Docker client finds the image (alpine in this case), creates the container and then runs a command in that container. When you run `docker container run alpine`, you provided a command (`ls -l`), so Docker started the command specified and you saw the listing.

Let’s try something more exciting.

```
$ docker container run alpine echo "hello from alpine"
hello from alpine
```

OK, that’s some actual output. In this case, the Docker client dutifully ran the echo command in our alpine container and then exited it. If you’ve noticed, all of that happened pretty quickly. Imagine booting up a virtual machine, running a command and then killing it. Now you know why they say containers are fast!

Try another command.

```
docker container run alpine /bin/sh
```

Wait, nothing happened! Is that a bug? Well, no. These interactive shells will exit after running any scripted commands, unless they are run in an interactive terminal - so for this example to not exit, you need to `docker container run -it alpine /bin/sh`.

You are now inside the container shell and you can try out a few commands like `ls -l`, `uname -a` and others. Exit out of the container by giving the `exit` command.

Ok, now it’s time to see the `docker container ls` command. The `docker container ls` command shows you all containers that are currently running.

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

Since no containers are running, you see a blank line. Let’s try a more useful variant: `docker container ls -a`

```
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
6038f5ecfd43        alpine              "/bin/sh"                2minutes ago       Exited (0) 2 minutes ago                       brave_shannon
4fb9cb940371        alpine              "/bin/sh"                2minutes ago       Exited (0) 2 minutes ago                       blissful_engelbart
77802250943a        alpine              "/bin/sh"                2minutes ago       Exited (0) 2 minutes ago                       relaxed_lamport
151beb86a150        alpine              "/bin/sh"                2minutes ago       Exited (0) 2 minutes ago                       condescending_bose
```

What you see above is a list of all containers that you ran. Notice that the STATUS column shows that these containers exited a few minutes ago. You’re probably wondering if there is a way to run more than just one command in a container. Let’s try that now:

```
$ docker container run -it alpine /bin/sh
/ # ls
bin    etc    lib    mnt    root   sbin   sys    usr
dev    home   media  proc   run    srv    tmp    var
/ # uname -a
Linux ac6c8b6d5a0f 4.4.0-1026-aws #35-Ubuntu SMP Thu Jul 20 21:59:09 UTC 2017 x86_64 Linux
```

Running the `run` command with the `-it` flags attaches us to an interactive tty in the container. Now you can run as many commands in the container as you want. Take some time to run your favorite commands.

That concludes a whirlwind tour of the `docker container run` command which would most likely be the command you’ll use most often. It makes sense to spend some time getting comfortable with it. To find out more about `run`, use `docker container run --help` to see a list of all flags it supports. As you proceed further, we’ll see a few more variants of `docker container run`.

### 1.2 Terminology

In the last section, you saw a lot of Docker-specific jargon which might be confusing to some. So before you go further, let’s clarify some terminology that is used frequently in the Docker ecosystem.

- *Images* - The file system and configuration of our application which are used to create containers. To find out more about a Docker image, run `docker image inspect alpine`. In the demo above, you used the `docker image pull` command to download the alpine image. When you executed the command `docker container run hello-world`, it also did a `docker image pull behind` the scenes to download the **hello-world** image.
- *Containers* - Running instances of Docker images — containers run the actual applications. A container includes an application and all of its dependencies. It shares the kernel with other containers, and runs as an isolated process in user space on the host OS. You created a container using `docker run` which you did using the alpine image that you downloaded. A list of running containers can be seen using the `docker container ls` command.
- *Docker daemon* - The background service running on the host that manages building, running and distributing Docker containers.
- *Docker client* - The command line tool that allows the user to interact with the Docker daemon.
- *Docker Store* - Store is, among other things, [a registry](https://store.docker.com/) of Docker images. You can think of the registry as a directory of all available Docker images. You’ll be using this later in this tutorial.
