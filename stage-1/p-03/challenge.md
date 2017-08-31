### 2.0 Webapps with Docker
Great! So you have now looked at `docker container run`, played with a Docker container and also got the hang of some terminology. Armed with all this knowledge, you are now ready to get to the real stuff — deploying web applications with Docker.

### 2.1 Run a static website in a container

> Note: Code for this section is in this repo in the [static-site directory](https://github.com/docker/labs/tree/master/beginner/static-site).

Let’s start by taking baby-steps. First, we’ll use Docker to run a static website in a container. The website is based on an existing image. We’ll pull a Docker image from Docker Store, run the container, and see how easy it is to set up a web server.

The image that you are going to use is a single-page website that was already created for this demo and is available on the Docker Store as `seqvence/static-site`. You can download and run the image directly in one go using `docker run` as follows.

```
docker container run -d seqvence/static-site
```

So, what happens when you run this command?

Since the image doesn’t exist on your Docker host, the Docker daemon first fetches it from the registry and then runs it as a container.

Now that the server is running, do you see the website? What port is it running on? And more importantly, how do you access the container directly from our host machine?

Actually, you probably won’t be able to answer any of these questions yet! ☺ In this case, the client didn’t tell the Docker Engine to publish any of the ports, so you need to re-run the `docker container run` command to add this instruction.

Let’s re-run the command with some new flags to publish ports and pass your name to the container to customize the message displayed. We’ll use the -d option again to run the container in detached mode.

First, stop the container that you have just launched. In order to do this, we need the container ID.

Since we ran the container in detached mode, we don’t have to launch another terminal to do this. Run `docker container ls` to view the running containers.

```
$ docker container ls
CONTAINER ID        IMAGE                  COMMAND CREATED             STATUS              PORTS               NAMES
ceb774bc772f        seqvence/static-site   "/bin/sh -c 'cd /u..." 12 seconds ago      Up 11 seconds       80/tcp, 443/tcp     dazzling_roentgen
```

Check out the `CONTAINER ID` column. You will need to use this `CONTAINER ID` value, a long sequence of characters, to identify the container you want to stop, and then to remove it. The example below provides the `CONTAINER ID` on our system; you should use the value that you see in your terminal.

```
$ docker container stop ceb774bc772f
$ docker container rm   ceb774bc772f
```
> Note: A cool feature is that you do not need to specify the entire `CONTAINER ID`. You can just specify a few starting characters and if it is unique among all the containers that you have launched, the Docker client will intelligently pick it up.

Now, let’s launch a container in **detached** mode as shown below:
```
$ docker container run --name static-site -e AUTHOR="Your Name" -d-P seqvence/static-site
d8c24deea1e4d055ff955976baeaec6dbd5b833590f03714da4a5dcdf12dea6a
```

In the above command:

- `-d` will create a container with the process detached from our terminal
- `-P` will publish all the exposed container ports to random ports on the Docker host
- `-e` is how you pass environment variables to the container
- `--name` allows you to specify a container name
- `AUTHOR` is the environment variable name and `Your Name` is the value that you can pass
Now you can see the ports by running the `docker container port` command.

```
$ docker container port static-site
443/tcp -> 0.0.0.0:32768
80/tcp -> 0.0.0.0:32769
```

If you were running [Docker for Mac](https://docs.docker.com/docker-for-mac/), [Docker for Windows](https://docs.docker.com/docker-for-windows/), or Docker on Linux, you would see your app at http://[YourDockerIP]:32769.

You can also run a second webserver at the same time, specifying a custom host port mapping to the container’s webserver.

```
$ docker container run --name static-site-2 -e AUTHOR="Your Name" -d -p 8888:80 seqvence/static-site
6f58e9fa67f80b2fa739d98a177b7b3b2dd023a098b42a7d85ac4293be6a0f51
```
You should see your website running [HERE](http://training.play-with-docker.com/)


