# Домашнее задание к занятию "`Индексы`" - `Кудряшов Андрей`

---

### Задание 1

```
sudo mysql -u root
USE sakila;
```
```
SELECT 
  ROUND(
    SUM(INDEX_LENGTH) / SUM(DATA_LENGTH + INDEX_LENGTH) * 100,
    2
  ) AS index_to_total_percent
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'sakila';
```

<img width="641" height="335" alt="1" src="https://github.com/user-attachments/assets/40e03aa3-ec0e-4c62-9e12-d150c8bb4496" />


---

### Задание 2

EXPLAIN ANALYZE не сработало.

Оригинальный запрос:
```
SELECT DISTINCT 
  CONCAT(c.last_name, ' ', c.first_name), 
  SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title)
FROM payment p, rental r, customer c, inventory i, film f
WHERE 
  DATE(p.payment_date) = '2005-07-30' 
  AND p.payment_date = r.rental_date 
  AND r.customer_id = c.customer_id 
  AND i.inventory_id = r.inventory_id;
```

<img width="827" height="916" alt="2" src="https://github.com/user-attachments/assets/9131d50d-4a2a-4dac-944e-e0bf667ffc96" />

* Нет связи между inventory и film → film f просто висит в воздухе.
* Связь p.payment_date = r.rental_date — странная.
* DISTINCT + оконная функция — выглядит как костыль.


Оптимизированный запрос:
```
SELECT 
  CONCAT(c.last_name, ' ', c.first_name) AS customer,
  f.title AS film,
  SUM(p.amount) AS total
FROM payment p
JOIN rental r ON p.rental_id = r.rental_id
JOIN customer c ON r.customer_id = c.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
WHERE 
  p.payment_date >= '2005-07-30' 
  AND p.payment_date < '2005-07-31'
GROUP BY c.customer_id, f.film_id;
```

<img width="503" height="878" alt="3" src="https://github.com/user-attachments/assets/2c304e28-fe34-440a-80cf-db23056a36c8" />

---

### Задание 3

GIN - Специализированный индекс для работы со сложными, составными типами данных, где один столбец может содержать несколько значений.

GiST - "Обобщённое дерево поиска". Инфраструктура для реализации различных стратегий индексирования.

SP-GiST - Специализированная версия GiST, предназначенная для данных, которые можно рекурсивно разделить на непересекающиеся области.

BRIN - Работает с диапазонами блоков данных на диске.

---
