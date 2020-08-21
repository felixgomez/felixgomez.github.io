---
title: Apuntes para Docker
categories:
- Apuntes
- Docker
- Devops
tags: 
    - Docker
    - docker-compose
thumbnail: /images/docker-logo.png
date: 2018-01-23
---

```bash
# Build 
$ docker-compose build

# Run containers in detached mode
$ docker-compose up -d

# Stop containers
$ docker-compose stop

# Bash commands
$ docker-compose exec <container_id> bash

# Check CPU consumption
$ docker stats $(docker inspect -f "{{ .Name }}" $(docker ps -q))

# Delete all containers
$ docker rm $(docker ps -aq)

# Delete all images
$ docker rmi $(docker images -q)
```
