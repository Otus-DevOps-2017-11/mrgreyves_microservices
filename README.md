Table of Contents
=================

* [Homework-14 docker-1](#homework-14-docker-1)
  * [Основное задание](#Основное-задание)
  * [Задание со *](#Задание-со-)
  
Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

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
