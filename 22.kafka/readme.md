#### Работаем с RabbitMQ

Развернем RabbitMQ в контейнере, docker-compose.yaml:

```yaml
# cat docker-compose.yaml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    ports:
      - '5672:5672'
      - '15672:15672'
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
```
Запуск
```sh
# docker compose -f docker-compose.yaml up -d
WARN[0000] /root/rabbit2/docker-compose.yaml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 2/2
 ✔ Network rabbit2_default  Created                                                                                                                     0.1s
 ✔ Container rabbitmq       Started                                                                                                                     0.4s
root@dvtest-dbc001lk:~/rabbit2# docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                                                                                                                                 NAMES
03dd06635c24   rabbitmq:3-management   "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp, :::5672->5672/tcp, 15671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp, :::15672->15672/tcp   rabbitmq
```
Заходим http://localhost:15672/ под учетной записью и паролем guest guest

Создадим exchange test

![Alt text](test_ex.png?raw=true "test_ex")

Создадим очередь test_q

![Alt text](test_q_manual.png?raw=true "test_q_manual")

Забиндим наш обменник и очередь

![Alt text](bind_manual.png?raw=true "bind_manual")

Опубликуем тестовое сообщение с ключом маршрутизации test

![Alt text](pub_test_mess.png?raw=true "pub_test_mess")

Тестовое сообщение попало в очередь в статусе ready

![Alt text](ready_test_mess_unack.png?raw=true "ready_test_mess_unack")


Получим тестовое сообщение 

![Alt text](get_test_mess.png?raw=true "get_test_mess")


При получении сообщения поставили флаг автоматического ack, соответсвующая картина видна на мониторинге очереди

![Alt text](test_ack.png?raw=true "test_ack")

Для отправки-получения сообщений программно установим Python и библиотеку для работы с RabbitMQ
```sh
#apt update
#apt install python3-pip

# pip install pika
Collecting pika
  Downloading pika-1.3.2-py3-none-any.whl (155 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 155.4/155.4 KB 1.6 MB/s eta 0:00:00
Installing collected packages: pika
Successfully installed pika-1.3.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```
Создадим файлик проекта и пропишем там необходимые зависимости и подключения
```python
# cat rabbitmq.py
import pika
import os

class RabbitMQ:
    def __init__(self):
        self.user = os.getenv('RABBITMQ_USER', 'guest')
        self.password = os.getenv('RABBITMQ_PASSWORD', 'guest')
        self.host = os.getenv('RABBITMQ_HOST', 'localhost')
        self.port = int(os.getenv('RABBITMQ_PORT', 5672))
        self.connection = None
        self.channel = None
        self.connect()

    def connect(self):
        credentials = pika.PlainCredentials(self.user, self.password)
        parameters = pika.ConnectionParameters(host=self.host, port=self.port, credentials=credentials)
        self.connection = pika.BlockingConnection(parameters)
        self.channel = self.connection.channel()

    def close(self):
        if self.connection and not self.connection.is_closed:
            self.connection.close()

    def consume(self, queue_name, callback):
        if not self.channel:
            raise Exception("Connection is not established.")
        self.channel.basic_consume(queue=queue_name, on_message_callback=callback, auto_ack=True)
        self.channel.start_consuming()

    def publish(self, queue_name, message):
        if not self.channel:
            raise Exception("Connection is not established.")
        self.channel.queue_declare(queue=queue_name, durable=True)
        self.channel.basic_publish(exchange='',
                                   routing_key=queue_name,
                                   body=message,
                                   properties=pika.BasicProperties(
                                       delivery_mode=2,  # make message persistent
                                   ))
        print(f"Sent message to queue {queue_name}: {message}")
```

Скрипт rabbitmq.py [приложен](https://github.com/ole-vep/otus_nosql/blob/main/13/rabbitmq.py)

Создадим скрипт публикации сообщений
```python
# cat publisher.py
from rabbitmq import RabbitMQ

def publish_test_message():
    rabbitmq = RabbitMQ()
    try:
        rabbitmq.publish(queue_name='test_queue', message='Test message')
        print("Test message published successfully.")
    except Exception as e:
        print(f"Failed to publish test message: {e}")
    finally:
        rabbitmq.close()

if __name__ == "__main__":
    publish_test_message()
```
Скрипт publisher.py [приложен](https://github.com/ole-vep/otus_nosql/blob/main/13/publisher.py)

И также создадим скрипт получения сообщений
```python
# cat main.py
from rabbitmq import RabbitMQ
import sys

def callback(ch, method, properties, body):
    print(f"Received message: {body}")

def main():
    rabbitmq = RabbitMQ()
    try:
        print("Connection to RabbitMQ established successfully.")
        rabbitmq.consume(queue_name='test_queue', callback=callback)
    except Exception as e:
        print(f"Failed to establish connection to RabbitMQ: {e}")
        sys.exit(1)
    finally:
        rabbitmq.close()

if __name__ == "__main__":
    main()
```

Скрипт main.py 

Создадим дополнительно очередь test_queue

Запускаем скрипт main.py

Успешно подключились к очереди и ждём сообщений

```sh
# python3 main.py
Connection to RabbitMQ established successfully.
```
Запускаем скрипт publisher.py в другом терминале
```sh
# python3 publisher.py
Sent message to queue test_queue: Test message
Test message published successfully.
```
Сообщение опубликовано



В первом терминале, где был запущен main.py, появилось сообщение



Также можно проверить web UI раздел обзор



Автоматически создался обменник (AMQP default)


Появились сведения об открытом соединении и канале





Сообщения в программном режиме было доставлено мнгновенно, поэтом наблюдаем просто всплески на мониторинге
