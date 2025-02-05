## 1. Выбор туроператоров ##
  Выбираем 4-5 популярных туроператоров:
  
      TUI
      Tez Tour
      Coral Travel
      Pegas Touristik
      Anex Tour

## 2. Создание нод для туроператоров ##

  Запускаем Neo4j и используем Cypher для создания узлов (нод) туроператоров:
  ```
  CREATE (:TourOperator {name: 'TUI'}),
         (:TourOperator {name: 'Tez Tour'}),
         (:TourOperator {name: 'Coral Travel'}),
         (:TourOperator {name: 'Pegas Touristik'}),
         (:TourOperator {name: 'Anex Tour'});
```
## 3. Выбор направлений ##

  Выбираем 10-15 популярных направлений. Страны и конкретные места могут выглядеть следующим образом:
  
      Турция — Анталья, Стамбул
      Египет — Шарм-эль-Шейх, Хургада
      Испания — Барселона, Мадрид
      Таиланд — Пхукет, Бангкок
      ОАЭ — Дубай, Абу-Даби

  Создаем страны и места как ноды:
```    
    CREATE (:Country {name: 'Turkey'})-[:HAS_PLACE]->(:Place {name: 'Antalya'}),
           (:Country {name: 'Turkey'})-[:HAS_PLACE]->(:Place {name: 'Istanbul'}),
           (:Country {name: 'Egypt'})-[:HAS_PLACE]->(:Place {name: 'Sharm El-Sheikh'}),
           (:Country {name: 'Egypt'})-[:HAS_PLACE]->(:Place {name: 'Hurghada'}),
           (:Country {name: 'Spain'})-[:HAS_PLACE]->(:Place {name: 'Barcelona'}),
           (:Country {name: 'Spain'})-[:HAS_PLACE]->(:Place {name: 'Madrid'}),
           (:Country {name: 'Thailand'})-[:HAS_PLACE]->(:Place {name: 'Phuket'}),
           (:Country {name: 'Thailand'})-[:HAS_PLACE]->(:Place {name: 'Bangkok'}),
           (:Country {name: 'UAE'})-[:HAS_PLACE]->(:Place {name: 'Dubai'}),
           (:Country {name: 'UAE'})-[:HAS_PLACE]->(:Place {name: 'Abu Dhabi'});
```
## 4. Создание связей между туроператорами и местами ##

  Связываем туроператоров с направлениями, куда они предоставляют туры:
  ```
    MATCH (tui:TourOperator {name: 'TUI'}), (place:Place {name: 'Antalya'})
    CREATE (tui)-[:OFFERS]->(place);
    
    MATCH (tez:TourOperator {name: 'Tez Tour'}), (place:Place {name: 'Sharm El-Sheikh'})
    CREATE (tez)-[:OFFERS]->(place);
```
## 5. Добавление городов с транспортной инфраструктурой ##

  Добавляем ноды для городов с аэропортами или вокзалами:
  ```
    CREATE (:City {name: 'Istanbul', has_airport: true}),
           (:City {name: 'Antalya', has_airport: true}),
           (:City {name: 'Sharm El-Sheikh', has_airport: true}),
           (:City {name: 'Barcelona', has_airport: true}),
           (:City {name: 'Phuket', has_airport: true});
```
## 6. Создание маршрутов между городами ##

  Добавляем связи между городами с указанием вида транспорта:
```
    MATCH (istanbul:City {name: 'Istanbul'}), (antalya:City {name: 'Antalya'})
    CREATE (istanbul)-[:ROUTE {transport: 'bus'}]->(antalya);
    
    MATCH (barcelona:City {name: 'Barcelona'}), (madrid:City {name: 'Madrid'})
    CREATE (barcelona)-[:ROUTE {transport: 'train'}]->(madrid);
```
## 7. Запрос для поиска маршрутов по наземному транспорту ##

  Создаем запрос для поиска маршрутов, которые доступны только наземным транспортом (автобус, поезд):
```
    MATCH path = (c1:City)-[r:ROUTE]->(c2:City)
    WHERE r.transport IN ['bus', 'train']
    RETURN path;
```
## 8. Составление плана запроса ##

  Для анализа плана выполнения запроса используем команду EXPLAIN:
```
    EXPLAIN
    MATCH path = (c1:City)-[r:ROUTE]->(c2:City)
    WHERE r.transport IN ['bus', 'train']
    RETURN path;
```
  Команда EXPLAIN покажет план запроса, но не выполнит его, что позволяет проверить, как Neo4j собирается обрабатывать запрос.
## 9. Добавление индексов для оптимизации ##

  Добавляем индексы для улучшения производительности запросов, например, на атрибут транспорта:
```
  CREATE INDEX FOR (r:ROUTE) ON (r.transport);
```
## 10. Проверка плана запроса ##

  Теперь используем команду PROFILE, чтобы убедиться, что индексы применяются:
```
    PROFILE
    MATCH path = (c1:City)-[r:ROUTE]->(c2:City)
    WHERE r.transport IN ['bus', 'train']
    RETURN path;
```
  PROFILE выполнит запрос и покажет, как Neo4j использует индексы для оптимизации.

### Выводы: В этой задаче работа с графовой структурой в Neo4j упрощает представление данных, таких как маршруты и связи между городами и операторами. Использование индексов помогает оптимизировать запросы, особенно при работе с большими наборами данных. ###
