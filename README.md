# Retail Sales Analysis

Проект включает в себя создание базы данных, проведение разведочного анализа (EDA) и решение конкретных бизнес-задач с помощью SQL-запросов.

## Цели проекта
1. **Создание базы данных розничных продаж**
2. **Очистка данных**: найти и удалить записи, содержащие нулевые значения или пропуски
3. **Разведочный анализ данных (EDA)**: провести первичный анализ данных для понимания структуры
4. **Бизнес анализ**: с помощью SQL-запросов ответить на ключевые бизнес-вопросы

## Структура проекта

### 1. Настройка базы данных

- **Создание базы данных**: проект начинается с создания базы данных `retail`
- **Создание таблицы**: создаётся таблица `retail_sales` для хранения данных о продажах. . Структура таблицы включает следующие колонки: идентификатор транзакции, дата продажи, время продажи, идентификатор клиента, пол, возраст, категория товара, количество проданных единиц, цена за единицу, себестоимость проданных товаров (COGS) и общая сумма продажи.
```sql
CREATE DATABASE retail;

DROP TABLE IF EXISTS retail_sales;
CREATE TABLE retail_sales
			(
				transactions_id	INT PRIMARY KEY,
				sale_date DATE,
				sale_time TIME,
				customer_id	INT,
				gender VARCHAR(15),
				age	INT,
				category VARCHAR(15),	
				quantiy	INT,
				price_per_unit FLOAT,
				cogs FLOAT,
				total_sale FLOAT
			);
```

### 2. Исследование и очистка данных

- **Подсчёт количества записей**: определение общего числа записей в датасете.

- **Количество клиентов**: подсчёт уникальных клиентов в датасете.

- **Количество категорий**: выявление всех уникальных категорий товаров в датасете.

- **Проверка на пропуски**: поиск записей с нулевыми значениями и их удаление.
```sql
SELECT COUNT(*) FROM retail_sales;
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;
SELECT DISTINCT category FROM retail_sales;

SELECT * FROM retail_sales
WHERE 
    sale_date IS NULL
    OR
    sale_time IS NULL
    OR
    customer_id IS NULL
    OR 
    gender IS NULL
    OR
    age IS NULL
    OR category IS NULL
    OR 
    quantity IS NULL
    OR
    price_per_unit IS NULL
    OR
    cogs IS NULL;

DELETE FROM retail_sales
WHERE 
    sale_date IS NULL
    OR
    sale_time IS NULL
    OR
    customer_id IS NULL
    OR 
    gender IS NULL
    OR
    age IS NULL
    OR
    category IS NULL
    OR 
    quantity IS NULL
    OR
    price_per_unit IS NULL
    OR
    cogs IS NULL;
```

### 3. Анализ данных и выводы

Для ответа на ключевые бизнес-вопросы были разработаны следующие SQL-запросы:

1. **Напишите SQL-запрос для вывода всех столбцов по продажам, совершённым 5 ноября 2022 года**:
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

2. **Напишите SQL-запрос для вывода всех транзакций, у которых категория «Одежда» (Clothing), а количество проданных единиц превышает 4 за ноябрь 2022 года**:
```sql
SELECT 
  *
FROM retail_sales
WHERE 
    category = 'Clothing'
    AND 
    TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
    AND
    quantity >= 4
```

3. **Напишите SQL-запрос для расчёта общей суммы продаж (total_sale) в разрезе каждой категории**:
```sql
SELECT 
    category,
    SUM(total_sale) as net_sale,
    COUNT(*) as total_orders
FROM retail_sales
GROUP BY 1
```

4. **Напишите SQL-запрос для вычисления среднего возраста покупателей, совершивших покупки в категории «Косметика» (Beauty)**:
```sql
SELECT
    ROUND(AVG(age), 2) as avg_age
FROM retail_sales
WHERE category = 'Beauty'
```

5. **Напишите SQL-запрос для поиска всех транзакций, у которых общая сумма продажи (total_sale) превышает 1000**:
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000
```

6. **Напишите SQL-запрос для подсчёта количества транзакций (transaction_id) в разрезе пола и категории товара**:
```sql
SELECT 
    category,
    gender,
    COUNT(*) as total_trans
FROM retail_sales
GROUP 
    BY 
    category,
    gender
ORDER BY 1
```
7. **Напишите SQL-запрос для расчёта среднего чека за каждый месяц. Определите месяц с максимальными продажами в каждом году**:
```sql
SELECT 
       year,
       month,
    avg_sale
FROM 
(    
SELECT 
    EXTRACT(YEAR FROM sale_date) as year,
    EXTRACT(MONTH FROM sale_date) as month,
    AVG(total_sale) as avg_sale,
    RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) as rank
FROM retail_sales
GROUP BY 1, 2
) as t1
WHERE rank = 1
```
8. **Напишите SQL-запрос для выявления 5 покупателей по наибольшей общей сумме покупок**:
```sql
SELECT 
    customer_id,
    SUM(total_sale) as total_sales
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

9. **Напишите SQL-запрос для подсчёта количества уникальных покупателей в каждой категории товаров**:
```sql
SELECT 
    category,    
    COUNT(DISTINCT customer_id) as cnt_unique_cs
FROM retail_sales
GROUP BY category
```

10. **Напишите SQL-запрос для распределения заказов по временны́м слотам и подсчёта их количества (утро — до 12:00, день — с 12:00 до 17:00, вечер — после 17:00)**:
```sql
WITH hourly_sale
AS
(
SELECT *,
    CASE
        WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END as shift
FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) as total_orders    
FROM hourly_sale
GROUP BY shift
```

## Выводы:
- **Возраст покупателей**: датасет охватывает покупателей различных возрастных групп, продажи распределены по разным категориям, включая одежду и косметику.

- **Крупные покупки**: несколько транзакций имеют общую сумму продажи более 1000, что позволяет выделить наиболее обеспеченных покупателей.

- **Сезонные тренды**: помесячный анализ показывает колебания в продажах, что помогает определить пиковые периоды.

- **Поведение клиентов**: анализ позволяет выявить самых обеспеченных покупателей и наиболее востребованные категории товаров.
