

# Couchbase

## Запуск 1 ноды


    d.yakovlev@MacBook-Pro-X5 HW4 % docker compose up -d
    [+] Running 15/15
    ⠿ couchbase Pulled              29.0s
    ⠿ c60055a51d74 Pull complete    11.4s
    ⠿ 755da0cdb7d2 Pull complete    11.5s
    ⠿ 969d017f67e6 Pull complete    11.5s
    ⠿ 37c9a9113595 Pull complete    11.6s
    ⠿ a3d9f8479786 Pull complete    11.7s
    ⠿ 1e2d8c867794 Pull complete    12.3s
    ⠿ d388cb7083d3 Pull complete    12.4s
    ⠿ b47800ec90de Pull complete    25.7s
    ⠿ e333ecf73262 Pull complete    25.8s
    ⠿ 07291fb18005 Pull complete    25.8s
    ⠿ c927f04411f1 Pull complete    25.9s
    ⠿ bf7f6ce74683 Pull complete    26.0s
    ⠿ 4e204879a4a0 Pull complete    26.0s
    ⠿ 0126d05103b4 Pull complete    26.1s
    [+] Running 2/2
    ⠿ Network hw4_default        Created   0.0s
    ⠿ Container hw4-couchbase-1  Started   1.2s

Прверяем, что работает

    d.yakovlev@MacBook-Pro-X5 HW4 % docker ps -a
    CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                   NAMES
    d3185a0ded35   arungupta/couchbase   "/entrypoint.sh /opt…"   8 seconds ago   Up 6 seconds   0.0.0.0:8091-8094->8091-8094/tcp, 11207/tcp, 11211/tcp, 0.0.0.0:11210->11210/tcp, 18091-18093/tcp   hw4-couchbase-1

Заходим на localhost:8091, вводим логин и пароль. Ура, сервер ответил.

## Кластер 

Добавим еще нод

    couchbase-replica-1:
        image: arungupta/couchbase
        deploy:
            replicas: 1
        ports:
            - 9091:8091
            - 9092:8092
            - 9093:8093
            - 9094:8094
            - 12210:11210    

    couchbase-replica-3:
        image: arungupta/couchbase
            deploy:
                replicas: 1
        ports:
            - 10091:8091
            - 10092:8092
            - 10093:8093
            - 10094:8094
            - 13210:11210  

Заходим в их UI, смотрим ip нод:

    172.23.0.2 - первая реплика
    172.23.0.4 - вторая реплика

Добавляем через UI сервера, затем делаем rebalance
Все узлы добавлены в кластер

## Пороняем ноды
Уроним одну ноду

    d.yakovlev@MacBook-Pro-X5 HW4 % docker ps -a | grep replica-1
    858837f89e1e   arungupta/couchbase   "/entrypoint.sh /opt…"   12 minutes ago   Up 11 minutes   11207/tcp, 11211/tcp, 18091-18093/tcp, 0.0.0.0:9091->8091/tcp, 0.0.0.0:9092->8092/tcp, 0.0.0.0:9093->8093/tcp, 0.0.0.0:9094->8094/tcp, 0.0.0.0:12210->11210/tcp       hw4-couchbase-replica-1-1

    
    d.yakovlev@MacBook-Pro-X5 HW4 % docker stop 858837f89e1e
    858837f89e1e    

Нажимаем в UI Fail Over

Нода требует перебалансировки. После перебалансировки нода удаляется из кластера

Вернем обратно ноду

    d.yakovlev@MacBook-Pro-X5 HW4 % docker start 858837f89e1e
    858837f89e1e

Снова добавим в кластер, нода добавилась

Остановим еще раз ноду

    d.yakovlev@MacBook-Pro-X5 HW4 % docker stop 858837f89e1e
    858837f89e1e    

Нажмем fail over и сразу запустим ноду обратно

    d.yakovlev@MacBook-Pro-X5 HW4 % docker start 858837f89e1e
    858837f89e1e

В UI видим, что сервер снова виден и есть предолжение о восстановлении 

Нажимаем rebalance и нода возвращается в кластер

## Отказоустойчивость и данные

Создадим bucket, сделаем фактор репликации 2, так как у нас 3 ноды.

Создаим индекс 

    CREATE PRIMARY INDEX ON test

Вставим данные 

    INSERT INTO test (KEY, VALUE)
    VALUES ("1", {"name":"Dima"}),
    VALUES ("2", {"name":"Jane"}),
    VALUES ("3", {"name":"Dex"}),
    VALUES ("4", {"name":"Masha"}),
    VALUES ("5", {"name":"Petya"})

Выберем данные 

    SELECT *
    FROM test

    {
        "requestID": "228dca64-e1b9-4122-8d55-8db24eadf68b",
        "clientContextID": "833beb1e-2636-4c5d-a658-3c1b21658def",
        "signature": {
            "*": "*"
        },
        "results": [
            {
                "test": {
                    "name": "Dima"
                }
            },
            {
                "test": {
                    "name": "Jane"
                }
            },
            {
                "test": {
                    "name": "Dex"
                }
            },
            {
                "test": {
                    "name": "Masha"
                }
            },
            {
                "test": {
                    "name": "Petya"
                }
            }
        ],
        "status": "success",
        "metrics": {
            "elapsedTime": "21.185128ms",
            "executionTime": "21.127174ms",
            "resultCount": 5,
            "resultSize": 391
        }
    }

Уроним одну ноду

    d.yakovlev@MacBook-Pro-X5 HW4 % docker stop 858837f89e1e  
    858837f89e1e

Выведем ее из кластера и перебалансируем

В консоли видим предупреждение, что мало нод для нашего бакета
    
    Fail Over Warning: Additional active servers required to provide the desired number of replicas!

Но тем не менее запросы также отрабатывают

    SELECT *
    FROM test

    {
        "requestID": "428cbe37-9afa-409d-8363-ab9b3753cb11",
        "clientContextID": "225a5f38-ea69-41b3-8009-8f2ec7882414",
        "signature": {
            "*": "*"
        },
        "results": [
            {
                "test": {
                    "name": "Dima"
                }
            },
            {
                "test": {
                    "name": "Jane"
                }
            },
            {
                "test": {
                    "name": "Dex"
                }
            },
            {
                "test": {
                    "name": "Masha"
                }
            },
            {
                "test": {
                    "name": "Petya"
                }
            }
        ],
        "status": "success",
        "metrics": {
            "elapsedTime": "18.497668ms",
            "executionTime": "18.405831ms",
            "resultCount": 5,
            "resultSize": 391
        }
    }

Уроним вторую ноду

    d.yakovlev@MacBook-Pro-X5 HW4 % docker stop 4cff617be319
    4cff617be319    

Делаем запрос

    SELECT *
    FROM test

Получаем ошибку

    [
        {
            "code": 4000,
            "msg": "No index available on keyspace test that matches your query. Use CREATE INDEX or CREATE PRIMARY INDEX to create an index, or check that your expected index is online.",
            "query_from_user": "SELECT *\nFROM test"
        }
    ]

Вернем ноды обратно 

    d.yakovlev@MacBook-Pro-X5 HW4 % docker start 858837f89e1e
    858837f89e1e
    d.yakovlev@MacBook-Pro-X5 HW4 % docker start 4cff617be319
    4cff617be319

Даже после восстановления нод получаем ту же ошибку

    SELECT *
    FROM test

    [
        {
            "code": 4000,
            "msg": "No index available on keyspace test that matches your query. Use CREATE INDEX or CREATE PRIMARY INDEX to create an index, or check that your expected index is online.",
            "query_from_user": "SELECT *\nFROM test"
        }
    ]

Еще раз создаем индекс

    CREATE PRIMARY INDEX ON test    

Делаем запрос

    SELECT *
    FROM test

Получаем данные в сохранности

    {
        "requestID": "428cbe37-9afa-409d-8363-ab9b3753cb11",
        "clientContextID": "225a5f38-ea69-41b3-8009-8f2ec7882414",
        "signature": {
            "*": "*"
        },
        "results": [
            {
                "test": {
                    "name": "Dima"
                }
            },
            {
                "test": {
                    "name": "Jane"
                }
            },
            {
                "test": {
                    "name": "Dex"
                }
            },
            {
                "test": {
                    "name": "Masha"
                }
            },
            {
                "test": {
                    "name": "Petya"
                }
            }
        ],
        "status": "success",
        "metrics": {
            "elapsedTime": "18.497668ms",
            "executionTime": "18.405831ms",
            "resultCount": 5,
            "resultSize": 391
        }
    }
    

Еще раз уроним ноды

    d.yakovlev@MacBook-Pro-X5 HW4 % docker stop 858837f89e1e 
    858837f89e1e
    d.yakovlev@MacBook-Pro-X5 HW4 % docker stop 4cff617be319 
    4cff617be319

Удалим их из кластера


Еще раз создадим индекс

    CREATE PRIMARY INDEX ON test

Делаем запрос

    SELECT *
    FROM test

Данные на месте

    {
        "requestID": "428cbe37-9afa-409d-8363-ab9b3753cb11",
        "clientContextID": "225a5f38-ea69-41b3-8009-8f2ec7882414",
        "signature": {
            "*": "*"
        },
        "results": [
            {
                "test": {
                    "name": "Dima"
                }
            },
            {
                "test": {
                    "name": "Jane"
                }
            },
            {
                "test": {
                    "name": "Dex"
                }
            },
            {
                "test": {
                    "name": "Masha"
                }
            },
            {
                "test": {
                    "name": "Petya"
                }
            }
        ],
        "status": "success",
        "metrics": {
            "elapsedTime": "18.497668ms",
            "executionTime": "18.405831ms",
            "resultCount": 5,
            "resultSize": 391
        }
    }


# Выводы

Кластер очень быстро и легко восстанавливается. Легко включаются и выключаются ноды. Данные реплицируются, но при ребалансировке надо пересоздавать индексы