## Цели занятия

описать основы работы с Clickhouse, архитектура и возможности.

## Краткое содержание

когда Clickhouse полезен, а когда не стоит его использовать;  
основные движки таблиц/БД Clickhouse и их особенности;  
движок MergeTree и его специфика;  
использование совместно с Kafka и PostgreSQL  
протоколы доступа (http, tcp и базовые библиотеки);  
установка и настройка


## Компетенции

1. эксплуатация БД  
   * \- установка NoSQL БД  
   * \- основы работы с NoSQL БД  
   * \- синтаксис и особенности работы встроенного языка NoSQL БД  
2. выбор БД в зависимости от задачи  
   * \- администрирование колоночных БД

## Домашнее задание

Установка ClickHouse  
**Цель:**

В результате выполнения ДЗ в разверните БД.

**Описание/Пошаговая инструкция выполнения домашнего задания:**

Необходимо, используя туториал [https://clickhouse.tech/docs/ru/getting-started/tutorial/](https://clickhouse.tech/docs/ru/getting-started/tutorial/) :

* развернуть БД;  
* выполнить импорт тестовой БД;  
* выполнить несколько запросов и оценить скорость выполнения.  
* развернуть дополнительно одну из тестовых БД [https://clickhouse.com/docs/en/getting-started/example-datasets](https://clickhouse.com/docs/en/getting-started/example-datasets) , протестировать скорость запросов  
  * развернуть Кликхаус в кластерном исполнении, создать распределенную таблицу, заполнить данными и протестировать скорость по сравнению с 1 инстансом  
    Дз сдается в виде миниотчета.

