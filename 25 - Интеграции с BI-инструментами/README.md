# Домашнее задание

## Подготовка сервисов

В данной работе нам необходимо два отдельных сервиса: ClickHouse и Apache Superset.

Будем использовать следующий [`docker-compose.yml`](./docker-compose.yml) файл:

```yml
version: "3.8"

services:
  clickhouse:
    image: clickhouse/clickhouse-server
    container_name: clickhouse
    ports:
      - "8123:8123"
      - "9000:9000"
    environment:
      - CLICKHOUSE_PASSWORD=123

  superset:
    image: apache/superset
    container_name: superset
    ports:
      - "8088:8088"
    environment:
      SUPERSET_SECRET_KEY: '123'
    depends_on:
      - clickhouse
```

Запустим сервисы:

```sh
docker-compose up -d
```

## Инициализация Apache Superset

Подключимся к контейнеру Superset и выполним следующие команды для инициализации:

```sh
docker exec -it superset superset fab create-admin \
  --username admin \
  --firstname admin \
  --lastname admin \
  --email admin@local \
  --password admin

docker exec -it superset superset db upgrade
docker exec -it superset superset init
```

![a](images/a.png)

## Установка драйвера ClickHouse для Superset

Подключимся к контейнеру Superset, выполним установку драйвера ClickHouse и перезапустим контейнер с Superset:

```sh
docker exec -it --user root superset bash
./.venv/bin/python -m ensurepip
./.venv/bin/python -m pip install clickhouse-connect
exit
docker restart superset
```

![b](images/b.png)

## Настройка подключения Superset к ClickHouse

Перейдем в интерфейс Apache Superset по адресу <http://localhost:8088>.

![c](images/c.png)

Авторизуемся используя учётные данные пользователя admin которого мы создали на этапе подготовки Superset.

Перейдем в раздел **Settings** / **Database Connections**

![d](images/d.png)

Добавим подключение к ClickHouse, используя следующую строку подключения:

```txt
clickhousedb://default:123@clickhouse:8123/default
```

![e](images/e.png)

## Наполнение данными

Возьмём датасет [Amazon Customer Review](https://clickhouse.com/docs/getting-started/example-datasets/amazon-reviews)

Подготовим таблицу:

```sql
CREATE DATABASE amazon

CREATE TABLE amazon.amazon_reviews
(
    `review_date` Date,
    `marketplace` LowCardinality(String),
    `customer_id` UInt64,
    `review_id` String,
    `product_id` String,
    `product_parent` UInt64,
    `product_title` String,
    `product_category` LowCardinality(String),
    `star_rating` UInt8,
    `helpful_votes` UInt32,
    `total_votes` UInt32,
    `vine` Bool,
    `verified_purchase` Bool,
    `review_headline` String,
    `review_body` String,
    PROJECTION helpful_votes
    (
        SELECT *
        ORDER BY helpful_votes
    )
)
ENGINE = MergeTree()
ORDER BY (review_date, product_category)
```

Наполним таблицу данными:

```sql
INSERT INTO amazon.amazon_reviews SELECT *
FROM s3('https://datasets-documentation.s3.eu-west-3.amazonaws.com/amazon_reviews/amazon_reviews_*.snappy.parquet')
```

## Создание датасета в Superset

Перейдем в раздел *Datasets** и добавим новый датасет на основе таблицы `amazon_reviews` из ClickHouse.

![f](images/f.png)

## Создание графиков и дашборда

В разделе **Dashboards** создадим пустой дашборд.

В разделе **Charts** создадим графики и добавим их на дашборд:

![g](images/g.png)

Итоговый дашборд:

![h](images/h.png)
