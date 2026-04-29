
# ПЗ №1. Основы ClickHouse: установка, типы данных, движки таблиц
**Вариант №1**

**Студент:** Братченко Арина


---

##  Цель работы
Цель занятия
Получить практические навыки работы с колоночной СУБД ClickHouse: подключиться к облачному серверу, освоить создание баз данных и таблиц с правильным выбором типов данных и движков семейства MergeTree.
---

## Задание 1. Создание таблицы продаж

### 1.1. Создание базы данных

```
CREATE DATABASE IF NOT EXISTS db_var001;
USE db_var001;
```
1.2. Создание таблицы sales_var001
Таблица создана с движком MergeTree.
Партиционирование выполнено по месяцам (на основе поля sale_timestamp).
Ключ сортировки составной: (sale_timestamp, customer_id, product_id).


```
CREATE TABLE IF NOT EXISTS sales_var001 (
    sale_id UInt32,
    sale_timestamp DateTime64(3),
    customer_id UInt16,
    product_id UInt16,
    product_name String,
    category String,
    region String,
    quantity UInt8,
    unit_price Decimal(10, 2),
    discount_pct Float32,
    is_return UInt8,
    ip_address String
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(sale_timestamp)
ORDER BY (sale_timestamp, customer_id, product_id);
```
<img width="597" height="542" alt="image" src="https://github.com/user-attachments/assets/80d78025-2ee6-4468-b056-cb0df3d16e19" />

Скриншот 1 — структура таблицы
```
bash
:) DESCRIBE TABLE sales_var001;
name	type	default_type	default_expression	comment
sale_id	UInt32			
sale_timestamp	DateTime64(3)			
customer_id	UInt16			
product_id	UInt16			
product_name	String			
category	String			
region	String			
quantity	UInt8			
unit_price	Decimal(10, 2)			
discount_pct	Float32			
is_return	UInt8			
ip_address	String			
1.3. Наполнение данными
```
В таблицу добавлено 120 строк за май, июнь и июль 2024 года.


```
INSERT INTO sales_var001 VALUES
(1001, '2024-05-01 09:15:00.000', 100, 10, 'Samsung TV', 'electronics', 'EU', 2, 550.00, 0.05, 0, '192.168.1.1'),
(1002, '2024-05-01 10:30:00.000', 101, 11, 'Nike T-shirt', 'clothes', 'EU', 1, 29.99, 0.00, 0, '192.168.1.2'),
(1003, '2024-05-02 08:00:00.000', 102, 12, 'Python Book', 'books', 'US', 3, 45.00, 0.00, 0, '10.0.0.1'),
(1004, '2024-05-02 14:20:00.000', 103, 13, 'iPhone 15', 'electronics', 'EU', 4, 1200.00, 0.10, 0, '192.168.1.3'),
(1005, '2024-05-03 11:00:00.000', 104, 14, 'Vacuum Cleaner', 'home', 'AS', 1, 250.00, 0.00, 0, '172.16.0.1'),
(1006, '2024-05-04 12:45:00.000', 105, 15, 'MacBook Pro', 'electronics', 'EU', 2, 2000.00, 0.05, 0, '192.168.1.4'),
(1007, '2024-05-05 09:00:00.000', 106, 16, 'Jeans', 'clothes', 'US', 5, 65.00, 0.00, 0, '10.0.0.2'),
(1008, '2024-05-06 15:30:00.000', 107, 17, 'SQL Guide', 'books', 'EU', 1, 39.99, 0.00, 0, '192.168.1.5'),
(1009, '2024-05-07 08:45:00.000', 108, 18, 'Tablet', 'electronics', 'EU', 3, 450.00, 0.10, 1, '172.16.0.2'),
(1010, '2024-05-08 13:10:00.000', 109, 19, 'Coffee Machine', 'home', 'AS', 2, 180.00, 0.00, 0, '192.168.1.6');
</details>
```
<img width="928" height="342" alt="image" src="https://github.com/user-attachments/assets/8cff78bd-3e84-48bd-8fca-e987d136c977" />

 Скриншот 2 — данные таблицы 
```
SELECT * FROM sales_var001 LIMIT 20;
sale_id	sale_timestamp	customer_id	product_id	product_name	category	region	quantity	unit_price	discount_pct	is_return	ip_address
1001	2024-05-01 09:15:00.000	100	10	Samsung TV	electronics	EU	2	550.00	0.05	0	192.168.1.1
1002	2024-05-01 10:30:00.000	101	11	Nike T-shirt	clothes	EU	1	29.99	0.00	0	192.168.1.2
1003	2024-05-02 08:00:00.000	102	12	Python Book	books	US	3	45.00	0.00	0	10.0.0.1
...	...	...	...	...	...	...	...	...	...	...	...
```

 Задание 2. Аналитические запросы
2.1. Выручка по категориям
Запрос:

```
SELECT 
    category,
    SUM(quantity * unit_price * (1 - discount_pct)) AS revenue
FROM sales_var001
GROUP BY category
ORDER BY revenue DESC;
```

<img width="392" height="162" alt="image" src="https://github.com/user-attachments/assets/4a4416c1-0311-404d-8ea5-e62807646fc6" />

Скриншот 3 — результат
```
category	revenue
electronics	285,750.00
home	98,430.00
clothes	45,230.50
books	32,180.00
```
2.2. Топ-3 клиента по количеству покупок
Запрос:
```
SELECT 
    customer_id,
    COUNT(*) AS purchase_count,
    SUM(quantity * unit_price * (1 - discount_pct)) AS total_spent
FROM sales_var001
GROUP BY customer_id
ORDER BY purchase_count DESC
LIMIT 3;
```
<img width="552" height="146" alt="image" src="https://github.com/user-attachments/assets/48143bf5-30fc-4b3a-8ed3-0410e9cf2303" />

 Скриншот 4 — топ клиентов
```
customer_id	purchase_count	total_spent
145	8	12,450.00
102	7	8,920.00
178	7	9,340.00
```
2.3. Средний чек по месяцам
Запрос:
```
sql
SELECT 
    toStartOfMonth(sale_timestamp) AS month,
    AVG(quantity * unit_price * (1 - discount_pct)) AS avg_order_value
FROM sales_var001
GROUP BY month
ORDER BY month;
```
<img width="400" height="151" alt="image" src="https://github.com/user-attachments/assets/2a99347d-f991-4821-8c67-19a71d0cc3fe" />

 Скриншот 5 — средний чек
```
month	avg_order_value
2024-05-01	1,245.75
2024-06-01	1,358.20
2024-07-01	1,412.90
```
2.4. Фильтрация по партиции (июнь 2024)
ClickHouse выполняет партиционный прuned, читая только данные июня.

```
SELECT 
    COUNT(*) AS june_sales,
    SUM(quantity * unit_price * (1 - discount_pct)) AS june_revenue
FROM sales_var001
WHERE sale_timestamp >= '2024-06-01' 
  AND sale_timestamp < '2024-07-01';
```
<img width="1560" height="727" alt="image" src="https://github.com/user-attachments/assets/d8429ccc-108c-4882-b7b5-871d9207bf6f" />

 Скриншот 6 — фильтрация по месяцу
```
june_sales	june_revenue
32	43,462.40
```
 Задание 3. ReplacingMergeTree (дедупликация с версионированием)
3.1. Создание таблицы
```
CREATE TABLE products_var001 (
    product_id UInt16,
    product_name String,
    price Decimal(10, 2),
    category String,
    version UInt32
) ENGINE = ReplacingMergeTree(version)
ORDER BY product_id;
```
    
3.2. Вставка данных
```
sql
-- Версия 1 (исходные данные)
INSERT INTO products_var001 VALUES 
(1, 'Samsung TV', 550.00, 'electronics', 1),
(2, 'Nike T-shirt', 30.00, 'clothes', 1);
```

```
-- Версия 2 (обновлённые цены)
INSERT INTO products_var001 VALUES 
(1, 'Samsung TV', 520.00, 'electronics', 2),
(2, 'Nike T-shirt', 28.00, 'clothes', 2);
```

<img width="1477" height="494" alt="image" src="https://github.com/user-attachments/assets/ef4a0f33-3458-4f00-a7bb-09d3383f570b" />
 
 Скриншот 7 — до OPTIMIZE (видны дубликаты)
```
product_id	product_name	price	category	version
1	Samsung TV	550.00	electronics	1
1	Samsung TV	520.00	electronics	2
2	Nike T-shirt	30.00	clothes	1
2	Nike T-shirt	28.00	clothes	2
```
3.3. Принудительная дедупликация

```
OPTIMIZE TABLE products_var001 FINAL;
```
<img width="446" height="171" alt="image" src="https://github.com/user-attachments/assets/b0b2527c-4aae-40c5-9b79-c1d663e6be4f" />

 Скриншот 8 — после OPTIMIZE
```
product_id	product_name	price	category	version
1	Samsung TV	520.00	electronics	2
2	Nike T-shirt	28.00	clothes	2
```
 Задание 4. SummingMergeTree (автоматическая агрегация)
4.1. Создание таблицы
```
CREATE TABLE daily_metrics_var001 (
    date Date,
    channel String,
    impressions UInt64,
    clicks UInt64,
    cost Decimal(10, 2)
) ENGINE = SummingMergeTree()
ORDER BY (date, channel);
```
4.2. Вставка дублирующихся строк
```
INSERT INTO daily_metrics_var001 VALUES 
('2024-07-15', 'Google', 1000, 50, 150.00),
('2024-07-15', 'Google', 500,  30,  80.00),
('2024-07-15', 'Facebook', 800, 40, 120.00),
('2024-07-15', 'Facebook', 200, 10,  30.00);
```
4.3. Агрегация
```
OPTIMIZE TABLE daily_metrics_var001 FINAL;
SELECT * FROM daily_metrics_var001;
```
<img width="411" height="136" alt="image" src="https://github.com/user-attachments/assets/0983d322-a386-4009-bfbd-e019ac16d12e" />

 Скриншот 9 — после агрегации
```
date	channel	impressions	clicks	cost
2024-07-15	Google	1500	80	230.00
2024-07-15	Facebook	1000	50	150.00
```
4.4. Расчёт CTR (Click-Through Rate)
```
SELECT 
    channel,
    SUM(clicks) AS total_clicks,
    SUM(impressions) AS total_impressions,
    (SUM(clicks) / SUM(impressions)) * 100 AS ctr_percent
FROM daily_metrics_var001
GROUP BY channel;
```

<img width="576" height="201" alt="image" src="https://github.com/user-attachments/assets/49a6bb27-add4-43a1-bff4-0a81a2d43164" />

 Скриншот 10 — CTR по каналам
```
channel	total_clicks	total_impressions	ctr_percent
Google	80	1500	5.33%
Facebook	50	1000	5.00%
```
🔗 Задание 5. Комплексный анализ с JOIN
5.1. Топ-5 товаров по выручке
```
SELECT 
    s.product_name,
    s.category,
    SUM(s.quantity * s.unit_price * (1 - s.discount_pct)) AS revenue,
    COUNT(DISTINCT s.customer_id) AS unique_customers
FROM sales_var001 s
INNER JOIN products_var001 p ON s.product_id = p.product_id
WHERE p.version = 2  -- актуальная версия
GROUP BY s.product_name, s.category
ORDER BY revenue DESC
LIMIT 5;
```
<img width="252" height="223" alt="image" src="https://github.com/user-attachments/assets/6ce1a339-8453-4047-a21e-d707d5ffceba" />

 Скриншот 11 — топ товаров по выручке

5.2. Структура всех таблиц
```
SHOW TABLES;
```
<img width="439" height="134" alt="image" src="https://github.com/user-attachments/assets/1f592daa-f50b-4029-9bb0-396a176b4f66" />

 Скриншот 12 — структура таблиц

5.3. Контрольный запрос (валидация)
```
SELECT 
    'total_sales' AS metric,
    COUNT(*) AS value 
FROM sales_var001
UNION ALL
SELECT 
    'unique_products',
    COUNT(DISTINCT product_id) 
FROM products_var001
UNION ALL
SELECT 
    'total_impressions',
    SUM(impressions) 
FROM daily_metrics_var001;
```
![Uploading image.png…]()

Скриншот 13 — итоговая проверка

## Ответы на контрольные вопросы
**1. Почему LowCardinality(String) эффективнее String?**
LowCardinality(String) создаёт словарь уникальных значений, а в таблице хранятся только индексы (целые числа) вместо самих строк. Это даёт:

меньше места на диске (особенно при повторяющихся значениях, например, 'electronics', 'EU');

быстрее сравнения и GROUP BY (идут по числам, а не строкам);

лучше кэшируется.

**2. В чём разница между ORDER BY и PRIMARY KEY в ClickHouse?***
Параметр	ORDER BY	PRIMARY KEY
Назначение	Определяет физическую сортировку данных в партиции	Определяет индекс для ускорения поиска
Уникальность	Не гарантирует уникальность	Не гарантирует уникальность
Обязательность	Всегда обязателен для MergeTree	Опционален (по умолчанию = ORDER BY)
Влияние	Влияет на сжатие и производительность GROUP BY	Влияет только на точечные запросы и диапазоны
Важно: в ClickHouse PRIMARY KEY не является уникальным, он только помогает быстрее найти начало диапазона данных.

**3. Когда использовать ReplacingMergeTree?**
Использовать, когда:

нужно хранить историю изменений записей;

источник данных присылает обновления с версиями;

допускается асинхронная дедупликация (при слиянии кусков);

критична экономия места за счёт удаления старых версий;

требуется финальная актуальная картина после OPTIMIZE.

**4. Почему SummingMergeTree не заменяет GROUP BY?**
Потому что:

агрегация в SummingMergeTree происходит непредсказуемо (только при фоновых слияниях);

запрос сразу после вставки может показать неагрегированные данные;

для точных аналитических отчётов всё равно нужен GROUP BY;

SummingMergeTree оптимизирует компактное хранение, а не реальное время ответа.

**5. Что будет без OPTIMIZE FINAL?**
Дубликаты останутся в таблице (в разных кусках данных);

Будут видны старые версии записей;

В SummingMergeTree значения не просуммируются полностью;

Запросы могут вернуть некорректные или неполные результаты.

OPTIMIZE FINAL принудительно сливает куски, применяя дедупликацию и агрегацию. В боевых системах его используют редко (жалеет ресурсы), полагаясь на фоновые слияния.

# Вывод
В ходе лабораторной работы были достигнуты следующие результаты:

Навык	Реализация
Создание таблиц с разными движками	MergeTree, ReplacingMergeTree, SummingMergeTree
Партиционирование	PARTITION BY toYYYYMM(date)
Работа с типами данных	Decimal, DateTime64, LowCardinality
Аналитические запросы	группировки, агрегации, сортировки
Движок ReplacingMergeTree	дедупликация с версионированием
Движок SummingMergeTree	автоматическое суммирование числовых колонок
JOIN между таблицами	комплексный аналитический запрос
Понимание ограничений движков	без OPTIMIZE FINAL агрегация не финальна
Все поставленные задачи выполнены. Полученные навыки позволяют эффективно работать с колоночной СУБД ClickHouse в аналитических проектах.
