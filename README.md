Table of Contents
=================

   * [Table of Contents](#table-of-contents)
      * [Homework-30 kubernetes-3](#homework-30-kubernetes-3)
      * [Homework-29 kubernetes-2](#homework-29-kubernetes-2)
      * [Homework-28 kubernetes-1](#homework-28-kubernetes-1)
      * [Homework-27 swarm-1](#homework-27-swarm-1)
         * [Основное задание](#Основное-задание)
         * [Задание со *](#Задание-со-)
         * [Задание со ***](#Задание-со--1)
      * [Homework-25 logging-1](#homework-25-logging-1)
         * [Основное задание](#Основное-задание-1)
      * [Homework-22 monitoring-2](#homework-22-monitoring-2)
         * [Основное задание](#Основное-задание-2)
         * [Задание со *](#Задание-со--2)
      * [Homework-21 monitoring-1](#homework-21-monitoring-1)
         * [Основное задание](#Основное-задание-3)
         * [Задание со * 1](#Задание-со--1-1)
         * [Задание со * 2](#Задание-со--2-1)
      * [Homework-20 docker-7](#homework-20-docker-7)
         * [Основное задание](#Основное-задание-4)
      * [Homework-19 docker-6](#homework-19-docker-6)
         * [Основное задание](#Основное-задание-5)
         * [Задание со * 1](#Задание-со--1-2)
         * [Задание со * 2](#Задание-со--2-2)
      * [Homework-17 docker-4](#homework-17-docker-4)
         * [Основное задание](#Основное-задание-6)
         * [Задание со *](#Задание-со--3)
      * [Homework-16 docker-3](#homework-16-docker-3)
         * [Основное задание](#Основное-задание-7)
         * [Задание со * 1](#Задание-со--1-3)
         * [Задание со * 2](#Задание-со--2-3)
         * [Задание со * 3](#Задание-со--3-1)
      * [Homework-15 docker-2](#homework-15-docker-2)
         * [Основное задание](#Основное-задание-8)
      * [Homework-14 docker-1](#homework-14-docker-1)
         * [Основное задание](#Основное-задание-9)
         * [Задание со *](#Задание-со--4)

Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)

## Homework-30 kubernetes-3

Был настроен Loadbalancer для нашего сервиса ui. Сервис был успешно доступен.
После этого был настроен сервис Ingrees. Ingrees конфигурировался у меня до 10.
Был создал самоподписной сертификат и загружен в кластер:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=IP"
kubectl create secret tls ui-ingress --key tls.key --cert tls.crt -n dev
```

Протокл http автоматически не удалился, пришлось пересоздать Ingrees. (Ушло порядка 10 минут на пересоздание)

Был создан диск:

```
gcloud compute disks create --size=25GB --zone=europe-west1-b reddit-mongo-disk
```

```
 kubectl get persistentvolume -n dev
 
 NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS   REASON    AGE
pvc-277cf72e-35df-11e8-af86-42010a840fcd   25Gi       RWO            Delete           Bound       dev/mongo-pvc           standard                 6m
pvc-b473398b-35df-11e8-af86-42010a840fcd   10Gi       RWO            Delete           Bound       dev/mongo-pvc-dynamic   fast                     2m
reddit-mongo-disk                          25Gi       RWO            Retain           Available                                                    7m
```

Был создан новый тип диска SSD и для создан свой Claim

## Homework-29 kubernetes-2

Для установки minikube используем комманду:  
```
brew cask install minikube
```

```
#запускаем локальный кластер, по умолчанию драйвер для virtualbox
minikube start

#просмотр нод кластера
kubectl get nodes

#просмотр текущего контекста
kubectl config current-context

#просмотр всех контекстов
kubectl config get-contexts
```

Для приложений ui, post, mongo, comment было созданы deployment.yml файлы с  
описанием их состояний. Для связки приложений между собой были созданы файлы  
service.yml

```
#запуск компонента ui
kubectl apply -f ui-deployment.yml

#просмотр всех сервисов
minikube services list

```

Был поднят кластер kubernetes в GCP, созданы правила firewall после чего приложение было  
задеплоено в этот кластер. [Скриншот](https://cloud.mail.ru/public/PGGm/ixDTbSzVb)

Так же был настроен дашборд kubernetes, создан аккаунт.




## Homework-28 kubernetes-1

Согласно данному [туториалу](https://github.com/kelseyhightower/kubernetes-the-hard-way) был развернут класте kubernetes.  
По пунктам:

1. Prerequisites

Проверяем версию gcloud и при необходимости обновляемся:  
```
Google Cloud SDK 193.0.0
bq 2.0.30
core 2018.03.09
gsutil 4.28
```

```
Vladimir:kubernetes vladimir$ gcloud config get-value compute/region
Your active configuration is: [docker]
europe-west1-b
Vladimir:kubernetes vladimird$ gcloud config get-value compute/zone
Your active configuration is: [docker]
europe-west1
```

2. Installing the Client Tools

Устанавливаем cfssl и cfssljson

```
Vladimir:kubernetes vladimir$ cfssl version
Version: 1.3.0
Revision: dev
Runtime: go1.10
```
Для Mac OS лучше использовать brew:
```
brew install cfssl
```

3. Provisioning Compute Resources

Создаем сеть:
```
gcloud compute networks create kubernetes-the-hard-way --subnet-mode custom
```
Создаем подсеть:
```
gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24
```
Создаем правила файрволла:
```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
  
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0
  
Vladimir:kubernetes vladimir$ gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"
NAME                                         NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY
kubernetes-the-hard-way-allow-external       kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp
kubernetes-the-hard-way-allow-internal       kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp
kubernetes-the-hard-way-allow-nginx-service  kubernetes-the-hard-way  INGRESS    1000      tcp:32038
```

Создаем инстансы (контроллеры и воркеры):

```
Vladimir:kubernetes vladimir$ gcloud compute instances list
NAME          ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
controller-0  europe-west1-b  n1-standard-1               10.240.0.10  35.195.108.247  RUNNING
controller-1  europe-west1-b  n1-standard-1               10.240.0.11  35.189.216.245  RUNNING
controller-2  europe-west1-b  n1-standard-1               10.240.0.12  35.205.23.228   RUNNING
worker-0      europe-west1-b  n1-standard-1               10.240.0.20  35.187.125.31   RUNNING
worker-1      europe-west1-b  n1-standard-1               10.240.0.21  104.199.43.161  RUNNING
worker-2      europe-west1-b  n1-standard-1               10.240.0.22  35.195.231.154  RUNNING
```
4 - 6. Создаем сертификаты и конфиги, ничего сложного и интересно.  

7. etcd Cluster

Создаем кластер etcd. Проверяем что все ноды работают:  

```
vladimir@controller-0:~$ ETCDCTL_API=3 etcdctl member list
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379
```
8. Load Balancer

Создаем балансировщик для kubernetes:  

```
Vladimir:kubernetes vladimir$ curl --cacert ca.pem https://${KUBERNETES_PUBLIC_ADDRESS}:6443/version

{
  "major": "1",
  "minor": "9",
  "gitVersion": "v1.9.0",
  "gitCommit": "925c127ec6b946659ad0fd596fa959be43f0cc05",
  "gitTreeState": "clean",
  "buildDate": "2017-12-15T20:55:30Z",
  "goVersion": "go1.9.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```
9. Worker Nodes

```
Vladimir:kubernetes vladimir$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
worker-0   Ready     <none>    52m       v1.9.0
worker-1   Ready     <none>    49m       v1.9.0
worker-2   Ready     <none>    46m       v1.9.0
```

10. Configuring kubectl for Remote Access

```
MacBook-Pro-Vladimir:kubernetes vladimirdrozdeckij$ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-2               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
MacBook-Pro-Vladimir:kubernetes vladimirdrozdeckij$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
worker-0   Ready     <none>    54m       v1.9.0
worker-1   Ready     <none>    50m       v1.9.0
worker-2   Ready     <none>    47m       v1.9.0
```
11. Provisioning Pod Network Routes

```
Vladimir:kubernetes vladimird$ gcloud compute routes list --filter "network: kubernetes-the-hard-way"
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP                  PRIORITY
default-route-048170f7e9f72122  kubernetes-the-hard-way  10.240.0.0/24                            1000
default-route-06b8f6cb1082999d  kubernetes-the-hard-way  0.0.0.0/0      default-internet-gateway  1000
kubernetes-route-10-200-0-0-24  kubernetes-the-hard-way  10.200.0.0/24  10.240.0.20               1000
kubernetes-route-10-200-1-0-24  kubernetes-the-hard-way  10.200.1.0/24  10.240.0.21               1000
kubernetes-route-10-200-2-0-24  kubernetes-the-hard-way  10.200.2.0/24  10.240.0.22               1000
```
12. DNS

```
Vladimir:kubernetes vladimird$ kubectl create -f https://storage.googleapis.com/kubernetes-the-hard-way/kube-dns.yaml

service "kube-dns" created
serviceaccount "kube-dns" created
configmap "kube-dns" created
deployment "kube-dns" created

Vladimir:kubernetes vladimir$ kubectl get pods -l k8s-app=kube-dns -n kube-system
NAME                        READY     STATUS    RESTARTS   AGE
kube-dns-6c857864fb-6qwk9   3/3       Running   0          44m
```

13. Smoke Test

```
Vladimir:kubernetes vladimir$ kubectl get pods -l run=nginx
NAME                   READY     STATUS    RESTARTS   AGE
nginx-8586cf59-qvk55   1/1       Running   0          40m

Vladimir:kubernetes vladimir$ kubectl logs $POD_NAME
127.0.0.1 - - [18/Mar/2018:19:20:17 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.54.0" "-"
10.240.0.20 - - [18/Mar/2018:19:21:50 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.54.0" "-"

Vladimir:kubernetes vladimir$ kubectl exec -ti $POD_NAME -- nginx -v
nginx version: nginx/1.13.9

Vladimir:kubernetes vladimir$ curl -I http://${EXTERNAL_IP}:${NODE_PORT}
HTTP/1.1 200 OK
Server: nginx/1.13.9
Date: Sun, 18 Mar 2018 20:01:04 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 20 Feb 2018 12:21:20 GMT
Connection: keep-alive
ETag: "5a8c12c0-264"
Accept-Ranges: bytes
```

14. Cleaning Up

Разбираем наш стенд согласно инструкциям в туториале.  

15. Задание

```
Vladimir:kubernetes vladimir$ kubectl get pods
NAME                                  READY     STATUS    RESTARTS   AGE
busybox-855686df5d-mb6jz              1/1       Running   0          46m
comment-deployment-79b8869764-zcf5z   1/1       Running   0          32m
mongo-deployment-74cccfb8-jmrld       1/1       Running   0          32m
nginx-8586cf59-qvk55                  1/1       Running   0          43m
post-deployment-7f844c5b88-txbvn      1/1       Running   0          32m
ui-deployment-9c557b8b4-8mzvw         1/1       Running   0          32m
```

## Homework-27 swarm-1
### Основное задание

Создаем нам кластер docker-swarm
```
#master
docker-machine create --driver google \
   --google-project  docker-182408  \
   --google-zone europe-west1-b \
   --google-machine-type g1-small \
   --google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) \
   master-1
```
```
#worker
docker-machine create --driver google \
   --google-project  docker-182408  \
   --google-zone europe-west1-b \
   --google-machine-type g1-small \
   --google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) \
   worker-1
```
Все воркеры создаются аналогично.  
Подключаемся к мастер ноде и создаем кластер docker swarm:  
```
docker swarm init
```
Для подключения воркека к мастеру используем комманду:  
```
docker swarm join --token OUR_TOKEN MASTER_IP:2377
```
Просмотр нод в кластере:  
```
docker node ls
```

Далее начинаем создавать на stack. В качестве основы используем созданный ранее docker-compose файл  
Команды для работы со стеком:  
```
docker stack deploy/rm/services/ls STACK_NAME
```

Диплой стека:  
```
docker stack deploy --compose-file=<(docker-compose -f docker-compose.yml config 2>/dev/null) DEV  
#Используем env для docker swarm
#DEV - название окружения
```
Просмотр состояния стека (Сводная информация по сервисам):
```
docker stack services DEV
```
Добавление лейблов к нужной ноде:  
```
docker node update --label-add reliability=high master-
```

Просмотр лейблов на всех нодах:  
```
docker node ls -q | xargs docker node inspect  -f '{{ .ID }} [{{ .Description.Hostname }}]: {{ .Spec.Labels }}'
```

Для указания конкретного размещения сервиса добавляем конструкцию:  
```
deploy:
      placement:
        constraints:
          - node.labels.reliability == high
```

Для масштабирования сервисов используем:  
```
deploy:
      mode: replicated
      replicas: 2
```
Так же можно использовать масштабирование сервисов на лету:  
```
docker service scale DEV_ui=3
```

```
docker service update --replicas 3 DEV_ui
```

Выключение всех задач сервиса:  
```
docker service update --replicas 0 DEV_ui
```

Для обновления сервисов используем следующую конструкцию:  
```
update_config:
        delay: 5s #задержка
        parallelism: 1 #сколько обновлять одновременно
        failure_action: pause #что делать в случаи аварии
```

Для ограничение ресурсов на сервисы используем следующую конструкцию:  
```
resources:
        limits:
          cpus: '0.30'
          memory: 300M
```

Для определения действий в случае завершения работы контейнера используем следующее:  
```
restart_policy:
    condition: on-failure#состояние
    max_attempts: 15#количество попыток
    delay: 1s#задержка
```

### Задание со *

После добавление дополнительно воркера (worker-3) некоторые контейнеры переезжают на него  
для выравниваю нагрузки.


### Задание со ***
Для определения переменных (env) для разных окружений можно использовать следующую конструкцию:  
```
services:
  post_db:
    env_file:
      - .env

```
Взята из [официального мануала](https://docs.docker.com/compose/environment-variables/#the-env-file)  
То есть для compose файлов разных окружений нам нужно указать необходимый env файл (.env_DEV, .env_STAGE, .env_PROD)  


## Homework-25 logging-1
### Основное задание
В данном дз мы обновили наши приложения. Была добавлена поддержка отправки логов.  
При обновлении из стороннего репозитория не забываем оставить Dockerfile для наших приложений.  
Пересобрали все образы одной коммандой:
```
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```
Был создан инстанс в GCP, так же не забываем указывать регион размещения.  
Был запущен стек EFK (Elasticsearch, Fluentd, Kibana).  
Для Fluentd мы собрали отдельный образ с файлом конфигурации.  
Был настроен парсинк логов при помощи шаблонов GROK.  
```
grok_pattern service=%{WORD:service} \| event=%{WORD:event} \| request_id=%{GREEDYDATA:request_id} \| message='%{GREEDYDATA:message}'
```
Для распределенного трейскинга был добавлен Zipkin. Post имеет в своем коде прямую зависимость,  
по этому он валит ошибку пока не запущен ZIpkin (на запуск требуется какое то время).  
Так же Zipkin и Post должны находится в одной сети.





## Homework-22 monitoring-2
### Основное задание

Была произведена разбивка наших docker compose файлов.  
В docker-compose.yml были описаны сервисы, в docker-compose-monitoring.yml описаны сервисы мониторинга.
Для заспуска docker-compose с другим файлом использовали комманду:
```
docker-compose -f docker-compose-monitoring.yml up -d
``` 

В мониторинг был добавлен [cAdvisor](https://github.com/google/cadvisor) для мониторинга  
наших контейнеров.
```
  cadvisor:
    image: google/cadvisor:v0.29.0
    volumes:
      - '/:/rootfs:ro'
      - '/var/run:/var/run:rw'
      - '/sys:/sys:ro'
      - '/var/lib/docker/:/var/lib/docker:ro'
    ports:
      - '8080:8080'
    networks:
      front_net:
      back_net:
```
И открыты для него необходимые порты в gcp:
```
gcloud compute firewall-rules create cadvisor-default --allow tcp:8080
```
Была добавлена grafana для построения графиков из prometeus:
```
  grafana:
    image: ${USER_NAME}/grafana:${GRAF_VER}
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=secret
    depends_on:
      - prometheus
    ports:
      - 3000:3000
    networks:
      front_net:
      back_net:
```

Открыты необходимые порты в gcp:
```
gcloud compute firewall-rules create grafana-default --allow tcp:3000
```
Так же были настроены свои дашборды и добавлен дашборд для Docker из [хранилища](https://grafana.com/dashboards)
Был добавлен alertmanager для отправки оповещений с разными статусами:
```
  alertmanager:
      image: ${USER_NAME}/alertmanager
      command:
        - '--config.file=/etc/alertmanager/config.yml'
      ports:
        - 9093:9093
      networks:
        front_net:
        back_net:
```
Настроена отправка нотификаций в slack:
```
  slack_api_url: 'https://hooks.slack.com/services/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'

route:
  receiver: 'slack-notifications'

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#vladimir_d_hw'

```
В prometheus добалены оповещения:
```
groups:
  - name: alert.rules
    rules:
    - alert: InstanceDown
      expr: up == 0
      for: 1m
      labels:
        severity: page
      annotations:
        description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute'
        summary: 'Instance {{ $labels.instance }} down'
```
### Задание со *
Был создан алерт срабатывающий на большое время записи на диск:
```
    - alert: InstanceDown
      expr: rate(node_disk_write_time_ms[1m]) > 5
      for: 1m
      labels:
        severity: critical
      annotations:
        description: '{{ $labels.instance }} of job {{ $labels.job }} has a large recording time than 1 minute'
        summary: 'Instance {{ $labels.instance }} disk critical'
```
Настроена интеграция с почтой:
```
  smtp_auth_username: 'somebody@email.com'
  smtp_auth_password: 'pa$$W0rd'
  smtp_from: 'somebody@email.com'
  smtp_smarthost: 'smtp.yandex.ru:465'
  smtp_require_tls: false
#в качестве примера
```
Был собран образ grafana включающий в себя настроенные дашборды и источники данных:
```
FROM grafana/grafana:5.0.0-beta4

COPY datasource.yml /etc/grafana/provisioning/datasources/datasource.yml
COPY dashboards.yml /etc/grafana/provisioning/dashboards/dashboards.yml
COPY dashboards/*.json /var/lib/grafana/dashboards/

```

Настроена интеграция со [stackdriver](https://github.com/frodenas/stackdriver_exporter)
Как и в example удалось вытащить compute.googleapis.com/instance/cpu,compute.googleapis.com/instance/disk
Настроена интеграция prometheus с docker.
По [инструкции](https://docs.docker.com/config/thirdparty/prometheus/#configure-docker) был настроен docker-daemon.  
В конфиг были добавлены следующие строки:
```
{
  "metrics-addr" : "0.0.0.0:9323",
  "experimental" : true
}
```
Созданные нами образы были запушины в [docker hub](https://hub.docker.com/u/mrgreyves/)



## Homework-21 monitoring-1
### Основное задание

В данном задании была произведена реорганизация нашего репозитория. (Навели в нем порядок)
Созданы правила для открытия нужных портов
```
gcloud compute firewall-rules create prometheus-default --allow tcp:9090
gcloud compute firewall-rules create puma-default --allow tcp:9292
```  
Был создан хост с docker-machine в котором запущен prometheus
```
docker run --rm -p 9090:9090 -d --name prometheus  prom/prometheus
#узнать адрес машины с docker host
docker-machine ip vm1
```
Создали prometheus.yml в котором указали настройки самого prometheus
```
global:
  scrape_interval: '5s'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets:
        - 'localhost:9090'
```
Так же мы использовали node-exporter который собирает информацию с docker хоста
```
  node-exporter:
    image: prom/node-exporter
    user: root
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points="^/(sys|proc|dev|host|etc)($$|/)"'
    networks:
      back_net:
```

Для node-exporter был добавлен соответствующий job в prometheus
```
  - job_name: 'node'
    static_configs:
      - targets:
        - 'node-exporter:9100'
```
Созданные нами образы были запушины в [docker hub](https://hub.docker.com/u/mrgreyves/)
```
docker push $USER_NAME/ui
docker push $USER_NAME/comment
docker push $USER_NAME/post
docker push $USER_NAME/prometheus
```

### Задание со * 1
Был добавлен мониторинг mongo-db при помощи экспортера от [percona](https://github.com/percona/mongodb_exporter)  
Для сборки использовалась вот эта [инструкция](https://github.com/dcu/mongodb_exporter)
Был изменен файл конфигурации prometheus.
Добавлен job для mongodb-exporter
```
  - job_name: 'node'
    static_configs:
      - targets:
        - 'node-exporter:9100'
```
Образ prometheus был пересобран.
UPD: собран свой образ mongodb-exporter.


### Задание со * 2
Был добавлен мониторинг при помощи [blackbox](https://github.com/prometheus/blackbox_exporter) для наших сервисов: comment, post,ui
В файле blackbox.yml настройки самого blackbox.
Изменен конфиг prometheus
```
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://comment:9292/healthcheck
        - http://post:5000/healthcheck
        - http://ui:9292/healthcheck
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```
Образ prometheus был пересобран.
Образ blackbox так же доступен в [docker hub](https://hub.docker.com/r/prom/blackbox-exporter/)
UPD: собран свой образ blackbox-exporter.


## Homework-20 docker-7
### Основное задание

В основной части задания был перенастроен pipeline с определением веток stage и production.  
Так же для нового проекта был выбран runner который мы использовали для предыдущего.  
Были описаны окружения stage и production, выкатку на которые можно сделать только руками  
(опция manual и кнопка в pipeline).
```
production:
  stage: production
  when: manual
```
В pipeline были созданы ограничения для выкатки на stage и production только с явно  
зафиксированной версией.
```
production:
  stage: production
  when: manual
  only:
    - /^\d+\.\d+.\d+/
```
Были определены динамические окружения:
```
branch review:
  stage: review
  script: echo "Deploy to $CI_ENVIRONMENT_SLUG"
  environment:
    name: branch/$CI_COMMIT_REF_NAME
    url: http://$CI_ENVIRONMENT_SLUG.example.com
```

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
Скрипт будет примерно такого плана:
```
for n in 1..N
do
sudo docker run -d --name gitlab-runner-${n} --restart always \
   -v /srv/gitlab-runner/config:/etc/gitlab-runner \
   -v /var/run/docker.sock:/var/run/docker.sock \
   gitlab/gitlab-runner:latest
sudo docker exec -it gitlab-runner-${n} gitlab-runner register --non-interactive \
 --description my-runner-${n} \
 --url http://GIT-LAB-IP \
 --registration-token REG-TOKEN \
 --executor docker \
 --run-untagged \
 --locked=false \
 --docker-image alpine:latest    
   
```
Лучше делать это при помощи ansible, хотя самый оптимальный на мой взгляд это использовать  
swarm или kubernetes.

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
