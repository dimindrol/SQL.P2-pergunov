# Домашнее задание к занятию "SQL. Часть 2" - Пергунов Д.В

### Задание 1
Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию:

фамилия и имя сотрудника из этого магазина;
город нахождения магазина;
количество пользователей, закреплённых в этом магазине.

```sql
SELECT 
CONCAT(staff.last_name, ' ', staff.first_name) AS First_Last_Name_of_Staff, #ФИО персонала
city.city as City, #наименование города
COUNT(cus.customer_id) AS count_of_customers #Суммирование покупателей
FROM sakila.staff staff
INNER JOIN sakila.store store ON staff.store_id = store.store_id 
INNER JOIN sakila.customer cus ON store.store_id = cus.store_id
INNER JOIN sakila.address addr ON store.address_id = addr.address_id
INNER JOIN sakila.city city ON addr.city_id = city.city_id
GROUP BY staff.staff_id, city.city #Группировка по сотруднику и городу
HAVING COUNT(cus.customer_id) > 300;  #Отбираем группы, где количество покупателей больше 300
```

![image](https://github.com/dimindrol/SQL.P2-pergunov/assets/103885836/f0a57985-67d4-40c5-af36-b7035d864d4e)


### Задание 2
Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

```sql
SELECT COUNT(*)
FROM sakila.film f
WHERE length > (SELECT AVG(length) FROM sakila.film);#Условие выборки когда длина фильма больше средней длины фильма
```
![image](https://github.com/dimindrol/SQL.P2-pergunov/assets/103885836/e083868a-82be-4316-bbf5-50607725e559)


### Задание 3
Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

```sql
WITH MonthPay AS ( #вычисляет сумму платежей для каждого месяца
SELECT
DATE_FORMAT(p.payment_date, '%Y-%m') AS pay_month,
SUM(p.amount) AS total_payments
FROM sakila.payment p
GROUP BY DATE_FORMAT(p.payment_date, '%Y-%m')
),
MaxPaymentMonth AS (  #Определяем месяц с наибольшей суммой платежей, используя RANK() для ранжирования по сумме платежей
SELECT pay_month, total_payments,
RANK() OVER (ORDER BY total_payments DESC) AS pay_rank
FROM MonthPay
)
SELECT 
mp.pay_month,
mp.total_payments,
COUNT(r.rental_id) AS rental_count
FROM MaxPaymentMonth mp
INNER JOIN sakila.rental r ON DATE_FORMAT(r.rental_date, '%Y-%m') = mp.pay_month #присоединяем таблицу аренд для подсчета количества аренд (rental_count) в месяце с наибольшей суммой платежей
WHERE mp.pay_rank = 1
GROUP BY mp.pay_month, mp.total_payments
ORDER BY mp.pay_month;
```
![image](https://github.com/dimindrol/SQL.P2-pergunov/assets/103885836/b9c662fd-2ad4-4a5d-94ca-611c7ede9261)



