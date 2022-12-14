# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [elasticsearch:7](https://hub.docker.com/_/elasticsearch) как базовый:

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib` 
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения
- обратите внимание на настройки безопасности такие как `xpack.security.enabled` 
- если докер образ не запускается и падает с ошибкой 137 в этом случае может помочь настройка `-e ES_HEAP_SIZE`
- при настройке `path` возможно потребуется настройка прав доступа на директорию

Далее мы будем работать с данным экземпляром elasticsearch.

## Ответ  
Логинемся:
```
root@gitlab-podman2:~/netology/virt-homeworks/06-db-05-elasticsearch# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: aleksshdevops
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

В настоящий момент актуальная версия elasticsearch - 7.17.8 (https://hub.docker.com/_/elasticsearch). На основе ней и будем собирать образ:  
```
FROM docker.io/elasticsearch:7.17.8

LABEL version=0.0.1
LABEL MAINTAINER aleks-sh


COPY --chown=elasticsearch:elasticsearch ./elasticsearch.yml /usr/share/elasticsearch/config/

RUN chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/config/ && \
chown -R elasticsearch:elasticsearch /var/lib && \
mkdir /usr/share/elasticsearch/snapshots && \
chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/snapshots

# https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html
CMD ["sysctl -w vm.max_map_count=262144"]

CMD ["elasticsearch"]

EXPOSE 9200
```



elasticsearch.yml:  
```
cluster.name: "docker-cluster-netology"
discovery.type: single-node
network.host: 0.0.0.0
node.name: netology_test
path.data: /var/lib
path.repo: /usr/share/elasticsearch/snapshots
xpack.license.self_generated.type: basic
xpack.security.enabled: false
xpack.monitoring.collection.enabled: true
```

Собираю образ  
```
root@gitlab-podman2:~/netology/virt-homeworks/06-db-05-elasticsearch# docker build . -t aleksshdevops/elasticsearch:latest
Sending build context to Docker daemon  9.728kB
Step 1/8 : FROM docker.io/elasticsearch:7.17.8
 ---> 6b4df3b1f757
Step 2/8 : LABEL version=0.0.1
 ---> Running in 2aa994f707ca
Removing intermediate container 2aa994f707ca
 ---> a74aff841f26
Step 3/8 : LABEL MAINTAINER aleks-sh
 ---> Running in 5b037f53be6a
Removing intermediate container 5b037f53be6a
 ---> 5a0214b8e4ba
Step 4/8 : COPY --chown=elasticsearch:elasticsearch ./elasticsearch.yml /usr/share/elasticsearch/config/
 ---> e30a077c173e
Step 5/8 : RUN chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/config/ && chown -R elasticsearch:elasticsearch /var/lib && mkdir /usr/share/elasticsearch/snapshots && chown -R elasticsearch:elasticsearch /usr/share/elasticsearch/snapshots
 ---> Running in 976fb90a17a4
Removing intermediate container 976fb90a17a4
 ---> d5696e51a560
Step 6/8 : CMD ["sysctl -w vm.max_map_count=262144"]
 ---> Running in 5bf9655c74a5
Removing intermediate container 5bf9655c74a5
 ---> 2731c8080a73
Step 7/8 : CMD ["elasticsearch"]
 ---> Running in 5c55cdfcc629
Removing intermediate container 5c55cdfcc629
 ---> 6424af34b61b
Step 8/8 : EXPOSE 9200
 ---> Running in cb068bc068cf
Removing intermediate container cb068bc068cf
 ---> 2743bb5002f5
Successfully built 2743bb5002f5
Successfully tagged aleksshdevops/elasticsearch:latest

root@gitlab-podman2:~/netology/virt-homeworks/06-db-05-elasticsearch# docker images
REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
aleksshdevops/elasticsearch   latest    2743bb5002f5   47 seconds ago   625MB
```

Стартуем:  
```
docker run  -d -p 9200:9200 --name el_netology aleksshdevops/elasticsearch

root@gitlab-podman2:~/netology/virt-homeworks/06-db-05-elasticsearch# docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED          STATUS         PORTS                              NAMES
7e02906a46de   aleksshdevops/elasticsearch   "/bin/tini -- /usr/l…"   10 seconds ago   Up 9 seconds   0.0.0.0:9200->9200/tcp, 9300/tcp   el_netology

root@gitlab-podman2:~/netology/virt-homeworks/06-db-05-elasticsearch# docker exec -it el_netology bash
root@7e02906a46de:/usr/share/elasticsearch#
root@7e02906a46de:/usr/share/elasticsearch# ls -lah /usr/share/elasticsearch/config/
total 72K
drwxrwxr-x 1 elasticsearch elasticsearch 4.0K Dec 14 11:08 .
drwxrwxr-x 1 root          root          4.0K Dec 14 11:09 ..
-rw-rw-r-- 1 elasticsearch elasticsearch 1.1K Dec  2 17:32 elasticsearch-plugins.example.yml
-rw-rw---- 1 elasticsearch root           199 Dec 14 11:08 elasticsearch.keystore
-rw-r--r-- 1 elasticsearch elasticsearch  298 Dec 14 11:01 elasticsearch.yml
-rw-rw-r-- 1 elasticsearch elasticsearch 3.2K Dec  2 17:32 jvm.options
drwxrwxr-x 1 elasticsearch elasticsearch 4.0K Dec  2 17:33 jvm.options.d
-rw-rw-r-- 1 elasticsearch elasticsearch  19K Dec  2 17:34 log4j2.file.properties
-rw-rw-r-- 1 elasticsearch elasticsearch  11K Dec  2 17:37 log4j2.properties
-rw-rw-r-- 1 elasticsearch elasticsearch  473 Dec  2 17:34 role_mapping.yml
-rw-rw-r-- 1 elasticsearch elasticsearch  197 Dec  2 17:34 roles.yml
-rw-rw-r-- 1 elasticsearch elasticsearch    0 Dec  2 17:34 users
-rw-rw-r-- 1 elasticsearch elasticsearch    0 Dec  2 17:34 users_roles

root@7e02906a46de:/usr/share/elasticsearch# curl -X GET "http://localhost:9200"
{
  "name" : "netology_test",
  "cluster_name" : "docker-cluster-netology",
  "cluster_uuid" : "b-VP_SQtTSSISnwb3JtJ0w",
  "version" : {
    "number" : "7.17.8",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "120eabe1c8a0cb2ae87cffc109a5b65d213e9df1",
    "build_date" : "2022-12-02T17:33:09.727072865Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
root@gitlab-podman2:~/netology/virt-homeworks/06-db-05-elasticsearch# curl -X GET "http://localhost:9200"
{
  "name" : "netology_test",
  "cluster_name" : "docker-cluster-netology",
  "cluster_uuid" : "b-VP_SQtTSSISnwb3JtJ0w",
  "version" : {
    "number" : "7.17.8",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "120eabe1c8a0cb2ae87cffc109a5b65d213e9df1",
    "build_date" : "2022-12-02T17:33:09.727072865Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

Пушим образ в докерхаб:  
```
root@gitlab-podman2:~/netology/virt-homeworks/06-db-05-elasticsearch# docker push aleksshdevops/elasticsearch:latest
The push refers to repository [docker.io/aleksshdevops/elasticsearch]
c0485d023b5c: Pushed
977b4153977c: Pushed
95e3c91ca412: Mounted from library/elasticsearch
2fb19d3db29c: Mounted from library/elasticsearch
b8649a4bec51: Mounted from library/elasticsearch
caeae38202e4: Mounted from library/elasticsearch
b20ca16e12ed: Mounted from library/elasticsearch
b20d89c2158f: Mounted from library/elasticsearch
d01b73126382: Mounted from library/elasticsearch
d3768bc47ba0: Mounted from library/elasticsearch
f4462d5b2da2: Mounted from library/elasticsearch
latest: digest: sha256:1b6545ea8ca923d15838a856b6cf865added690059c77d27ecc8186a2a4175a6 size: 2622
```

Успех https://hub.docker.com/r/aleksshdevops/elasticsearch

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

