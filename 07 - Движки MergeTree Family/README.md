# Домашнее задание

1. По заданным описаниям таблиц и вставки данных определить используемый движок.
2. Заполнить пропуски, запустить код
3. Сравнить полученный вывод и результат из условия

## Пример 1

```sql
CREATE TABLE tbl1
(
    UserID UInt64,
    PageViews UInt8,
    Duration UInt8,
    Sign Int8,
    Version UInt8
)
ENGINE = VersionedCollapsingMergeTree(Sign, Version)
ORDER BY UserID;

INSERT INTO tbl1 VALUES (4324182021466249494, 5, 146, -1, 1);
INSERT INTO tbl1 VALUES (4324182021466249494, 5, 146, 1, 1),(4324182021466249494, 6, 185, 1, 2);
```

Ответ: VersionedCollapsingMergeTree. Это можно понять по колонке Sign и Version, а так же по удалению первой строки после вставки третьей с противоположным Sign и тем же Version.

![1](./images/1.png)

## Пример 2

```sql
CREATE TABLE tbl2
(
    key UInt32,
    value UInt32
)
ENGINE = SummingMergeTree()
ORDER BY key;

INSERT INTO tbl2 Values(1,1),(1,2),(2,1);
```

Ответ: SummingMergeTree. Так как мы видим, что строки схлопываются по ключу, но при этом значения остальных колонок суммируются.

![2](./images/2.png)

## Пример 3

```sql
CREATE TABLE tbl3
(
    `id` Int32,
    `status` String,
    `price` String,
    `comment` String
)
ENGINE = ReplacingMergeTree
PRIMARY KEY (id)
ORDER BY (id, status);

INSERT INTO tbl3 VALUES (23, 'success', '1000', 'Confirmed');
INSERT INTO tbl3 VALUES (23, 'success', '2000', 'Cancelled'); 

SELECT * from tbl3 WHERE id=23;
SELECT * from tbl3 FINAL WHERE id=23;
```

Ответ: ReplacingMergeTree. Так как мы видим, что для одного и того же первичного ключа остаются только новые строки.

![3](./images/3.png)

## Пример 4

```sql
CREATE TABLE tbl4
(   CounterID UInt8,
    StartDate Date,
    UserID UInt64
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(StartDate) 
ORDER BY (CounterID, StartDate);

INSERT INTO tbl4 VALUES(0, '2019-11-11', 1);
INSERT INTO tbl4 VALUES(1, '2019-11-12', 1);

CREATE TABLE tbl5
(   CounterID UInt8,
    StartDate Date,
    UserID AggregateFunction(uniq, UInt64)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(StartDate) 
ORDER BY (CounterID, StartDate);

INSERT INTO tbl5
select CounterID, StartDate, uniqState(UserID)
from tbl4
group by CounterID, StartDate;

INSERT INTO tbl5 VALUES (1,'2019-11-12',1);

SELECT uniqMerge(UserID) AS state 
FROM tbl5 
GROUP BY CounterID, StartDate;

SELECT * FROM tbl6;

SELECT * FROM tbl6 FINAL;
```

Ответ: MergeTree и AggregatingMergeTree. Это можно понять по `AggregateFunction` в колонках таблицы.

![4](./images/4.png)

## Пример 5

```sql
CREATE TABLE tbl6
(
    `id` Int32,
    `status` String,
    `price` String,
    `comment` String,
    `sign` Int8
)
ENGINE = CollapsingMergeTree(sign)
PRIMARY KEY (id)
ORDER BY (id, status);

INSERT INTO tbl6 VALUES (23, 'success', '1000', 'Confirmed', 1);
INSERT INTO tbl6 VALUES (23, 'success', '1000', 'Confirmed', -1), (23, 'success', '2000', 'Cancelled', 1);

SELECT * FROM tbl6;

SELECT * FROM tbl6 FINAL;
```

Ответ: CollapsingMergeTree. Так как записи с противоположным значением в колонке sing схлопываются и при этом нет версии.

![5](./images/5.png)
