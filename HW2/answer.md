# ДЗ2

## Установка и подготовка

- Ставил mongo в docker через docker-compose.yaml (рядом в папке лежит)

Логин и пароль задаются с помощью environment

     environment:
      - MONGO_INITDB_ROOT_USERNAME=user
      - MONGO_INITDB_ROOT_PASSWORD=pass

Запуск

    d.yakovlev@MacBook-Pro-X5 ~ % docker compose up -d

Ожидание запуска

    [+] Running 2/2
     ⠿ Container hw2-mongodb-1       Started
     ⠿ Container hw2-myapplication-1  Started       


Далее ставил mongosh

    d.yakovlev@MacBook-Pro-X5 ~ % brew install mongosh

## Подключение к БД

    d.yakovlev@MacBook-Pro-X5 ~ % mongosh

Пробуем вывести список БД

    test> show dbs
    MongoServerError: command listDatabases requires authentication

Переподключаемся под пользователем, который был задан в настройках

    d.yakovlev@MacBook-Pro-X5 ~ % mongosh -u user -p pass

Еще раз пробуем вывести список БД

    test> show dbs
    admin   100.00 KiB
    config   60.00 KiB
    local    72.00 KiB

## Работа с данными

Создаем коллекцию

    test> db.createCollection("orders")
    { ok: 1 }

Переключаемся в нее

    use orders
    switched to db orders
    orders> 

Делаем простые вставки на придуманных данных

    orders> db.orders.insert([{
    ...     "order_id": 1,
    ...     "shop_id": 10,
    ...     "user_id": 12300,
    ...     "items" : [
    ...         {
    ...             "id": 1,
    ...             "name": "картофель"
    ...         },
    ...         {
    ...             "id": 2,
    ...             "name": "чипсы Lays"
    ...         },
    ...         {
    ...             "id": 3,
    ...             "name": "молоко Простоквашино"
    ...         }
    ...         
    ...     ]
    ... 
    ... },
    ... {
    ...     "order_id": 2,
    ...     "shop_id": 10,
    ...     "user_id": 12300,
    ...     "items" : [
    ...         {
    ...             "id": 1,
    ...             "name": "картофель"
    ...         },
    ...         {
    ...             "id": 4,
    ...             "name": "печенье"
    ...         },
    ...         {
    ...             "id": 5,
    ...             "name": "конфеты"
    ...         }
    ...         
    ...     ]
    ... 
    ... },
    ... {
    ...     "order_id": 1,
    ...     "shop_id": 10,
    ...     "user_id": 12300,
    ...     "items" : [
    ...         {
    ...             "id": 4,
    ...             "name": "печенье"
    ...         },
    ...         {
    ...             "id": 6,
    ...             "name": "чипсы Pringles"
    ...         },
    ...         {
    ...             "id": 3,
    ...             "name": "молоко Вкуснотеево"
    ...         }
    ...         
    ...     ]
    ... 
    ... }]
    ... )
    DeprecationWarning: Collection.insert() is deprecated. Use insertOne, insertMany, or bulkWrite.
    {
        acknowledged: true,
        insertedIds: {
            '0': ObjectId("643a5343f8d5092f7dc617b9"),
            '1': ObjectId("643a5343f8d5092f7dc617ba"),
            '2': ObjectId("643a5343f8d5092f7dc617bb")
        }
    }
    orders> 


Достаем все вставленные элементы 

    orders> db.orders.find()
    [
        {       
            _id: ObjectId("643a5343f8d5092f7dc617b9"),
            order_id: 1,
            shop_id: 10,
            user_id: 12300,
            items: [
            { id: 1, name: 'картофель' },
            { id: 2, name: 'чипсы Lays' },
            { id: 3, name: 'молоко Простоквашино' }
            ]
        },
        {
            _id: ObjectId("643a5343f8d5092f7dc617ba"),
            order_id: 2,
            shop_id: 10,
            user_id: 12300,
            items: [
            { id: 1, name: 'картофель' },
            { id: 4, name: 'печенье' },
            { id: 5, name: 'конфеты' }
            ]
        },
        {
            _id: ObjectId("643a5343f8d5092f7dc617bb"),
            order_id: 1,
            shop_id: 10,
            user_id: 12300,
            items: [
            { id: 4, name: 'печенье' },
            { id: 6, name: 'чипсы Pringles' },
            { id: 3, name: 'молоко Вкуснотеево' }
            ]
        }   
    ]
    orders> 

Поиск заказов, где есть картофель

    orders> db.orders.find({"items.name":"картофель"})
    [
        {
            _id: ObjectId("643a5343f8d5092f7dc617b9"),
            order_id: 1,
            shop_id: 10,
            user_id: 12300,
            items: [
            { id: 1, name: 'картофель' },
            { id: 2, name: 'чипсы Lays' },
            { id: 3, name: 'молоко Простоквашино' }
            ]
        },
        {
            _id: ObjectId("643a5343f8d5092f7dc617ba"),
            order_id: 2,
            shop_id: 10,
            user_id: 12300,
            items: [
            { id: 1, name: 'картофель' },
            { id: 4, name: 'печенье' },
            { id: 5, name: 'конфеты' }
            ]
        }
    ]

Количество заказов содержащих печенье 

    orders> db.orders.count({"items.name" : "печенье"})
    2

Обновим order_id в заказе 3, так как в изначальных данных он равен 1

    orders> db.orders.updateOne({"_id": ObjectId("643a5343f8d5092f7dc617bb")}, {$set: {"order_id": 3}})
    {
        acknowledged: true,
        insertedId: null,
        matchedCount: 1,
        modifiedCount: 1,
        upsertedCount: 0
    }

И обновим по новому order_id user_id

    orders> db.orders.updateOne({"order_id": 3}, {$set: {"user_id": 12301}})
    {
        acknowledged: true,
        insertedId: null,
        matchedCount: 1,
        modifiedCount: 1,
        upsertedCount: 0
    }

Увидим, что все изменения сработали

    orders> db.orders.find({"order_id":3})
    [
        {
            _id: ObjectId("643a5343f8d5092f7dc617bb"),
            order_id: 3,
            shop_id: 10,
            user_id: 12301,
            items: [
            { id: 4, name: 'печенье' },
            { id: 6, name: 'чипсы Pringles' },
            { id: 3, name: 'молоко Вкуснотеево' }
            ]
        }
    ]

Посчитаем, сколько заказов у каких пользователей 

    orders> db.orders.aggregate([ {$group: {_id: "$user_id", number_of_orders: {$sum: 1}}}])
    [
        { _id: 12300, number_of_orders: 2 },
        { _id: 12301, number_of_orders: 1 }
    ]

## Проверка работы индексов

Создадим новую коллекцию

    orders> db.createCollection("mail")
    { ok: 1 }

Через MongoDB Compass импортируем данные из датасета взятые [отсюда](https://www.kaggle.com/datasets/shwetabh123/mall-customers)

Пробуем запросить данные 

    orders> db.mail.find({})
    [
    {
        _id: ObjectId("643bfa871c072dfe4a1dd01f"),
        CustomerID: 1,
        Genre: 'Male',
        Age: 19,
        'Annual Income (k$)': 15
    },
    {
        _id: ObjectId("643bfa871c072dfe4a1dd020"),
        CustomerID: 2,
        Genre: 'Male',
        Age: 21,
        'Annual Income (k$)': 15
    },
    {
        _id: ObjectId("643bfa871c072dfe4a1dd021"),
        CustomerID: 3,
        Genre: 'Female',
        Age: 20,
        'Annual Income (k$)': 16
    },
    ...

Делаем поиск по полю Age и делаем explain запроса

    orders> db.mail.find({"Age": {$gt: 50}}).explain()
    {
        explainVersion: '1',
        queryPlanner: {
            namespace: 'orders.mail',
            indexFilterSet: false,
            parsedQuery: { Age: { '$gt': 50 } },
            queryHash: 'A4552BB7',
            planCacheKey: 'A4552BB7',
            maxIndexedOrSolutionsReached: false,
            maxIndexedAndSolutionsReached: false,
            maxScansToExplodeReached: false,
            winningPlan: {
                stage: 'COLLSCAN',
                filter: { Age: { '$gt': 50 } },
                direction: 'forward'
            },
            rejectedPlans: []
    },
    command: { find: 'mail', filter: { Age: { '$gt': 50 } }, '$db': 'orders' },
    serverInfo: {
        host: 'c4719cc4cdbd',
        port: 27017,
        version: '6.0.5',
        gitVersion: 'c9a99c120371d4d4c52cbb15dac34a36ce8d3b1d'
    },
    serverParameters: {
        internalQueryFacetBufferSizeBytes: 104857600,
        internalQueryFacetMaxOutputDocSizeBytes: 104857600,
        internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
        internalDocumentSourceGroupMaxMemoryBytes: 104857600,
        internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
        internalQueryProhibitBlockingMergeOnMongoS: 0,
        internalQueryMaxAddToSetBytes: 104857600,
        internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
    },
    ok: 1
    }

Добавим индекс на поле Age

    orders> db.mail.createIndex({"Age" : 1})
    Age_1

Попробуем выполнить тот же запрос

    orders> db.mail.find({"Age": {$gt: 50}}).explain()
    {
        explainVersion: '1',
        queryPlanner: {
            namespace: 'orders.mail',
            indexFilterSet: false,
            parsedQuery: { Age: { '$gt': 50 } },
            queryHash: 'A4552BB7',
            planCacheKey: 'B32C835B',
            maxIndexedOrSolutionsReached: false,
            maxIndexedAndSolutionsReached: false,
            maxScansToExplodeReached: false,
            winningPlan: {
            stage: 'FETCH',
            inputStage: {
                stage: 'IXSCAN',
                keyPattern: { Age: 1 },
                indexName: 'Age_1',
                isMultiKey: false,
                multiKeyPaths: { Age: [] },
                isUnique: false,
                isSparse: false,
                isPartial: false,
                indexVersion: 2,
                direction: 'forward',
                indexBounds: { Age: [ '(50, inf.0]' ] }
            }
            },
            rejectedPlans: []
        },
        command: { find: 'mail', filter: { Age: { '$gt': 50 } }, '$db': 'orders' },
        serverInfo: {
            host: 'c4719cc4cdbd',
            port: 27017,
            version: '6.0.5',
            gitVersion: 'c9a99c120371d4d4c52cbb15dac34a36ce8d3b1d'
        },
        serverParameters: {
            internalQueryFacetBufferSizeBytes: 104857600,
            internalQueryFacetMaxOutputDocSizeBytes: 104857600,
            internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
            internalDocumentSourceGroupMaxMemoryBytes: 104857600,
            internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
            internalQueryProhibitBlockingMergeOnMongoS: 0,
            internalQueryMaxAddToSetBytes: 104857600,
            internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600
        },
        ok: 1
    }


Как видим происходит поиск по индексу

    winningPlan: {
        stage: 'FETCH',
        inputStage: {
            stage: 'IXSCAN',
            keyPattern: { Age: 1 },
            indexName: 'Age_1',
            isMultiKey: false,
            multiKeyPaths: { Age: [] },
            isUnique: false,
            isSparse: false,
            isPartial: false,
            indexVersion: 2,
            direction: 'forward',
            indexBounds: { Age: [ '(20, inf.0]' ] }
        }
    },

Результаты в MongoDB Compass

Без индекса 

    {
        "stage": "COLLSCAN",
        "filter": {
        "Age": {
        "$gte": 50
        }
        },
        "nReturned": 45,
        "executionTimeMillisEstimate": 0,
        "works": 202,
        "advanced": 45,
        "needTime": 156,
        "needYield": 0,
        "saveState": 0,
        "restoreState": 0,
        "isEOF": 1,
        "direction": "forward",
        "docsExamined": 200
    }

C индексом

    {
        "stage": "FETCH",
        "nReturned": 45,
        "executionTimeMillisEstimate": 0,
        "works": 46,
        "advanced": 45,
        "needTime": 0,
        "needYield": 0,
        "saveState": 0,
        "restoreState": 0,
        "isEOF": 1,
        "docsExamined": 45,
        "alreadyHasObj": 0
    }

Как видно в первом случае без индекса

    "docsExamined": 200

А во-втором 

    "docsExamined": 45

Таким образом использование индексов ускоряет процесс, так как позволяет не проходить все документы подряд, а идти по индексу