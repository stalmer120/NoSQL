# Базовые возможности mongodb. HomeWork

## 1. установить MongoDB одним из способов: ВМ, докер; ##

**Были развернуты 3 ВМ**

| VM | IP | Name | OS | mongodb |
| ----- | ----- | ----- | ----- | ----- |
| 1 | 192.168.50.4 | mongo1 | Ubuntu 20.04.6 LTS | 7.0.14 |
| 2 | 192.168.50.5 | mongo2 | Ubuntu 20.04.6 LTS | 7.0.14 |
| 3 | 192.168.50.6 | mongo3 | Ubuntu 20.04.6 LTS | 7.0.14 |

Схема *Primary - Secondaries - Hiddens*

>rs.conf()
```js
{
  _id: 'rs01',
  version: 6,
  term: 6,
  members: [
    {
      _id: 0,
      host: '192.168.50.4:27017',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    },
    {
      _id: 1,
      host: '192.168.50.6:27017',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: false,
      priority: 1,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    },
    {
      _id: 2,
      host: '192.168.50.5:27017',
      arbiterOnly: false,
      buildIndexes: true,
      hidden: true,
      priority: 0,
      tags: {},
      secondaryDelaySecs: Long('0'),
      votes: 1
    }
  ],
  protocolVersion: Long('1'),
  writeConcernMajorityJournalDefault: true,
  settings: {
    chainingAllowed: true,
    heartbeatIntervalMillis: 2000,
    heartbeatTimeoutSecs: 10,
    electionTimeoutMillis: 10000,
    catchUpTimeoutMillis: -1,
    catchUpTakeoverDelayMillis: 30000,
    getLastErrorModes: {},
    getLastErrorDefaults: { w: 1, wtimeout: 0 },
    replicaSetId: ObjectId('66ea14d6c4d64c76deee06ba')
  }
}
```

>rs.status()

```js
  members: [
    {
      _id: 0,
      name: '192.168.50.4:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 40954,
      optime: { ts: Timestamp({ t: 1726655448, i: 1 }), t: Long('6') },
      optimeDate: ISODate('2024-09-18T10:30:48.000Z'),
      lastAppliedWallTime: ISODate('2024-09-18T10:30:48.984Z'),
      lastDurableWallTime: ISODate('2024-09-18T10:30:48.984Z'),
      syncSourceHost: '192.168.50.6:27017',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 6,
      configTerm: 6,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: '192.168.50.6:27017',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 16106,
      optime: { ts: Timestamp({ t: 1726655448, i: 1 }), t: Long('6') },
      optimeDurable: { ts: Timestamp({ t: 1726655448, i: 1 }), t: Long('6') },
      optimeDate: ISODate('2024-09-18T10:30:48.000Z'),
      optimeDurableDate: ISODate('2024-09-18T10:30:48.000Z'),
      lastAppliedWallTime: ISODate('2024-09-18T10:30:48.984Z'),
      lastDurableWallTime: ISODate('2024-09-18T10:30:48.984Z'),
      lastHeartbeat: ISODate('2024-09-18T10:30:52.599Z'),
      lastHeartbeatRecv: ISODate('2024-09-18T10:30:50.964Z'),
      pingMs: Long('0'),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1726639355, i: 1 }),
      electionDate: ISODate('2024-09-18T06:02:35.000Z'),
      configVersion: 6,
      configTerm: 6
    },
    {
      _id: 2,
      name: '192.168.50.5:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 16106,
      optime: { ts: Timestamp({ t: 1726655448, i: 1 }), t: Long('6') },
      optimeDurable: { ts: Timestamp({ t: 1726655448, i: 1 }), t: Long('6') },
      optimeDate: ISODate('2024-09-18T10:30:48.000Z'),
      optimeDurableDate: ISODate('2024-09-18T10:30:48.000Z'),
      lastAppliedWallTime: ISODate('2024-09-18T10:30:48.984Z'),
      lastDurableWallTime: ISODate('2024-09-18T10:30:48.984Z'),
      lastHeartbeat: ISODate('2024-09-18T10:30:52.599Z'),
      lastHeartbeatRecv: ISODate('2024-09-18T10:30:52.416Z'),
      pingMs: Long('1'),
      lastHeartbeatMessage: '',
      syncSourceHost: '192.168.50.6:27017',
      syncSourceId: 1,
      infoMessage: '',
      configVersion: 6,
      configTerm: 6
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1726655448, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('BFCsJTeOz9nzHXxZxOaUd8aQm04=', 0),
      keyId: Long('7415762645774499847')

```

## 2. заполнить данными; ##

Создана БД countries-big и коллекция stb1

С github был скачан countries-big.json для теста.

https://github.com/ozlerhakan/mongodb-json-files/blob/master/datasets/countries-big.json

при помощи mongoimport json импортирован в коллекцию.

mongoimport --authenticationDatabase=admin --username mongo-root --password passw0rd --db countries-big --collection stb1 --file /vagrant/ansible/countries-big.json

>db.stats()
```js
{
  db: 'countries-big',
  collections: Long('1'),
  views: Long('0'),
  objects: Long('21640'),
  avgObjSize: 89.38632162661737,
  dataSize: 1934320,
  storageSize: 815104,
  indexes: Long('1'),
  indexSize: 561152,
  totalSize: 1376256,
  scaleFactor: Long('1'),
  fsUsedSize: 3704492032,
  fsTotalSize: 41555521536,
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1726655629, i: 1 }),
    signature: {
      hash: Binary.createFromBase64('MoyC5UeWbcsgx+xSC/QQDa1cJeg=', 0),
      keyId: Long('7415762645774499847')
    }
  },
  operationTime: Timestamp({ t: 1726655629, i: 1 })
}

```

## 3. написать несколько запросов на выборку и обновление данных ##

*  **find**

>db.stb1.find().limit(3)
```js
[
  {
    _id: ObjectId('55a0f1d420a4d760b5fbdbd7'),
    'Country Name': 'Oseanië',
    Language: 'af',
    'ISO': 0
  },
  {
    _id: ObjectId('55a0f1d420a4d760b5fbdbd6'),
    'Country Name': 'Afrika',
    Language: 'af',
    'ISO': 0
  },
  {
    _id: ObjectId('55a0f1d420a4d760b5fbdbd8'),
    'Country Name': 'Suid-Amerika',
    Language: 'af',
    'ISO': 0
  }
]
```

>db.stb1.find({'Country Name': 'Россия'})
```js
[
  {
    _id: ObjectId('55a0f1d420a4d760b5fc1f04'),
    'Country Name': 'Россия',
    Language: 'ru',
    'ISO': 'RU'
  },
  {
    _id: ObjectId('55a0f1d420a4d760b5fc1f15'),
    'Country Name': 'Россия',
    Language: 'tt',
    'ISO': 'RU'
  },
  {
    _id: ObjectId('55a0f1d420a4d760b5fc1f18'),
    'Country Name': 'Россия',
    Language: 'uz',
    'ISO': 'RU'
  }
]
```

*  **insert**

> [!NOTE]
>*Insert на primary чтение с Hidden*

>db.stb1.find(db.stb1.insertMany([ {"Country Name":"Мумитролия","Language":"ru","ISO":"MU"}, {"Country Name":"Muumimaailma","Language":"fi","ISO":"MU"}, {"Country Name":"Mumintroll","Language":"sv","ISO":"MU"} ])
```js
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId('66eabd0a2332c81490964033'),
    '1': ObjectId('66eabd0a2332c81490964034'),
    '2': ObjectId('66eabd0a2332c81490964035')
  }
}
```

>db.stb1.find(db.stb1.find({"ISO":"MU"}))
```js
db.stb1.find({"ISO":"MU"})
[
  {
    _id: ObjectId('66eabd0a2332c81490964033'),
    'Country Name': 'Мумитролия',
    Language: 'ru',
    ISO: 'MU'
  },
  {
    _id: ObjectId('66eabd0a2332c81490964034'),
    'Country Name': 'Muumimaailma',
    Language: 'fi',
    ISO: 'MU'
  },
  {
    _id: ObjectId('66eabd0a2332c81490964035'),
    'Country Name': 'Mumintroll',
    Language: 'sv',
    ISO: 'MU'
  }
]

```

*  **Update**

>db.stb1.updateOne({'Country Name': 'Mumintroll'},{$set: {'Country Name': 'ムーミン', "Language":"ja"}})
```js
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
```

>db.stb1.find({"ISO":"MU"})
```js
[
  {
    _id: ObjectId('66eabd0a2332c81490964033'),
    'Country Name': 'Мумитролия',
    Language: 'ru',
    ISO: 'MU'
  },
  {
    _id: ObjectId('66eabd0a2332c81490964034'),
    'Country Name': 'Muumimaailma',
    Language: 'fi',
    ISO: 'MU'
  },
  {
    _id: ObjectId('66eabd0a2332c81490964035'),
    'Country Name': 'ムーミン',
    Language: 'ja',
    ISO: 'MU'
  }
]
```
