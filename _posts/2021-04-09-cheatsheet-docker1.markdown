---
layout: post
title:  "Docker CLI Cheatsheet"
date:   2021-04-09 00:00:00 +0300
categories: cheatsheet docker cli
---

The amount of times I had re-Google things when it comes to docker CLI is crazy. Thus, I decide I would place everything "docker CLI" related which I use frequently in one place.

Docker Engine install [link](https://docs.docker.com/engine/install/)

#### Show Running Containers
`docker ps`
#### Show All Containers
`docker ps -a`
#### Start/Stop Containers
`docker stop CONTIANER_ID\CONTAINER NAME`

`docker start CONTIANER_ID\CONTAINER NAME`
#### Show Available Images (on machine)
`docker image ls`
#### Delete Image
`docker image rm IMAGE_ID`
#### Download Docker Image
`docker pull python:3.9.4-alpine3.12`

`docker pull ruby:3.0.1-alpine3.12`
#### Create and Run a Container
``` bash
docker run -it --rm --name pycontainer \
-p 127.0.0.1:5000:5000 python:3.9.4-alpine3.12;
```

* **-it**: keep STDIN open and allocate a pseudo-tty (interactive, see output)
* **--rm**: container will be deleted once it is stopped
* **-p 127.0.0.1:port1:port2** port-forwarding
    * **127.0.0.1**: port forwarded will be listening locally
    * **port1**: host port
    * **port2**: container port
* **--name**: name to give the container, if not provided then a random name will be generated

``` bash
docker run -it --rm --name pycontainer \
-p 5000:5000 python:3.9.4-alpine3.12;
```

Note -p 5000:5000.
**The forwarded port will be exposed on the host, it will NOT be listening locally**

[Docker Run Command Reference](https://docs.docker.com/engine/reference/run/)

#### Volume Mounting

``` bash
docker run --rm \
-p 80:80 -p 443:443 \
--name vcontainer -v /my/host/dir:/container/dir myimage
```

* **-v /my/host/dir:/container/dir** mounting the /my/host/dir on to the containers /container/dir directory, any changes that the container makes inside that dir will be visible and accessible from the host. Moreover, this will ensure that data is retained even after the container itself is deleted.

``` bash
docker run --rm -p 8000:8888 \
-v "$PWD":/home/jovyan/work jupyter/datascience-notebook:python-3.8.8
```

The above will run a jupyter notebook docker container and will mount the current working directory into the directory which the jupyter notebook container is configure to operate from.

[Jupyter Docs](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/running.html)



#### Env Vars

``` bash
docker run --name somecontainer \
--env VAR1=value_a \
--env VAR2=value_b \
-p 127.0.0.1:443:443 python:3.9.4-alpine3.12;
```

#### Create Containers and Run Commands

``` bash
docker run -it --rm -p 80:80 \
--name nginx-test nginx:1.19.3-alpine /bin/ash
```

* **/bin/ash** will be executed and you will an interactive shell session.
* If this was a debian image, we would have probably used `/bin/sh` instead

Note that the nginx server will not run, /bin/ash will override running the server.


``` bash
docker run -it --rm -p 80:80 \
--name nginx-test nginx:1.19.3-alpine /bin/ash -c whoami
```

The `whoami` command will be executed after which you will received the output. Once that is done, the container will be stopped and then deleted (--rm).

#### Detached Mode (Run in Background)

``` bash
docker run -d --rm -p 80:80 \
--name nginx-test nginx:1.19.3-alpine
```

* **-d**: detached mode, run in background

After running the command above, you will have an nginx container, with port 80 forwarded from the host to the container, running in the background. You can see all running containers by running `docker ps`.

#### Attaching to a Container Running in the Background
``` bash
docker attach nginx-test
```

Since by default the nginx container outputs access logs, once you attach to the container you will see access logs in real time in the terminal. **Keep in mind that if you ctrl-c the container will stop**.

More details: [Docker Attach Command Reference](https://docs.docker.com/engine/reference/commandline/attach/)


#### Detaching From a Container After Attaching
Taken from: [stackoverflow](https://stackoverflow.com/questions/25267372/correct-way-to-detach-from-a-container-without-stopping-it)

The following will work if the container was created with the **-it** switch

``` bash
docker run -it -d --rm -p 80:80 \
--name nginx-test nginx:1.19.3-alpine
```

After attaching (`docker attach nginx-test`), press ctrl-p then ctrl-q.

#### Execute Commands in Running Containers

``` bash
docker exec -it nginx-test /bin/ash -c ps -a
```

* **nginx-test**: This is the name of the container

The command will run `ps -a` inside the container and exit. After the commands is done the container will still be running.

Note that you can use the container name or ID.

``` bash
docker exec -it nginx-test /bin/ash
```

The command above will give you an interactive shell session inside the container. Once you are done you can type `exit` to end the interactive session. The container will still be running in the background after you leave the interactive session.