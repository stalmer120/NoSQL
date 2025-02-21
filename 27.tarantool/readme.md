#### Работа с Tarantool

Скачаем докер образ и запустим последнюю версию Tarantool
```sh
# docker pull tarantool/tarantool
# docker run --name mytarantool -p 3301:3301 -d tarantool/tarantool
# docker exec -t -i mytarantool status
running

```
Зайдем в консоль, создадим таблицу, зададим её формат

```sh
# docker exec -t -i mytarantool console

/var/run/tarantool/sys_env/default/instance-001/tarantool.control> box.schema.space.create('avia_tickets',{ if_not_exists = true })
---
- is_local: false
  engine: memtx
  before_replace: 'function: 0x7fe570117558'
  field_count: 0
  is_sync: false
  on_replace: 'function: 0x7fe570117520'
  state:
    is_sync: false
  temporary: false
  index: []
  type: normal
  enabled: false
  name: avia_tickets
  id: 512
- created
...

> box.space.avia_tickets:format({
    {name = 'id', type = 'unsigned'},
    {name = 'airline', type = 'string'},
    {name = 'departure_date', type = 'string'},
    {name = 'departure_city', type = 'string'},
    {name = 'arrival_city ', type = 'string'},
    {name = 'min_price', type = 'unsigned'}
})
---
...
```

Создадим первичный и вторичный индексы
```sh
> box.space.avia_tickets:create_index('indx_avia_tickets_primary', {parts = {'id'}})
---
- unique: true
  parts:
  - fieldno: 1
    sort_order: asc
    type: unsigned
    exclude_null: false
    is_nullable: false
  hint: true
  id: 0
  type: TREE
  space_id: 512
  name: indx_avia_tickets_primary
...

> box.space.avia_tickets:create_index('indx_avia_tickets_secondary', {parts = {'departure_date', 'airline','departure_city'}})
---
- unique: true
  parts:
  - fieldno: 3
    sort_order: asc
    type: string
    exclude_null: false
    is_nullable: false
  - fieldno: 2
    sort_order: asc
    type: string
    exclude_null: false
    is_nullable: false
  - fieldno: 4
    sort_order: asc
    type: string
    exclude_null: false
    is_nullable: false
  hint: true
  id: 1
  type: TREE
  space_id: 512
  name: indx_avia_tickets_secondary
...
```
Вставим несколько записей
```sh
box.space.avia_tickets:insert{1, 'S7', '01.01.2025','Moscow','Perm',2000}
box.space.avia_tickets:insert{2, 'Pobeda','01.01.2025','Moscow','Ufa',3000}
box.space.avia_tickets:insert{3, 'UTAir','01.01.2025','Moscow','Tomsk',4000}
box.space.avia_tickets:insert{4, 'Aeroflot', '01.01.2025','Moscow','Saint-Petersburg',2500}
box.space.avia_tickets:insert{5, 'Red Wings','02.01.2025','Ekaterinburg','Kazan',3000}
box.space.avia_tickets:insert{6, 'Pobeda','02.01.2025','Ufa','Irkuts',5000}
box.space.avia_tickets:insert{7, 'Aeroflot','03.01.2025','Moscow','Saint-Petersburg',2000}
box.space.avia_tickets:insert{8, 'UTAir','03.01.2025','Moscow','Samara',1500}

> box.space.avia_tickets:select()
---
- - [1, 'S7', '01.01.2025', 'Moscow', 'Perm', 2000]
  - [2, 'Pobeda', '01.01.2025', 'Moscow', 'Ufa', 3000]
  - [3, 'UTAir', '01.01.2025', 'Moscow', 'Tomsk', 4000]
  - [4, 'Aeroflot', '01.01.2025', 'Moscow', 'Saint-Petersburg', 2500]
  - [5, 'Red Wings', '02.01.2025', 'Ekaterinburg', 'Kazan', 3000]
  - [6, 'Pobeda', '02.01.2025', 'Ufa', 'Irkuts', 5000]
  - [7, 'Aeroflot', '03.01.2025', 'Moscow', 'Saint-Petersburg', 2000]
  - [8, 'UTAir', '03.01.2025', 'Moscow', 'Samara', 1500]
...


#Исправим опечатку
> box.space.avia_tickets:update({6},{{'=',5,'Irkutsk'}})
---
- [6, 'Pobeda', '02.01.2025', 'Ufa', 'Irkutsk', 5000]

```
Посмотреть все рейсы на 01.01.2025 по вторичному индексу
```sh
> box.space.avia_tickets.index.indx_avia_tickets_secondary:select('01.01.2025')
---
- - [4, 'Aeroflot', '01.01.2025', 'Moscow', 'Saint-Petersburg', 2500]
  - [2, 'Pobeda', '01.01.2025', 'Moscow', 'Ufa', 3000]
  - [1, 'S7', '01.01.2025', 'Moscow', 'Perm', 2000]
  - [3, 'UTAir', '01.01.2025', 'Moscow', 'Tomsk', 4000]
```
Запрос для выборки минимальной стоимости авиабилета на рейсы с датой вылета 01.01.2025. Переключимся на язык sql
```sh
\set language sql

> select * from "avia_tickets" where "departure_date" == '01.01.2025' order by "min_price" limit 1
---
- metadata:
  - name: id
    type: unsigned
  - name: airline
    type: string
  - name: departure_date
    type: string
  - name: departure_city
    type: string
  - name: 'arrival_city '
    type: string
  - name: min_price
    type: unsigned
  rows:
  - [1, 'S7', '01.01.2025', 'Moscow', 'Perm', 2000]
...

```
Cписок рейсов с минимальной стоимостью билета менее 3000 рублей можно сделать через итератор
```sh
\set language lua

> box.space.avia_tickets.index.indx_avia_tickets_secondary:pairs({}, {iterator = 'REQ'}):filter(function(x) return x.min_price<3000 end):totable()
---
- - [8, 'UTAir', '03.01.2025', 'Moscow', 'Samara', 1500]
  - [7, 'Aeroflot', '03.01.2025', 'Moscow', 'Saint-Petersburg', 2000]
  - [1, 'S7', '01.01.2025', 'Moscow', 'Perm', 2000]
  - [4, 'Aeroflot', '01.01.2025', 'Moscow', 'Saint-Petersburg', 2500]
...

```
