## Запуск клика


docker-compose

    version: '3'

    services:
    clickhouse:
        container_name: click
        image: clickhouse/clickhouse-server
        ports:
        - 8123:8123
        - 9000:9000
        volumes:
        - type: bind
        target: /var/lib/clickhouse/
        source: ./click-data



