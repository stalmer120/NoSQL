# etcd cluster test

## Тестовая среда

**etcd 3.4.18, 5 nodes**.

Сборка:
````bash
docker-compose up -d etcd-1 etcd-2 etcd-3 etcd-4 etcd-5
````

Запуск теста:
````bash
./test-etcd.sh
````

Очистка:
````bash
docker-compose down -v
````

## Пример вывода

Тест:
````bash
[KEYS = 10][NODES = 5][FAILURE TOLERANCE = 2]

[NODE1] etcdctl member list
1e738073e57b0810, started, etcd3, http://etcd-3:2380, http://etcd-3:2379, false
7ab40cdf5677e876, started, etcd5, http://etcd-5:2380, http://etcd-5:2379, false
97ecdd3706cc2d90, started, etcd2, http://etcd-2:2380, http://etcd-2:2379, false
a3959071884acd0c, started, etcd1, http://etcd-1:2380, http://etcd-1:2379, false
eb60c5a827c7b5ca, started, etcd4, http://etcd-4:2380, http://etcd-4:2379, false

[NODE2] etcdctl del "" --from-key=true
0

[NODE1] etcdctl put key0 value0
OK

[NODE2] etcdctl put key1 value1
OK

[NODE3] etcdctl put key2 value2
OK

[NODE4] etcdctl put key3 value3
OK

[NODE5] etcdctl put key4 value4
OK

[NODE1] etcdctl put key5 value5
OK

[NODE2] etcdctl put key6 value6
OK

[NODE3] etcdctl put key7 value7
OK

[NODE4] etcdctl put key8 value8
OK

[NODE5] etcdctl put key9 value9
OK

[NODE1] etcdctl get --prefix key
key0
value0
key1
value1
key2
value2
key3
value3
key4
value4
key5
value5
key6
value6
key7
value7
key8
value8
key9
value9

[NODE1] STOP
[+] Running 1/1
 ⠿ Container etcd-1  Stopped                                                                                                                                                                                     0.8s

[NODE2] etcdctl get --prefix key
key0
value0
key1
value1
key2
value2
key3
value3
key4
value4
key5
value5
key6
value6
key7
value7
key8
value8
key9
value9

[NODE2] STOP
[+] Running 1/1
 ⠿ Container etcd-2  Stopped                                                                                                                                                                                     0.6s

[NODE5] etcdctl get --prefix key
key0
value0
key1
value1
key2
value2
key3
value3
key4
value4
key5
value5
key6
value6
key7
value7
key8
value8
key9
value9

[NODE3] STOP
[+] Running 1/1
 ⠿ Container etcd-3  Stopped                                                                                                                                                                                     0.6s

[NODE5] etcdctl get --prefix key
{"level":"warn","ts":"2022-07-06T19:12:10.999Z","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"endpoint://client-f891c00b-f249-4d71-9c84-61ef5d61e336/127.0.0.1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = context deadline exceeded"}
Error: context deadline exceeded
````


# Consul cluster test

## Тестовая среда

**Consul 1.12.2, 3 nodes**.

Сборка:
````bash
docker-compose up -d consul-1 consul-2 consul-3
````

Запуск теста:
````bash
./test-consul.sh
````

Очистка:
````bash
docker-compose down -v
````

## Пример вывода

Тест:
````bash
[KEYS = 10][NODES = 3][FAILURE TOLERANCE = 1]

[NODE1] consul members
Node           Address             Status  Type    Build   Protocol  DC   Partition  Segment
192.168.224.2  192.168.224.2:8301  alive   server  1.12.2  2         dc1  default    <all>
192.168.224.3  192.168.224.3:8301  alive   server  1.12.2  2         dc1  default    <all>
192.168.224.4  192.168.224.4:8301  alive   server  1.12.2  2         dc1  default    <all>

[NODE2] consul kv delete test/
Success! Deleted key: test/

[NODE1] consul kv put test/key0 value0
Success! Data written to: test/key0

[NODE2] consul kv put test/key1 value1
Success! Data written to: test/key1

[NODE3] consul kv put test/key2 value2
Success! Data written to: test/key2

[NODE1] consul kv put test/key3 value3
Success! Data written to: test/key3

[NODE2] consul kv put test/key4 value4
Success! Data written to: test/key4

[NODE3] consul kv put test/key5 value5
Success! Data written to: test/key5

[NODE1] consul kv put test/key6 value6
Success! Data written to: test/key6

[NODE2] consul kv put test/key7 value7
Success! Data written to: test/key7

[NODE3] consul kv put test/key8 value8
Success! Data written to: test/key8

[NODE1] consul kv put test/key9 value9
Success! Data written to: test/key9

[NODE1] consul kv get -recurse
test/key0:value0
test/key1:value1
test/key2:value2
test/key3:value3
test/key4:value4
test/key5:value5
test/key6:value6
test/key7:value7
test/key8:value8
test/key9:value9

[NODE1] STOP
[+] Running 1/1
 ⠿ Container consul-1  Stopped                                                                                                                                                                                   0.6s

[NODE3] consul kv get -recurse
test/key0:value0
test/key1:value1
test/key2:value2
test/key3:value3
test/key4:value4
test/key5:value5
test/key6:value6
test/key7:value7
test/key8:value8
test/key9:value9

[NODE2] STOP
[+] Running 1/1
 ⠿ Container consul-2  Stopped                                                                                                                                                                                   0.6s

[NODE3] consul kv get -recurse
Error querying Consul agent: Unexpected response code: 500 (No cluster leader)
````
