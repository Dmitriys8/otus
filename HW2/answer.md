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
