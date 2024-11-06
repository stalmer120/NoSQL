# Cassandra: System Components

**поднять 3 узловый Cassandra кластер.**

```yml
version: "3.8"

networks:
  cassandra-net:
    driver: bridge

services:

  cassandra-1:
    image: "cassandra:latest"
    container_name: "cassandra-1"
    ports:
      - 7000:7000
      - 9042:9042
    networks:
      - cassandra-net
    environment:
      - CASSANDRA_START_RPC=true
      - CASSANDRA_RPC_ADDRESS=0.0.0.0
      - CASSANDRA_LISTEN_ADDRESS=auto
      - CASSANDRA_CLUSTER_NAME=my-cluster
      - CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch
      - CASSANDRA_DC=my-datacenter-1
    volumes:
      - ~/cassandra/node1:/var/lib/cassandra:rw
    restart:
      on-failure
    healthcheck:
      test: ["CMD-SHELL", "nodetool status"]
      interval: 2m
      start_period: 2m
      timeout: 10s
      retries: 3

  cassandra-2:
    image: "cassandra:latest"
    container_name: "cassandra-2"
    ports:
      - 9043:9042
    networks:
      - cassandra-net
    environment:
      - CASSANDRA_START_RPC=true
      - CASSANDRA_RPC_ADDRESS=0.0.0.0
      - CASSANDRA_LISTEN_ADDRESS=auto
      - CASSANDRA_CLUSTER_NAME=my-cluster
      - CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch
      - CASSANDRA_DC=my-datacenter-1
      - CASSANDRA_SEEDS=cassandra-1
    depends_on:
      cassandra-1:
        condition: service_healthy
    volumes:
      - ~/cassandra/node2:/var/lib/cassandra:rw
    restart:
      on-failure
    healthcheck:
      test: ["CMD-SHELL", "nodetool status"]
      interval: 2m
      start_period: 2m
      timeout: 10s
      retries: 3

  cassandra-3:
    image: "cassandra:latest"
    container_name: "cassandra-3"
    ports:
      - 9044:9042
    networks:
      - cassandra-net
    environment:
      - CASSANDRA_START_RPC=true
      - CASSANDRA_RPC_ADDRESS=0.0.0.0
      - CASSANDRA_LISTEN_ADDRESS=auto
      - CASSANDRA_CLUSTER_NAME=my-cluster
      - CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch
      - CASSANDRA_DC=my-datacenter-1
      - CASSANDRA_SEEDS=cassandra-1
    depends_on:
      cassandra-2:
        condition: service_healthy
    volumes:
      - ~/cassandra/node3:/var/lib/cassandra:rw
    restart:
      on-failure
    healthcheck:
      test: ["CMD-SHELL", "nodetool status"]
      interval: 2m
      start_period: 2m
      timeout: 10s
      retries: 3

volumes:
  node1:
  node2:
  node3:
```

>docker ps -a --format 'table {{.Names}}\t{{.ID}}\t{{.Status}}\t{{.Ports}}'

```
NAMES               CONTAINER ID   STATUS                   PORTS
cassandra-3         527a0730c6ed   Up 2 minutes (healthy)   7000-7001/tcp, 7199/tcp, 9160/tcp, 0.0.0.0:9044->9042/tcp
cassandra-2         457237384b43   Up 3 minutes (healthy)   7000-7001/tcp, 7199/tcp, 9160/tcp, 0.0.0.0:9043->9042/tcp
cassandra-1         7346c7d50bb3   Up 4 minutes (healthy)   7001/tcp, 0.0.0.0:7000->7000/tcp, 7199/tcp, 0.0.0.0:9042->9042/tcp, 9160/tcp
```

>nodetool status
```
Datacenter: my-datacenter-1
===========================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load        Tokens  Owns (effective)  Host ID                               Rack 
UN  172.18.0.3  85.1 KiB    16      59.3%             281f0fbe-7028-4e33-bdc3-d29795efa0f2  rack1
UN  172.18.0.2  119.82 KiB  16      64.7%             c3aea46d-37c9-452e-9f08-deb030f7e12e  rack1
UN  172.18.0.4  85.1 KiB    16      76.0%             89cfb208-ebe1-4c2d-b7b8-e834f7aca56e  rack1
```

**Создать keyspase с 2-мя таблицами. Одна из таблиц должна иметь составной Partition key, как минимум одно поле - clustering key, как минимум одно поле не входящее в primiry key.**

```sql
CREATE KEYSPACE my_keyspace WITH REPLICATION = {
  'class': 'SimpleStrategy',
  'replication_factor': 3
};

```
*Создание первой таблицы с составным Partition key и clustering key:*

```sql
USE my_keyspace;
```
```sql
CREATE TABLE table_one (
    username TEXT,
    error_time TIMESTAMP,
    error_type TEXT,
    error_message TEXT,
    PRIMARY KEY ((username, error_time), error_type)
);

```

* Partition Key: (username, error_time) — это составной ключ, который позволяет распределять данные по узлам кластера. username используется для группировки записей по пользователям, а error_time для временной сортировки ошибок.  
* Clustering Key: error_type — это ключ, который позволяет сортировать записи внутри одной партиции по типу ошибки.  
* Поле error_message: это дополнительное поле, которое хранит сообщение об ошибке и не входит в Primary Key.

>SELECT * FROM table_one;
```
username | error_time | error_type | error_message
----------+------------+------------+---------------

(0 rows)
```

*Создание второй таблицы:*
```sql
CREATE TABLE table_two (
  id uuid PRIMARY KEY,
  name text,
  age int
);
```
* id является primary key.  
* name и age — это дополнительные поля, которые не входят в primary key.
```

 id | age | name
----+-----+------

(0 rows)
```
**Заполнить данными обе таблицы.**

**заполнение первой таблицы *table_one*:**
```sql
INSERT INTO table_one (username, error_time, error_type, error_message) VALUES ('user1', '2023-10-01 10:00:00', 'Invalid Password', 'The password entered is incorrect.');
INSERT INTO table_one (username, error_time, error_type, error_message) VALUES ('user1', '2023-10-01 10:05:00', 'Account Locked', 'The account has been locked due to multiple failed login attempts.');
INSERT INTO table_one (username, error_time, error_type, error_message) VALUES ('user2', '2023-10-01 11:00:00', 'Invalid Username', 'The username does not exist.');
INSERT INTO table_one (username, error_time, error_type, error_message) VALUES ('user3', '2023-10-01 12:00:00', 'Invalid Password', 'The password entered is incorrect.');
INSERT INTO table_one (username, error_time, error_type, error_message) VALUES ('user2', '2023-10-01 12:05:00', 'Invalid Password', 'The password entered is incorrect.');
INSERT INTO table_one (username, error_time, error_type, error_message) VALUES ('user1', '2023-10-01 12:10:00', 'Session Expired', 'The session has expired, please log in again.');
INSERT INTO table_one (username, error_time, error_type, error_message) VALUES ('user3', '2023-10-01 12:15:00', 'Account Locked', 'The account has been locked due to multiple failed login attempts.');

```
>SELECT * FROM table_one;
```
username | error_time                      | error_type       | error_message
----------+---------------------------------+------------------+--------------------------------------------------------------------
    user3 | 2023-10-01 12:00:00.000000+0000 | Invalid Password |                                 The password entered is incorrect.
    user1 | 2023-10-01 12:10:00.000000+0000 |  Session Expired |                      The session has expired, please log in again.
    user2 | 2023-10-01 12:05:00.000000+0000 | Invalid Password |                                 The password entered is incorrect.
    user1 | 2023-10-01 10:05:00.000000+0000 |   Account Locked | The account has been locked due to multiple failed login attempts.
    user3 | 2023-10-01 12:15:00.000000+0000 |   Account Locked | The account has been locked due to multiple failed login attempts.
    user2 | 2023-10-01 11:00:00.000000+0000 | Invalid Username |                                       The username does not exist.
    user1 | 2023-10-01 10:00:00.000000+0000 | Invalid Password |                                 The password entered is incorrect.

(7 rows)
```

* user1: имеет три записи с разными типами ошибок (неверный пароль, заблокированная учетная запись, истекшая сессия).  
* user2: имеет две записи (неверное имя пользователя и неверный пароль).  
* user3: имеет две записи (неверный пароль и заблокированная учетная запись).


*заполнение второй таблицы  *table_two*:*
```sql
INSERT INTO table_two (id, name, age) VALUES (uuid(), 'Иван', 30);
INSERT INTO table_two (id, name, age) VALUES (uuid(), 'Марина', 25);
INSERT INTO table_two (id, name, age) VALUES (uuid(), 'Алексей', 35);
```
 * UUID: Функция uuid() автоматически генерирует уникальный идентификатор для каждого нового ввода данных.  
 *  name и age: Эти поля заполняются обычными строковыми и числовыми значениями соответственно.

```
 id                                   | age | name
--------------------------------------+-----+---------
 37bfd87f-1893-4be0-b280-306125df85a0 |  30 |    Иван
 a2c4f179-359f-4e28-bfb1-871b4aacd58e |  35 | Алексей
 5ece6334-2304-4bec-8fb6-94bda29165d0 |  25 |  Марина
```
**Выполнить 2-3 варианта запроса использую WHERE**

> SELECT * FROM table_one WHERE username = 'user1' ALLOW FILTERING;  
* поиск по user1
```
 username | error_time                      | error_type       | error_message
----------+---------------------------------+------------------+--------------------------------------------------------------------
    user1 | 2023-10-01 12:10:00.000000+0000 |  Session Expired |                      The session has expired, please log in again.
    user1 | 2023-10-01 10:05:00.000000+0000 |   Account Locked | The account has been locked due to multiple failed login attempts.
    user1 | 2023-10-01 10:00:00.000000+0000 | Invalid Password |                                 The password entered is incorrect.

(3 rows)
```
>SELECT * FROM table_one WHERE username = 'user2' AND error_type = 'Invalid Password' ALLOW FILTERING;  
* поиск по user2 и типу ошибки Invalid Password
```
 username | error_time                      | error_type       | error_message
----------+---------------------------------+------------------+------------------------------------
    user2 | 2023-10-01 12:05:00.000000+0000 | Invalid Password | The password entered is incorrect.

(1 rows)
```
>SELECT * FROM table_one WHERE username = 'user3' AND error_time >= '2023-10-01 00:00:00' AND error_time <= '2023-10-01 23:59:59' ALLOW FILTERING;
* поиск по user3 и временному диапазону
```
 username | error_time                      | error_type       | error_message
----------+---------------------------------+------------------+--------------------------------------------------------------------
    user3 | 2023-10-01 12:00:00.000000+0000 | Invalid Password |                                 The password entered is incorrect.
    user3 | 2023-10-01 12:15:00.000000+0000 |   Account Locked | The account has been locked due to multiple failed login attempts.

(2 rows)
```
**Создать вторичный индекс на поле, не входящее в primiry key.**
> CREATE INDEX ON table_one (error_message);

>SELECT * FROM table_one WHERE error_message = 'The password entered is incorrect.';
* запрос по дополнительному полю, которое хранит сообщение об ошибке.  

```
 username | error_time                      | error_type       | error_message
----------+---------------------------------+------------------+------------------------------------
    user3 | 2023-10-01 12:00:00.000000+0000 | Invalid Password | The password entered is incorrect.
    user2 | 2023-10-01 12:05:00.000000+0000 | Invalid Password | The password entered is incorrect.
    user1 | 2023-10-01 10:00:00.000000+0000 | Invalid Password | The password entered is incorrect.

(3 rows)
```
