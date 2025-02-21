# Redis test

## Тестовая среда

**Redis 7.0.2**, single node.

**Redis 7.0.2**, 3 masters, 3 replicas.

Сборка:
````bash
docker-compose up -d
````

Создание кластера:
````bash
docker-compose exec redis-1 redis-cli --cluster create redis-1:6379 redis-2:6379 redis-3:6379 redis-4:6379 redis-5:6379 redis-6:6379 --cluster-replicas 1
````

Тест одной ноды:
````bash
docker-compose exec redis-0 redis-benchmark -n 1000000 -c 100 -k 1 -P 16 -q
````

Тест json:
````bash
docker-compose exec redis-0 /test-json.sh
````

Тест кластера:
````bash
docker-compose exec redis-1 redis-benchmark -n 1000000 -c 100 -k 1 -P 16 -q --cluster
````

Очистка:
````bash
docker-compose down -v
````

## Пример вывода

Создание кластера:
````bash
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica redis-5:6379 to redis-1:6379
Adding replica redis-6:6379 to redis-2:6379
Adding replica redis-4:6379 to redis-3:6379
M: eeacd2550723429b692fd7a1fec5830313f97389 redis-1:6379
   slots:[0-5460] (5461 slots) master
M: 9f263a1d6bc1fea9f03bdd26eb9299561a52f257 redis-2:6379
   slots:[5461-10922] (5462 slots) master
M: 90090e5ee5bdf9e837e4b5988f3f64216c6a0236 redis-3:6379
   slots:[10923-16383] (5461 slots) master
S: 7aa3c0721510b31ddd64a3a199cb618fcf5ea4e8 redis-4:6379
   replicates 90090e5ee5bdf9e837e4b5988f3f64216c6a0236
S: 7195e573a91592da5c9e1e143edea7a589d90159 redis-5:6379
   replicates eeacd2550723429b692fd7a1fec5830313f97389
S: d61a6c4421bf925ef5168cf510451bb9a12a8c9d redis-6:6379
   replicates 9f263a1d6bc1fea9f03bdd26eb9299561a52f257
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node redis-1:6379)
M: eeacd2550723429b692fd7a1fec5830313f97389 redis-1:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 9f263a1d6bc1fea9f03bdd26eb9299561a52f257 172.27.0.5:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 7aa3c0721510b31ddd64a3a199cb618fcf5ea4e8 172.27.0.2:6379
   slots: (0 slots) slave
   replicates 90090e5ee5bdf9e837e4b5988f3f64216c6a0236
S: 7195e573a91592da5c9e1e143edea7a589d90159 172.27.0.6:6379
   slots: (0 slots) slave
   replicates eeacd2550723429b692fd7a1fec5830313f97389
S: d61a6c4421bf925ef5168cf510451bb9a12a8c9d 172.27.0.7:6379
   slots: (0 slots) slave
   replicates 9f263a1d6bc1fea9f03bdd26eb9299561a52f257
M: 90090e5ee5bdf9e837e4b5988f3f64216c6a0236 172.27.0.4:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
````

Тест одной ноды:
````bash
PING_INLINE: 44676.77 requests per second, p50=35.199 msec                    
PING_MBULK: 45108.03 requests per second, p50=34.975 msec                    
SET: 46334.91 requests per second, p50=36.191 msec                    
GET: 42780.75 requests per second, p50=36.255 msec                    
INCR: 41389.02 requests per second, p50=36.703 msec                    
LPUSH: 42292.24 requests per second, p50=38.367 msec                    
RPUSH: 40272.24 requests per second, p50=37.631 msec                    
LPOP: 41027.32 requests per second, p50=37.887 msec                    
RPOP: 44002.46 requests per second, p50=37.279 msec                    
SADD: 45308.32 requests per second, p50=35.903 msec                    
HSET: 41228.61 requests per second, p50=37.599 msec                    
SPOP: 48353.56 requests per second, p50=35.039 msec                    
ZADD: 44812.91 requests per second, p50=37.055 msec                    
ZPOPMIN: 45945.33 requests per second, p50=35.327 msec                     
LPUSH (needed to benchmark LRANGE): 44620.95 requests per second, p50=37.983 msec                    
LRANGE_100 (first 100 elements): 29969.73 requests per second, p50=38.847 msec                    
LRANGE_300 (first 300 elements): 12683.43 requests per second, p50=66.047 msec                    
LRANGE_500 (first 500 elements): 9826.85 requests per second, p50=64.927 msec                     
LRANGE_600 (first 600 elements): 6834.20 requests per second, p50=93.503 msec                     
MSET (10 keys): 42354.93 requests per second, p50=38.943 msec
````

Тест json:
````bash
OK

SET_PREPARE_TMP_FILE
33777832 /tmp/redis-test
EXECUTE_TMP_FILE
OK
real    0m 1.41s
user    0m 1.34s
sys     0m 0.05s

HSET_PREPARE_TMP_FILE
43777780 /tmp/redis-test
EXECUTE_TMP_FILE_PIPE
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1000000
real    0m 3.19s
user    0m 0.38s
sys     0m 0.38s

ZADD_PREPARE_TMP_FILE
41777780 /tmp/redis-test
EXECUTE_TMP_FILE_PIPE
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1000000
real    0m 3.39s
user    0m 0.37s
sys     0m 0.36s

RPUSH_PREPARE_TMP_FILE
30888890 /tmp/redis-test
EXECUTE_TMP_FILE_PIPE
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1000000
real    0m 2.06s
user    0m 0.38s
sys     0m 0.28s


GET_PREPARE_TMP_FILE
11 /tmp/redis-test
EXECUTE_TMP_FILE_PIPE
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1
real    0m 0.09s
user    0m 0.03s
sys     0m 0.04s

HGET_PREPARE_TMP_FILE
23888890 /tmp/redis-test
EXECUTE_TMP_FILE_PIPE
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1000000
real    0m 1.84s
user    0m 0.44s
sys     0m 0.27s

HGETALL_PREPARE_TMP_FILE
13 /tmp/redis-test
EXECUTE_TMP_FILE_PIPE
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1
real    0m 1.04s
user    0m 0.38s
sys     0m 0.12s

ZRANGE_PREPARE_TMP_FILE
22 /tmp/redis-test
EXECUTE_TMP_FILE_PIPE
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1
real    0m 0.36s
user    0m 0.16s
sys     0m 0.06s

LRANGE_PREPARE_TMP_FILE
17 /tmp/redis-test
EXECUTE_TMP_FILE_PIPE
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1
real    0m 0.35s
user    0m 0.16s
sys     0m 0.05s

````

Тест кластера:
````bash
Cluster has 3 master nodes:

Master 0: 9f263a1d6bc1fea9f03bdd26eb9299561a52f257 172.27.0.5:6379
Master 1: eeacd2550723429b692fd7a1fec5830313f97389 172.27.0.3:6379
Master 2: 90090e5ee5bdf9e837e4b5988f3f64216c6a0236 172.27.0.4:6379

PING_INLINE: 86490.23 requests per second, p50=10.127 msec                     
PING_MBULK: 159565.97 requests per second, p50=9.831 msec                    
SET: 136537.41 requests per second, p50=10.927 msec                     
GET: 153209.75 requests per second, p50=10.279 msec                     
INCR: 136537.41 requests per second, p50=10.791 msec                     
LPUSH: 131787.03 requests per second, p50=11.575 msec                     
RPUSH: 136761.48 requests per second, p50=10.895 msec                     
LPOP: 131787.03 requests per second, p50=11.279 msec                     
RPOP: 87527.35 requests per second, p50=13.159 msec                      
SADD: 110729.71 requests per second, p50=10.967 msec                     
HSET: 91945.56 requests per second, p50=12.623 msec                      
SPOP: 153162.81 requests per second, p50=10.199 msec                    
ZADD: 147536.16 requests per second, p50=10.775 msec                     
ZPOPMIN: 153374.23 requests per second, p50=10.151 msec                    
LPUSH (needed to benchmark LRANGE): 127860.88 requests per second, p50=11.559 msec                     
LRANGE_100 (first 100 elements): 19511.81 requests per second, p50=39.711 msec                    
LRANGE_300 (first 300 elements): 8627.01 requests per second, p50=56.735 msec                   
LRANGE_500 (first 500 elements): 4334.99 requests per second, p50=80.831 msec                    
LRANGE_600 (first 600 elements): 3861.53 requests per second, p50=100.607 msec                   
MSET (10 keys): 103971.72 requests per second, p50=13.175 msec
````
