
# Лабораторная работа №1  SQL
## Основы работы с SQL-запросами: выборка и модификация данных

**Номер варианта: 1** 


## Задание 1.1  
Вывести товары типа `'scooter'` дороже $500.  
Сортировка: цена по убыванию.

**Скриншот выполнения SELECT (из основной БД)**  
<img width="1301" height="141" alt="image" src="https://github.com/user-attachments/assets/5d03ad01-b33e-4853-91d5-05cc926e730b" />

```sql
SELECT *
FROM products
WHERE product_type = 'scooter' 
  AND base_msrp > 500
ORDER BY base_msrp DESC;
```

---

## Задание 2  
Вывести продажи через канал `'internet'` на сумму от $15 000 до $30 000.

**Скриншот выполнения SELECT (из основной БД)**  
<img width="1126" height="667" alt="image" src="https://github.com/user-attachments/assets/a249daff-3468-43e0-8104-2061f05dd9d6" />

```sql
SELECT *
FROM public.sales
WHERE channel = 'internet' 
  AND sales_amount BETWEEN 15000 AND 30000;
```

---

## Задание 3  
Создать таблицу `high_price` (товары дороже $1000).  
Изменить: увеличить цену на 10%.  
Удалить: товары выпущенные до 2015 года.

**Изменения:**  
В исходных данных отсутствуют записи с годом выпуска до 2012, поэтому условие удаления было скорректировано на `year < 2015`, чтобы продемонстрировать работу операции `DELETE`.

**Скриншот подтверждения CRUD (итоговая выборка из локальной таблицы `high_price` после всех изменений)**  
<img width="1081" height="212" alt="image" src="https://github.com/user-attachments/assets/38063612-ae9d-45a4-a636-1ed4c1d5416b" />


```sql
-- Шаг 1: Создание таблицы
DROP TABLE IF EXISTS high_price;
CREATE TABLE high_price AS
SELECT *
FROM products
WHERE base_msrp > 1000;
```
<img width="406" height="73" alt="image" src="https://github.com/user-attachments/assets/6317a5b9-276c-4380-bd71-6a028e4cf804" />
<img width="1062" height="212" alt="image" src="https://github.com/user-attachments/assets/ff06e833-a40d-4414-8a29-256c26216797" />

```
-- Шаг 2: Обновление цены (+10%)
UPDATE high_price
SET base_msrp = base_msrp * 1.10;
```
<img width="1056" height="219" alt="image" src="https://github.com/user-attachments/assets/eeec0cae-090e-41bf-87ff-0c15bac27662" />

```
-- Шаг 3: Удаление товаров выпущенных до 2015 года
DELETE FROM high_price
WHERE year < 2015;
-- Итоговая выборка
SELECT * FROM high_price;
```
<img width="1069" height="179" alt="image" src="https://github.com/user-attachments/assets/5f675e5d-8490-4fc1-b2cd-a4742a5ef6e9" />


# Вывод

В ходе выполнения лабораторной работы мною были выполнены SELECT-запросы с фильтрацией по типу товара, каналу продаж и диапазону суммы, реализованы операции CRUD, включающие создание таблицы на основе выборки, массовое обновление данных и удаление записей по условию, а также скорректированы условия заданий с учётом фактического наполнения базы данных для корректного выполнения всех операций.
