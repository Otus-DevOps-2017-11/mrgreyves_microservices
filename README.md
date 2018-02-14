Table of Contents
=================

   * [Table of Contents](#table-of-contents)
      * [Homework-19 docker-6](#homework-19-docker-6)
         * [Основное задание](#Основное-задание)
         * [Задание со * 1](#Задание-со--1)
         * [Задание со * 2](#Задание-со--2)
      * [Homework-17 docker-4](#homework-17-docker-4)
         * [Основное задание](#Основное-задание-1)
         * [Задание со *](#Задание-со-)
      * [Homework-16 docker-3](#homework-16-docker-3)
         * [Основное задание](#Основное-задание-2)
         * [Задание со * 1](#Задание-со--1-1)
         * [Задание со * 2](#Задание-со--2-1)
         * [Задание со * 3](#Задание-со--3)
      * [Homework-15 docker-2](#homework-15-docker-2)
         * [Основное задание](#Основное-задание-3)
      * [Homework-14 docker-1](#homework-14-docker-1)
         * [Основное задание](#Основное-задание-4)
         * [Задание со *](#Задание-со--4)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Homework-19 docker-6
### Основное задание

Выполнил установку инстанса для gitlab-ci вручную. Во время установки при помощи  
docker-machine, как в предыдущем ДЗ, все время высыпались разного рода ошибки.  
Так же заметил что инстанс с 2мя ядрами работает на порядок быстрее.

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-compose
sudo mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs
cd /srv/gitlab/
sudo touch docker-compose.yml
```

Docker-compose файл для нашего gitlab:
```
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.example.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://<YOUR-VM-IP>'
  ports:
    - '80:80'
    - '443:443'
    - '2222:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
```
Запускаем gitlab:
```
docker-compose up -d
```

Необходимо немного подождать так как gitlab запускается не мгновенно.
Во время переустановки инстанса столкнулся с моментом когда удаленный репозиторий  
уже указан, но у него не верный ip адрес.  
```
#просмотр всех  remote
git remote -v
#удаление  remote
git remote remove remote_name

```

Был запущен контейнер с runner:
```
sudo docker run -d --name gitlab-runner --restart always \
   -v /srv/gitlab-runner/config:/etc/gitlab-runner \
   -v /var/run/docker.sock:/var/run/docker.sock \
   gitlab/gitlab-runner:latest
```

Регистрируем runner:
```
sudo docker exec -it gitlab-runner gitlab-runner register
```
В интерактивном режиме настраиваем runner.

### Задание со * 1
Была найден [мануал](https://github.com/sameersbn/docker-gitlab-ci-multi-runner#getting-started).
Так же вариант подойдет для docker-compose.
Соответственно мы можем запускать дополнительные runner:
```
sudo docker run -d --name gitlab-runner-2 --restart always \
   -v /srv/gitlab-runner/config:/etc/gitlab-runner \
   -v /var/run/docker.sock:/var/run/docker.sock \
   gitlab/gitlab-runner:latest
```
Настройка runner:
```
sudo docker exec -it gitlab-runner-2 gitlab-runner register --non-interactive \
 --description my-runner-2 \
 --url http://GIT-LAB-IP \
 --registration-token REG-TOKEN \
 --executor docker \
 --run-untagged \
 --locked=false \
 --docker-image alpine:latest 
```
Настройка производится не в интерактивном режиме при помощи ключа --non-interactive.
В итоге мы можем при помощи скрипта запустить и настроить несколько runner.

### Задание со * 2
Настроена интеграция со [slack.](https://devops-team-otus.slack.com/messages/C8CG4B8RH)
Для того что бы приходили уведомления в slack не только при не удачных проходах  
пайплайна нужно не ставить в Notify only broken pipelines.




## Homework-17 docker-4
### Основное задание

В данном задании задании нами были протестированы сетевые возможности docker.  
После запуска контейнера joffotron/docker-net-tools и запуска команды ifconfig  
мы увидели что контейнеру доступен только loopback интерфейс. После запуска его в хостовой  
сети мы увидели что контейнер теперь имеет доступ к сети хоста.  
Проверили что будет если несколько раз запустить контейнер с nginx привязанный к  
хостой сети:
```
docker run --network host -d nginx
```
Контейнер запускается первый раз и занимает 80 порт, запуск дополнительных контейнеров  
не возможен так как сокет уже используется.  

Когда мы запускаем контейнер в хостовой сети, новый сетевой namespace не создается.  
Если же запускает с драйвером none то создается новый сетевой namespace.  

Был создан наш docker-compose.yml в котором описано наше приложение. 
Так же был создан файл .env в котором мы указали переменные окружения.
```
COMPOSE_PROJECT_NAME=reddit
USERNAME=mrgreyves
UI_VER=1.0
POST_VER=1.0
COMMENT_VER=1.0
EXT_PORT=9292
INT_PORT=9292

```
### Задание со *

Имя проекта задается при помощи переменно окружения COMPOSE_PROJECT_NAME.
Был docker-compose.override.yml в которм мы указавали переменные для разных частей нашего проекта. 
[Официальная документация.](https://docs.docker.com/compose/extends/#example-use-case)
```
version: '3.3'
services:

  ui:
    command: ["puma", "--debug", "-w", "2"]

  comment:
    command: ["puma", "--debug", "-w", "2"]

```
Для сервиса puma использовались паметры --debug и -w 2.


## Homework-16 docker-3

### Основное задание

Во время выполнения денного ДЗ мы создали образы для нашей микросервисной архитектуры.  
Были созданы образы для comment, post, ui.  
Во время собрки ui на docker-machine возникла ошибка, образ так и не собирался.  
Пришлось пересоздать docker-machine использую в качестве шаблона g1-small вместо f1-micro (больше  
оперативной памяти).

### Задание со * 1

Была создана новая сеть:
```
docker create network reddit_hw
```
Соответственно было созданы новые алиасы для наших контейнером, при это пришлось передавать  
переменные окружения непосредственно в контейнер при помощи ключа -е:
```
docker run -d --network=reddit_hw --network-alias=post_db_hw --network-alias=comment_db_hw mongo:latest
docker run -d --network=reddit_hw --network-alias=post_hw -e "POST_DATABASE_HOST=post_db_hw" mrgreyves/post:1.0
docker run -d --network=reddit_hw --network-alias=comment_hw -e "COMMENT_DATABASE_HOST=comment_db_hw" mrgreyves/comment:1.0
docker run -d --network=reddit_hw -p 9292:9292 -e "POST_SERVICE_HOST=post_hw" -e "COMMENT_SERVICE_HOST=comment_hw" mrgreyves/ui:1.0
```
После запуска наш сервис стартанул штатно.

Был создан volume который был подключен к контейнеру с mongodb для хранения наших постов:
```
docker volume create reddit_db
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
```
Для просмотра volume используем комманду:  
```
docker volume ls
```

### Задание со * 2

Был собран образ на основе alpine linux. Для установки приложений и обновления кеша  
необходимо использовать комманду:
```
apk add --update
```

Для того что бы не переделывать файл Docker был создан файл Docker_small. Для того что  
бы указать docker какой файл использовать для билда используем ключ -f:
```
docker build -t mrgreyves/ui:2.1 -f ./ui/Docker_small ./ui
```


### Задание со * 3

Вместо ADD был использован COPY:
```
COPY . $APP_HOME
```
Был сгруппирован блок RUN:  
```
RUN apk add --no-cache build-base ruby ruby-json ruby-dev ruby-bundler && \
    gem install bundler --no-ri --no-rdoc && bundle install && apk del build-base ruby-dev
```

Образ соответственно еще уменьшился.

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
