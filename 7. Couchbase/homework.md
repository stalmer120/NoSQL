# Couchbase
**Развернуть кластер Couchbase**

*Docker-compose*

```yml
version: '7.6.3'  # Убедитесь, что версия соответствует вашей конфигурации
services:
  couchbase1:
    image: couchbase/server
    volumes:
      - ~/couchbase/node1:/opt/couchbase/var
    ports:
      - 8094:8091 
  couchbase2:
    image: couchbase/server
    volumes:
      - ~/couchbase/node2:/opt/couchbase/var
    ports:
      - 8095:8091
  couchbase3:
    image: couchbase/server
    volumes:
      - ~/couchbase/node3:/opt/couchbase/var
    ports:
      - 8091:8091
      - 8092:8092
      - 8093:8093
      - 11210:11210
```

>docker ps
```
CONTAINER ID   IMAGE              COMMAND                  CREATED              STATUS              PORTS                                                                                                                        NAMES
f7e7db48b8c7   couchbase/server   "/entrypoint.sh couc…"   About a minute ago   Up About a minute   8094-8097/tcp, 9123/tcp, 0.0.0.0:8091-8093->8091-8093/tcp, 11207/tcp, 11280/tcp, 0.0.0.0:11210->11210/tcp, 18091-18097/tcp   stalmer-couchbase3-1
6ee99a8ce0db   couchbase/server   "/entrypoint.sh couc…"   About a minute ago   Up About a minute   8092-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp, 0.0.0.0:8095->8091/tcp                            stalmer-couchbase2-1
0c4b213e2b2c   couchbase/server   "/entrypoint.sh couc…"   About a minute ago   Up About a minute   8092-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp, 0.0.0.0:8094->8091/tcp                            stalmer-couchbase1-1
```

>docker inspect -f '{{.Name}}: {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```
/stalmer-couchbase1-1: 172.18.0.2
/stalmer-couchbase2-1: 172.18.0.4
/stalmer-couchbase3-1: 172.18.0.3
```

*Проверить отказоустойчивость*

![failover](https://github.com/stalmer120/NoSQL/blob/main/png/Couchbase-001.png)

![rebalance](https://github.com/stalmer120/NoSQL/blob/main/png/Couchbase-002.png)

*Создать БД, наполнить небольшими тестовыми данными*

```sql
INSERT INTO `test_bucket` (KEY, VALUE)
   > VALUES 
   >     ("test_doc_1", {"name": "Test Document 1", "value": 1}),
   >     ("test_doc_2", {"name": "Test Document 2", "value": 2}),
   >     ("test_doc_3", {"name": "Test Document 3", "value": 3}),
   >     ("test_doc_4", {"name": "Test Document 4", "value": 4}),
   >     ("test_doc_5", {"name": "Test Document 5", "value": 5});
```   
![db](https://github.com/stalmer120/NoSQL/blob/main/png/Couchbase-003.png)
