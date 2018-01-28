Table of Contents
=================

   * [Table of Contents](#table-of-contents)
      * [Homework-15 docker-2](#homework-15-docker-2)
         * [Основное задание](#Основное-задание)
      * [Homework-14 docker-1](#homework-14-docker-1)
         * [Основное задание](#Основное-задание-1)
         * [Задание со *](#Задание-со-)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Homework-15 docker-2

### Основное задание

Создан новый проект в GCP с названием docker.
Gcloud SDK был установлен ранее поэтому сейчас была произведена только его инициализация командой:  
```
gcloud init
```

Настроена аутентификация:  
```
gcloud auth application-default login
```
Создан docker-machine:
```
docker-machine create --driver google \
--google-project docker-181710 \
--google-zone europe-west1-b \
--google-machine-type g1-small \
--google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) \ docker-host
```

```
NAME          ACTIVE   DRIVER   STATE     URL                         SWARM   DOCKER        ERRORS
docker-host   *        google   Running   tcp://ip-address           v18.01.0-ce
```
(По понятным прикинам ip не указываю)

```
docker run --rm -ti tehbilly/htop
```
Htop показывает процессы внутри контейнера

```
docker run --rm --pid host -ti tehbilly/htop
```
Htop показывает процессы на хосте с docker контейнером

Были созданы файлы Dockerfile, db_config, mongod.conf, start.sh
Был создан обран:
```
docker build -t reddit:latest .
```

Так же запущен контейнер на нашем docker-host
```
docker run --name reddit -d --network=host reddit:latest
```

После создания правила с файрволле удалост подключиться к нашему приложению.
Была произведена регистрации и запушин обран на [docker hub](https://hub.docker.com/r/mrgreyves/otus-reddit/).

```
docker tag reddit:latest mrgreyves/otus-reddit:1.0
docker push mrgreyves/otus-reddit:1.0
```

Дополнительные команды:

```
#прибить docker-machine
docker-machine rm name
#удалить убраз
docker rmi image_name
#удалить контейнер
docker rm con_id
```

## Homework-14 docker-1  

### Основное задание
Основное задание было выполнено без каких либо проблем. 
Основные команды:

```

docker run con_id
#запуск контейнера hello-world

docker ps
#список запущенных контейнеров, опция -а покажет и остановленные контейнеры

docker images
#список сохраненных образов

Команда run создает и запускает контейнер из Image
docker run каждый раз запускает новый контейнер
Если не указать флаг —rm при запуске docker run, то после остановки контейнер вместе с содержимым останется на диске

docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.CreatedAt}}\t{{.Names}}" 
#просмотр всех контейнеров в красивой таблице

docker start cont_id
#запускает остановленный (ранее созданный) контейнер

docker attach cont_id
#присоединяет терминал к нужному контейнеру

docker exec -it <u_container_id> bash 
#exec запускает новый процесс (bash) в контейнере

-i – запускает контейнер в foreground режиме (docker attach) 
-d – запускает контейнер в background режиме 
-t создает TTY

docker commit <con_id> yourname/ubuntu-tmp-file 
#создает образ из контейнера 

docker kill
#сразу посылает сигнал sigkill в контейнер

docker stop
#посылает sigterm (остановка приложение) и через 10 с sigkill (наглухо вырубает приложение)

docker system df
#показывает сколько место занято контейнера, образами, volume

docker rm
#удаляет контейнер

docker rmi
#удаляет образ

```

### Задание со *

Выводы по разнице вывода inspect для образа и контейнера в файле docker-1.log
