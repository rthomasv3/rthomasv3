---
title: Docker Cheat Sheet
date: 2023-10-30
authors:
  - name: rthomasv3
    link: https://github.com/rthomasv3
    image: https://avatars.githubusercontent.com/u/39268276?v=4
tags:
  - Docker
excludeSearch: false
type: docs
weight: 1
---

| Command | Description | Examples |
| --- | --- | --- |
| `docker run` | Run an image in a container. | $ docker run -it ubuntu /bin/bash |
| `docker ps` | List all running containers. | $ docker ps |
| `docker stop` | Stop a running container. | $ docker stop my-container |
| `docker rm` | Remove a stopped container. | $ docker rm my-container |
| `docker rmi` | Remove an image from the Docker Hub registry. | $ docker rmi ubuntu |
| `docker login` | Log in to your Docker Hub account. | $ docker login `<your-username>` `<your-password>` |
| `docker build` | Build a new image from a Dockerfile. | $ docker build -t my-image . |
| `docker commit` | Create a new image based on an existing one and push it to the registry. | $ docker commit my-container >my-image |
| `docker tag` | Tag an image with a different name. | $ docker tag my-image `<new-name>` |
| `docker login` | Log in to your Docker Hub account. | $ docker login `<your-username>` `<your-password>` |
| `docker network create` | Create a new network. | $ docker network create my-network |
| `docker network connect` | Connect a container to a network. | $ docker network connect my-container >my-network |
| `docker network disconnect` | Disconnect a container from a network. | $ docker network disconnect my-container `<my-network>` |
| `docker volume create` | Create a new volume. | $ docker volume create my-volume |
| `docker volume ls` | List all volumes. | $ docker volume ls |
| `docker volume rm` | Remove an existing volume. | $ docker volume rm my-volume |
| `docker run --rm` | Run a container in detached mode and automatically remove it when the process exits. | $ docker run -it --rm ubuntu /bin/bash |
| `docker exec` | Execute a command inside a running container. | $ docker exec -it my-container bash |
| `docker stop` | Stop a running container. | $ docker stop my-container |
| `docker rm` | Remove a stopped container. | $ docker rm my-container |
| `docker rmi` | Remove an image from the Docker Hub registry. | $ docker rmi ubuntu |
| `docker system prune` | Prunes all unused containers, images, and cache. | $ docker system prune |
| `docker container inspect` | Inspect a property of the container. | $ docker container inspect -f '{{.HostConfig.LogConfig}}' `<id>` |
