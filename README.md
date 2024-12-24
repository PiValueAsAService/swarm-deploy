# Docker swarm deploy

## Собрать локально образы

```sh
docker build -t "$(npm pkg get name | tr -d '"'):$(npm pkg get version | tr -d '"')" .
```

Запустить образы локально

```sh
docker run --rm -p 1337:1337 -e SERVER__LISTEN__HOST=0.0.0.0 -d limiter-proxy-service:v1.0.0
docker run --rm -p 1338:1338 -e SERVER__LISTEN__HOST=0.0.0.0 -d compute-service:v1.0.4
```


## Запускаем swarm

https://k21academy.com/docker-kubernetes/docker-swarm/

https://hub.docker.com/_/docker/

https://habr.com/ru/articles/659813/

```sh
docker run --privileged -d --name pivaas-swarm-node1 docker:27.3.1-dind
docker run --privileged -d --name pivaas-swarm-node2 docker:27.3.1-dind
docker run --privileged -d --name pivaas-swarm-node3 docker:27.3.1-dind

docker exec -it pivaas-swarm-node1 /bin/sh
docker exec -it pivaas-swarm-node2 /bin/sh
docker exec -it pivaas-swarm-node3 /bin/sh

docker swarm init --advertise-addr 172.17.0.2
docker swarm join --token SWMTKN-1-1u3wf95f1zvvizobmf3frhmad4gruhfa2o9pxg0fu8ishgkry5-b3r5ca5nsij8pd7abtfy5ujzo 172.17.0.2:2377

docker node ls
```


## Запускаем приложение

```sh
docker stack deploy -c ./docker-compose.demo.yaml demo

docker stack ls

docker stack services demo

docker service ps demo_compute-service

docker service scale demo_limiter-proxy-service=2 demo_compute-service=6
docker service scale demo_compute-service=6
```
