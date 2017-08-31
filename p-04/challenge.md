### Swarm stack introduction

Let’s deploy the voting app stack on a swarm

### Purpose
The purpose of this lab is to illustrate how to deploy a stack (multi services application) against a Swarm using a docker compose file.

### The application
The voting app is a very handy multi containers application often used for demo purposes during meetup and conferences.

It basically allow users to vote between cat and dog (but could be “space” or “tab” too if you feel like it).

This application is available on Github and updated very frequently when new features are developed.

### Init your swarm
Let’s create a Docker Swarm first

```
[node1](local) root@10.0.13.3 ~
$ docker swarm init --advertise-addr $(hostname -i)
Swarm initialized: current node (ujbrmkbf2ta9pf97qoxjkmy6p) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0ebm5blf5fimh36pgkqaepugw0mo9yjzg4fkgjqk4avkeo0oaz-9at8s4if6ijjvby4jo1fy41sq 10.0.13.3:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

From the output above, copy the join command (watch out for newlines) and paste it in the other terminal.

### Show members of swarm
From the first terminal, check the number of nodes in the swarm (running this command from the second terminal worker will fail as swarm related commands need to be issued against a swarm manager).

```
docker node ls
```

The above command should output 2 nodes, the first one being the manager, and the second one a worker.

```
[node1] (local) root@10.0.70.3 ~
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
ojgqdyr71ub3p0qhrhnz0hgxp *   node1               Ready               Active              Leader
rm6fonwu56paq5nfmu7tvmde4     node2               Ready               Active
```

### Clone the voting-app
Let’s retrieve the voting app code from Github and go into the application folder.

Ensure you are in the first terminal and do the below:

```
git clone https://github.com/docker/example-voting-app
cd example-voting-app
```

### Deploy a stack

```
$ cat docker-stack.yml
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

A stack is a group of services that are deployed together. The docker-stack.yml in the current folder will be used to deploy the voting app as a stack.

Ensure you are in the first terminal and do the below:

```
[node1] (local) root@10.0.2.3 ~/example-voting-app
$ docker stack deploy --compose-file=docker-stack.yml voting_stack
Creating network voting_stack_default
Creating network voting_stack_backend
Creating network voting_stack_frontend
Creating service voting_stack_vote
Creating service voting_stack_result
Creating service voting_stack_worker
Creating service voting_stack_visualizer
Creating service voting_stack_redis
Creating service voting_stack_db
```

Note: been able to create a stack from a docker compose file is a great feature added in Docker 1.13.

Check the stack deployed from the first terminal

```
[node1] (local) root@10.0.2.3 ~/example-voting-app
$ docker stack ls
```

The output should be the following one. It indicates the 6 services of the voting app’s stack (named voting_stack) have been deployed.

```
NAME                SERVICES
voting_stack        6
```

Let’s check the service within the stack

```
$ docker stack services voting_stack
```

The output should be like the following one (your ID should be different though).

```
ID                  NAME                      MODE                REPLICAS            IMAGE                                          PORTS
g9pnpf32i42a        voting_stack_vote         replicated          2/2              dockersamples/examplevotingapp_vote:before     *:5000->80/tcp
j8ah4sieh7ft        voting_stack_db           replicated          1/1              postgres:9.4
kgls9x3y0g28        voting_stack_result       replicated          1/1              dockersamples/examplevotingapp_result:before   *:5001->80/tcp
qykllpn10soo        voting_stack_worker       replicated          1/1              dockersamples/examplevotingapp_worker:latest
wrulzsjy1gxa        voting_stack_visualizer   replicated          1/1              dockersamples/visualizer:stable                *:8080->8080/tcp
wwvw8gat5vjk        voting_stack_redis        replicated          1/1              redis:alpine                                   *:0->6379/tcp
```

Let’s list the tasks of the vote service.

```
docker service ps voting_stack_vote
```

You should get an output like the following one where the 2 tasks (replicas) of the service are listed.

```
ID                  NAME                  IMAGE               NODE                DESIRED STATE       CURRENT STATE       ERROR               PORTS
85n5rztszabt        voting_stack_vote.1   dockersamples/examplevotingapp_vote:before   node2               Running             Running 4 minutes ago
wboj5fg4810k        voting_stack_vote.2   dockersamples/examplevotingapp_vote:before   node1               Running             Running 4 minutes ago
```

From the NODE column, we can see one task is running on each node.

Finally, we can check that our [APP](http://training.play-with-docker.com/) is running, the [RESULT](http://training.play-with-docker.com/) page is also available as well as [SWARM VISUALIZER](http://training.play-with-docker.com/)

### Conclusion
Using only a couple of commands enables to deploy a stack of services on a Docker Swarm using the really great Docker Compose file format.

- What is a stack ?

a multi-service app running on a Swarm

- A stack can:

be deployed from the commandline
can use the compose file format to deploy
can be used to manage services over multiple nodes
