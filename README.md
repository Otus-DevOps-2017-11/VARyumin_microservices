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

## Homework 16

#### 1. Новая структура приложения
1) Все файлы предыдущих домашних заданий (14 и 15) перенесены в каталог monolith
2) Домашнее задание 16 находится в каталоге reddit-microservices. Приложение разбито на 3 каталога:

      - post-py - сервис отвечающий за написание постов
      - comment - сервис отвечающий за написание комментариев
      - ui - веб-интерфейс, работающий с другими сервисами
      - Отдельно создан контейнер с БД на базе MongoDB
3) Установить linter не удалось на CentOs 7, поэтому воспользовался сайтом: https://www.fromlatest.io
#### 2. Dockerfile сервисов
1) Сервис post-py
```Docker
FROM python:3.6.0-alpine
RUN pip install flask pymongo

WORKDIR /app
COPY . /app

RUN pip install -r /app/requirements.txt

ENV POST_DATABASE_HOST=post_db \
    POST_DATABASE=posts

CMD ["python3", "post_app.py"]
```
Заменено использование ADD на COPY. ENV записано в одну строку. Больше файл изменений не притерпел.
Онлайн сервис linter предложений по улучшению не показал.

2) Сервис comment
```Docker
FROM ruby:2.2
RUN apt-get update -qq && \
    apt-get install -y --no-install-recommends build-essential &&\
    rm -rf /var/lib/apt/lists/*

ENV APP_HOME /app
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

COPY Gemfile* $APP_HOME/
RUN bundle install
COPY . $APP_HOME

ENV COMMENT_DATABASE_HOST=comment_db
ENV COMMENT_DATABASE=comments

CMD ["puma"]
```
Заменено использование ADD на COPY. Онлайн linter предложил удалять кеш rm -rf /var/lib/apt/lists/* и использовать конструкцию --no-install-recommends.
Эти изменения повелкли к изменению размера образа на 10 МБ.
Без рекомендаций linter:
```bash
[kirill@localhost reddit-microservices]$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
varyumin/comment   1.0                 ae9e46b04522        16 minutes ago      771MB
```
После применения рекомендация linter:
```bash
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
varyumin/comment   1.0                 bd415ce462db        9 minutes ago       761MB
```
3) Сервис UI
```Docker
FROM ruby:2.2
RUN apt-get update -qq && \
    apt-get install -y --no-install-recommends build-essential && \
    rm -rf /var/lib/apt/lists/*

ENV APP_HOME /app
RUN mkdir $APP_HOME
WORKDIR $APP_HOME
COPY Gemfile* $APP_HOME/

RUN bundle install
COPY . $APP_HOME
ENV POST_SERVICE_HOST=post
ENV POST_SERVICE_PORT=5000
ENV COMMENT_SERVICE_HOST=comment
ENV COMMENT_SERVICE_PORT=9292
CMD ["puma"]
```
Заменено использование ADD на COPY. Онлайн linter предложил удалять кеш rm -rf /var/lib/apt/lists/* и использовать конструкцию --no-install-recommends.
Эти изменения повелкли к изменению размера образа на 11 МБ.
Без рекомендаций linter:
```bash
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
varyumin/ui        1.0                 c4e566f81f5a        15 minutes ago      779MB
```
После применения рекомендация linter:
```bash
REPOSITORY                  TAG                 IMAGE ID            CREATED              SIZE
varyumin/ui        1.0                 93c6eb4456a2        About a minute ago   768MB
```
#### 3. Сборка приложений
Команды для сборки приложений:
```bash

docker pull mongo:latest
docker build -t varyumin/post:1.0 ./post-py
docker build -t varyumin/comment:1.0 ./comment
docker build -t varyumin/ui:1.0 ./ui
```
Сборка приложения UI началалось не с первого пункта, так как docker уже успел закешировать несколько слоев при выполнении comment.

#### 4. Запуск приложений
1) Создадим специальную сеть для приложения
```bash
docker network create reddit
```
Создали bridge-сеть для контейнеров, так как сетевые алиасы не работают в сети по умолчанию.

2) Запустим контейнеры в созданные сети с алиасами к контейнерам. Команды для запуска контейнеров:
```bash
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post varyumin/post:1.0
docker run -d --network=reddit --network-alias=comment varyumin/comment:1.0
docker run -d --network=reddit -p 9292:9292 varyumin/ui:1.0
```
#### 5. Задание со звездочкой
Конмады для запусков контейнеров с новыми сетевыми алиасами. Переменные окружения задаются в команде:
```bash
docker run -d --network=reddit --network-alias=post_db_mydocker --network-alias=comment_db_mydocker mongo:latest
docker run -d --network=reddit --network-alias=post_mydocker -e POST_DATABASE_HOST=post_db_mydocker varyumin/post:1.0
docker run -d --network=reddit --network-alias=comment_mydocker -e COMMENT_DATABASE_HOST=comment_db_mydocker varyumin/comment:1.0
docker run -d --network=reddit -p 9292:9292 -e POST_SERVICE_HOST=post_mydocker -e COMMENT_SERVICE_HOST=comment_mydocker varyumin/ui:1.0
```
#### 6. Улучшение образа UI
Собран образ на базе Ubuntu - версия 2. Так же применены все рекомендации linter. Образ на основе ruby переименрован в Dockerfile_ruby.
Результат:
```bush
[kirill@localhost reddit-microservices]$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
varyumin/ui        2.0                 6e4896f7e84d        21 seconds ago      394MB
varyumin/ui        1.0                 93c6eb4456a2        45 minutes ago      768MB
```
Образ уменьшился почти в 2 раза - 48,7%
#### 7. Задание со звездочкой. Образ на основе Alpine Linux
Собран образ на базе Apline Linix - версия 3. Так же применены все рекомендации linter. Образ на основе Ubuntu переименрован в Dockerfile_ubuntu.
Результат:
docker_images_size
![Images Size](.pic/docker_images_size.jpg)

Применение рекомендация linter ведет к снижению замнаиемого образа, так как чистится кеш.

Рекомендации от linter позволяют сделать вывод, что удаление временных файлов, кешей и другого мусора так же позволяют уменьшить образ.
Уменьшение количества слоев так же ведет к меньшению образа.

#### 8. Docker Volume
1) Создадим docker volume
```bas
docker volume create reddit_db
```
2) Запустим контейнеры, используя volume
```bash
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
docker run -d --network=reddit --network-alias=post varyumin/post:1.0
docker run -d --network=reddit --network-alias=comment varyumin/comment:1.0
docker run -d --network=reddit -p 9292:9292 varyumin/ui:2.0
```
Запуск контейнеров с volume позволило удалять и запускать контейнер без потери данных в базе данных.



## Homework 17

### Подключаемся к ранее созданному docker host’у

> docker-machine ls

```markdown
NAME          ACTIVE   DRIVER   STATE     URL                        SWARM   DOCKER        ERRORS
docker-host   *        google   Running   tcp://35.205.246.70:2376           v18.02.0-ce  
```

> eval $(docker-machine env docker-host)

## Работа с сетью в Docker

### None network driver

1. Запустим контейнер с использованием none-драйвера, с временем жизни 100 секунд, по истечении автоматически удаляется. В качестве образа используем joffotron/docker-net-tools, в него входят утилиты bind-tools, net-tools и curl.

```bash
docker run --network none --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
docker exec -ti net_test ifconfig
```

```markdown
В результате, видим:
• что внутри контейнера из сетевых интерфейсов существует только loopback.
• сетевой стек самого контейнера работает (ping localhost), но без возможности контактировать с внешним миром.
• Значит, можно даже запускать сетевые сервисы внутри такого контейнера, но лишь для локальных экспериментов
```


2. Запустили контейнер в сетевом пространстве docker-хоста

```bash
docker run --network host --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
```

3. Вывод команды docker exec -ti net_test ifconfig

Вывод команды docker exec -ti net_test ifconfig

Видим, что значения совпадают.

4. Запустили docker run --network host -d nginx несколько раз, а контейнер всё равно один запущен.

5. На docker-host машине выполнили команду:

```bash
> sudo ln -s /var/run/docker/netns /var/run/netns

```
Теперь можно просматривать существующие неймспейсы с помощью

```bash
> sudo ip netns
```

#### Примечание: ip netns exec <namespace> <command> - позволит выполнять команды в выбранном namespace

## Bridge network driver

##### 6. Создаём brige сеть в  docker

```bash
docker network create reddit --driver bridge
```
##### 7. Запускаем наш проект reddit с использованием brige-сети:

```bash
> docker run -d --network=reddit mongo:latest
> docker run -d --network=reddit  varyumin/post:1.0
> docker run -d --network=reddit  varyumin/comment:1.0
> docker run -d --network=reddit -p 9292:9292  varyumin/ui:1.0
```

Наши сервисы ищут друг друга по ДНС именам.

##### 8. Присваиваем контейнерам имена или алиасы:

```bash
> docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
> docker run -d --network=reddit --network-alias=post  varyumin/post:1.0
> docker run -d --network=reddit --network-alias=comment  varyumin/comment:1.0
> docker run -d --network=reddit -p 9292:9292  varyumin/ui:1.0
```

##### 9. Убиваем все докер контейнеры

> docker kill $(docker ps -q)

##### 10. Создаём докер сети для фронтэндв и бекэнда:

> docker network create back_net —subnet=10.0.2.0/24
> docker network create front_net --subnet=10.0.1.0/24

##### 11. Запускаем сети в соответсвующих сетях

> docker run -d --network=front_net -p 9292:9292 --name ui  varyumin/ui:1.0
> docker run -d --network=back_net --name comment  varyumin/comment:1.0
> docker run -d --network=back_net --name post  varyumin/post:1.0
> docker run -d --network=back_net --name mongo_db --network-alias=post_db --network-alias=comment_db mongo:latest

#### Info brige, iptables, docker-proxy
![Images Size](.pic/docker_brige1.jpg)

![Images Size](.pic/docker_brige2.jpg)

![Images Size](.pic/iptables.jpg)

![Images Size](.pic/docker-proxy.jpg)

## Docker-compose

##### 12. Ставим Docker Compose

> pip install docker-compose

##### 13. Создаём файл docker-compose.yml в папке reddit-microservices с конфигурацией 4 наших контейнеров

```docker
version: '3.3'
services:
  post_db:
    image: mongo:3.2
    volumes:
      - post_db:/data/db
    networks:
      - reddit
  ui:
    build: ./ui
    image: ${USERNAME}/ui:1.0
    ports:
      - 9292:9292/tcp
    networks:
      - reddit
  post:
    build: ./post-py
    image: ${USERNAME}/post:1.0
    networks:
      - reddit
  comment:
    build: ./comment
    image: ${USERNAME}/comment:1.0
    networks:
      - reddit

volumes:
  post_db:

networks:
  reddit:
```
##### 14. Перед запуском необходимо экспортировать значения данных переменных окружения. В нашем случае это имя пользователя

> export USERNAME= varyumin

Поднимаем наши сервисы
> docker-compose up -d
> docker-compose ps

Наблюдаем такую картину

```markdown
            Name                          Command             State           Ports         
--------------------------------------------------------------------------------------------
redditmicroservices_comment_1   puma                          Up                            
redditmicroservices_post_1      python3 post_app.py           Up                            
redditmicroservices_post_db_1   docker-entrypoint.sh mongod   Up      27017/tcp             
redditmicroservices_ui_1        puma                          Up      0.0.0.0:9292->9292/tcp

```

## Самостоятельное задание

```docker
version: '3.3'
services:
  post_db:
    image: mongo:${VERSION_MONGO}
    volumes:
      - post_db:/data/db
    networks:
      back_net:
        aliases:
          - post_db
          - comment_db
  ui:
    build: ./ui
    image: ${USERNAME}/ui:${APP_VERSION_UI}
    ports:
      - ${APP_PORT}:9292/tcp
    networks:
      - front_net
  post:
    build: ./post-py
    image: ${USERNAME}/post:${APP_VERSION_PTSO}
    networks:
      - front_net
      - back_net
  comment:
    build: ./comment
    image: ${USERNAME}/comment:${APP_VERSION_COMMENT}
    networks:
      - back_net
      - front_net

volumes:
  post_db:

networks:
  front_net:
  back_net:
```

## Homework 19
Разворачиваем инстанс
```
docker-machine create --driver google    --google-project docker-198106 \
  --google-zone europe-west1-b  --google-disk-size 100  --google-machine-type n1-standard-1 \
  --google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri)    gitlab-ci
```
Создаем docker-compose.yml

Настройки в web интерфейсе (группа, проект)

Создаем структуру каталогов на инстансе
mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs

Добавляем наш проект на инстанс gitlab
```
git checkout -b docker-6
git remote add gitlab http://35.205.139.135/homework/example.git
git push gitlab docker-6
```
Добавляем пайплайн, коммитим и отправляем на инстанс
```
git add .gitlab-ci.yml
git commit -m 'add pipeline definition'
git push gitlab docker-6
```
Запускаем runner на инстансе
```
docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
```
Регистрируем runner
```
docker exec -it gitlab-runner gitlab-runner register
```
Скачиваем и добавляем reddit на сервер
```
git clone https://github.com/express42/reddit.git && rm -rf ./reddit/.git
git add reddit/
git commit -m “Add reddit app”
git push gitlab docker-6
```
Настраиваем пайплайн для тестирования, вносим изменения в .gitlab-ci.yml
