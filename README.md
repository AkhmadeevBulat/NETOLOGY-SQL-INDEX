# Домашнее задание к занятию «Индексы»

---
## Ахмадеев Булат Наилевич

---

## Задание 1

![alt text](<images/Снимок экрана 2024-08-19 в 12.54.25.png>)

---

## Задание 2

### Исходный запрос:

```
SELECT DISTINCT
    CONCAT(c.last_name, ' ', c.first_name),
    SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM
    payment p,
    rental r,
    customer c,
    inventory i,
    film f
WHERE
    DATE(p.payment_date) = '2005-07-30'
    AND p.payment_date = r.rental_date
    AND r.customer_id = c.customer_id
    AND i.inventory_id = r.inventory_id;
```

### Анализ:

1. Соединения и условия фильтрации:

* Запрос соединяет пять таблиц (```cross join```) без явного указания ```JOIN```, что затрудняет чтение и потенциально снижает производительность.
* Используется функция ```DATE(p.payment_date)```, которая требует приведения типов, что может негативно сказываться на производительности.

2. Использование ```DISTINCT```:

* ```DISTINCT``` может потребовать дополнительной сортировки и объединения, что увеличивает время выполнения запроса.

3. ```OVER (PARTITION BY)```:

* Использование оконной функции с ```PARTITION BY``` выполняет операцию суммирования по конкретным группам, что добавляет сложности.

### Оптимизация запроса:

1. Улучшение использования индексов:

* Добавление индекса по полю ```payment_date``` улучшит производительность запроса:

```
CREATE INDEX idx_payment_date ON payment(payment_date);
```

* Нужно избегать использования функции ```DATE()``` на колонке, которая уже может быть индексирована. Вместо этого нужно сравнивать дату с диапазоном, что позволит использовать индекс:

```
WHERE p.payment_date >= '2005-07-30 00:00:00' 
  AND p.payment_date < '2005-07-31 00:00:00'
```

2. Переписывание запроса с явным использованием ```JOIN```:

```
SELECT DISTINCT
    CONCAT(c.last_name, ' ', c.first_name) AS full_name,
    SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title) AS total_amount
FROM
    payment p
JOIN
    rental r ON p.payment_date = r.rental_date AND p.rental_id = r.rental_id
JOIN
    customer c ON r.customer_id = c.customer_id
JOIN
    inventory i ON r.inventory_id = i.inventory_id
JOIN
    film f ON i.film_id = f.film_id
WHERE
    p.payment_date >= '2005-07-30 00:00:00' 
    AND p.payment_date < '2005-07-31 00:00:00';
```

3. Создание дополнительных индексов:

* Добавление индексов на часто используемые поля, которые участвуют в условиях соединения:

```
CREATE INDEX idx_rental_date ON rental(rental_date);
CREATE INDEX idx_customer_id ON customer(customer_id);
CREATE INDEX idx_inventory_id ON inventory(inventory_id);
```

Эти шаги должны значительно улучшить производительность исходного запроса.

![alt text](<images/Снимок экрана 2024-08-19 в 13.03.22.png>)

---