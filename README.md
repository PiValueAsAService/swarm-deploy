# Docker swarm deploy

## Собрать локально образы

```
docker build -t "$(npm pkg get name | tr -d '"'):$(npm pkg get version | tr -d '"')" .
```

Запустить образы локально

```
docker run --rm -p 1337:1337 -e SERVER__LISTEN__HOST=0.0.0.0 -d limiter-proxy-service:v1.0.0
docker run --rm -p 1338:1338 -e SERVER__LISTEN__HOST=0.0.0.0 -d compute-service:v1.0.0
```


## Запускаем swarm

https://k21academy.com/docker-kubernetes/docker-swarm/

https://hub.docker.com/_/docker/

https://habr.com/ru/articles/659813/

```
docker run --privileged -d --name pivaas-swarm-node1 docker:24.0.7-dind
docker run --privileged -d --name pivaas-swarm-node2 docker:24.0.7-dind
docker run --privileged -d --name pivaas-swarm-node3 docker:24.0.7-dind

docker exec -it pivaas-swarm-node1 /bin/sh
docker exec -it pivaas-swarm-node2 /bin/sh
docker exec -it pivaas-swarm-node3 /bin/sh

docker swarm init --advertise-addr 172.17.0.2
docker swarm join --token SWMTKN-1-1u3wf95f1zvvizobmf3frhmad4gruhfa2o9pxg0fu8ishgkry5-b3r5ca5nsij8pd7abtfy5ujzo 172.17.0.2:2377

docker node ls
```


## Запускаем приложение

apk add nano curl

nano docker-compose.demo.yaml

```yaml
version: "3.9"

services:
  limiter-proxy-service:
    image: arobingood/pivaas-limiter-proxy-service:v1.0.1
    ports:
      - "1337:1337"
    environment:
      SERVER__LISTEN__HOST: "0.0.0.0"
      COMPUTE_SERVICE__PROTOCOL: "http"
      COMPUTE_SERVICE__HOSTNAME: "compute-service"
      COMPUTE_SERVICE__PORT: "1338"
    deploy:
      replicas: 1
    healthcheck:
      test: curl -sS http://0.0.0.0:1337/health || echo 1
      interval: 5s
      timeout: 1s
      retries: 3

  compute-service:
    image: arobingood/pivaas-compute-service:v1.0.3
    ports:
      - "1338:1338"
    environment:
      SERVER__LISTEN__HOST: "0.0.0.0"
    deploy:
      replicas: 3
    healthcheck:
      test: curl -sS http://0.0.0.0:1338/health || echo 1
      interval: 5s
      timeout: 1s
      retries: 3
```

```
docker stack deploy -c ./docker-compose.demo.yaml demo

docker stack ls

docker stack services demo

docker service ps demo_compute-service

docker service scale demo_limiter-proxy-service=2 demo_compute-service=6
```
