# VARyumin_microservices

## Homework  14

Использованные в ходе выполнени команды

```
docker version
docker run hello-world - Запуск контейнера из образа
docker ps - Список запущенных контейнеров
docker images - Список образов
docker run -it ubuntu:16.04 /bin/bash
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.CreatedAt}}\t{{.Names}}"
docker start - запуск остановленного контейнера
docker attach - подсоединение терминала к контейнеру
docker exec - запуск нового процесса внутри контейнера
docker commit - создает image из контейнера
docker inspect - low -level information about docker object
docker kill - посылает SIGKILL (безусловное завершение процесса)
docker stop - SIGTERM (остановка приложения)
docker system df - информация о дисковом пространстве (сколько занято образами, контейнерами, volume)
docker rm - удаление контейнера Ex: docker rm $(docker ps -a -q)
docker  rmi - удаление образа Ex: docker rmi $(docker images -q)
```

## Homework 15

1. Создан новый проект в gcloud
2. Создал Docker-хост

```
docker-machine create --driver google \
--google-project docker-198106 \
--google-zone europe-west1-b \
--google-machine-type g1-small \
--google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) \
docker-host
```
3. Добавил 4 файла:
> Dockerfile
> mongod.conf
> db_config
> start.sh

4. Произвёл сборку командой:
> docker build -t reddit:latest .
5. запустил контейнер
> docker run --name reddit -d --network=host reddit:latest
6. Открыл порт 9292
```
gcloud compute firewall-rules create reddit-app \
--allow tcp:9292 --priority=65534 \
--target-tags=docker-machine \
--description="Allow TCP connections" \
--direction=INGRESS
```
7. Зарегистрировался в "Docker hub"
8. Залогинелся в консоли на "Docker hub"
```
docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: varyumin
Password:
Login Succeeded
```
9. Загрузил образ на docker hub
```
docker push varyumin/otus-reddit:1.0
The push refers to repository [docker.io/varyumin/otus-reddit]
01a42ad32823: Pushed
9f1b6153c0ae: Pushed
79468bccfed2: Pushed
571c1cb72f10: Pushed
df7c5b9b9bde: Pushed
779f7eb7e77f: Pushed
b7a823e50c49: Pushed
d94151d9dc6b: Pushed
45620340f8c2: Pushed
db584c622b50: Mounted from library/ubuntu
52a7ea2bb533: Mounted from library/ubuntu
52f389ea437e: Mounted from library/ubuntu
88888b9b1b5b: Mounted from library/ubuntu
a94e0d5a7c40: Mounted from library/ubuntu
1.0: digest: sha256:4e62aaa84a4619c703f63b8819fa1bde6773ca45312d49ffd597eeb412698dbd size: 3241
```
