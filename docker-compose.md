---
title: Docker Compose
theme: beige
---




## Docker Compose

- First released in 2014 as standalone tool for defining and running multi-container Docker applications
- Later became a Docker plugin
- Now, it's shipped with Docker Desktop by default

Note: You may not know it, but you may already have it.

---

## What's the big deal?

- WordPress as an example
- 2 containers: app-server and database(mysql)
- 1 network
- 1 persistent volume


Note: What is defining and running multi-container Docker applications?  
- Instead of explaining what it is, I'll show you by example.
- WordPress: popular website/webpage creating tool
- Containers run in isolation

---

### With Docker

```bash [1-3|4-12|13-20|21-26]

docker network create -d bridge wordpress-net
docker volume create db_data
  
docker run -d --name db --network wordpress-net \
    -v db_data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=somewordpress \
    -e MYSQL_DATABASE=wordpress \
    -e MYSQL_USER=wordpress \
    -e MYSQL_PASSWORD=wordpress \
    mysql

docker run -d --name wordpress --network wordpress-net \
    -p 80:80 \
    -e WORDPRESS_DB_HOST=db \
    -e WORDPRESS_DB_USER=wordpress \
    -e WORDPRESS_DB_PASSWORD=wordpress \
    -e WORDPRESS_DB_NAME=wordpress \
    wordpress 

# To shut down
docker stop wordpress db
docker rm wordpress db

docker network rm wordpress-net
docker volume rm db_data

```

---

### With Docker Compose

Define everything in compose.yml
```YAML [1-11|12-20|21-23]
services:
  db:
    image: mysql
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress

  wordpress:
    image: wordpress
    ports:
      - 80:80
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress

  volumes:
    db_data:
```

Note: This is the "define" part of docker compose.

---

Simple commands for doing things
```bash [1-2|3-6|7-9]
docker compose up [-d]
# take care of container, network, and volume

docker compose down [-v] [--rmi all]
# stop and remove container, network, 
# and optionally volume or images

docker compose pull
# to update your app to a later version
```

---

### Compose is more than just for runtime

- Define parameters for building the image
  - registry, org, repos, and tag
- Used for `push` (publish)
- Define parameters for building the application
  - build arguments
  - secrets, platform, context, etc
- Dev related tasks
  - testing
  - debugging

Note: registry/org/repo:tag

---

<!-- .element: class="fragment" -->

### IRSAViewer
- Take IRSAViewer as an example
- Uses Firefly multi-stage builds
- Handles building, testing, running the application
- DevMode: Development Mode
  - Auto build/deploy when source code changes


**Stage graph**  <br>
  deps &rarr; builder &rarr; final   <br>
  &#8627; dev_mode

<!-- .element: class="fragment" -->

Note: Instead of going over all of the features available, I'll show 
you how we're using it.

---

### IRSAViewer compose file


```YAML [3-4|5-9|13-21|16,17|23-30|24-26|27-29]
version: '3.4'
services:
  irsaviewer:
    image: ipac/irsa-irsaviewer:${BUILD_TAG:-latest}
    build:
      dockerfile: firefly/docker/Dockerfile
      args:
        build_dir: irsa-ife
        target: irsaviewer:war
    ports:
      - "8080:8080"

  test:
    build:
      dockerfile: firefly/docker/Dockerfile
      target: deps
    command: bash -c "cd irsa-ife && gradle test"
    volumes: &volInfo
      - ../firefly_test_data:/opt/work/firefly_test_data
      - ../firefly:/opt/work/firefly
      - ../irsa-ife:/opt/work/irsa-ife

  devMode:
    build:
      dockerfile: firefly/docker/Dockerfile
      target: dev_mode
    ports: &devPorts
      - "8080:8080"
      - "5050:5050"
    volumes: *volInfo
```
---

### Dockerized DEV environment

```bash
docker compose up irsaviewer [-d]
# build and deploy irsaviewer locally on port 8080

docker compose run test
# runs test then exit 

docker compose up devMode
# build and deploy irsaviewer locally, then enter DevMode
```
___

(Side Track) &darr;


----

### Benefits of Dev Environment as code

- Dev env is defined in Dockerfile and commit with source code
- Build from previous commits
- Avoid maintain lengthy, mostly outdated setup documents

Note: Have you tried going back to fix something on an old branch,
after you've updated your dev env?  Nightmare, I know.
Not to mention build script and Jenkins setup.  Anyways, let's move on.

---

### There's more

- Share compose configurations 
  - Chain multiple compose files
  - YAML anchors and extensions 
- Service profile
- Variable substitution

---

##### Wonder how I created this slide using only Markdown?

https://github.com/loitly/grits-2022

___

## Thank You

