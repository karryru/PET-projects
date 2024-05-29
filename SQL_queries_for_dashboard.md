## Визуализация данных оформлена в Redash 
### Дашборд прикреплен в виде [pdf-файла](https://github.com/karryru/retails/blob/main/Dashboard.pdf)
#### 1. Revenue - выручка по странам (без UK) за последние три месяца продаж для отображения актуального состояния бизнеса
   ```sql
   SELECT 
       Country,
       toStartOfMonth(InvoiceDate) AS InvoiceDate,
       SUM(Quantity*UnitPrice) AS total_revenue
   FROM default.retail
   WHERE (InvoiceDate = '2011-09-01' OR InvoiceDate = '2011-10-01' OR InvoiceDate = '2011-11-01') 
         AND Country != 'United Kingdom' 
         AND Quantity > 0
   GROUP BY InvoiceDate, Country
   ORDER BY Country ASC, InvoiceDate ASC
   ``` 

За последние три месяца продаж лидерство по объему выручки сохраняют Netherlands, EIRE, Germany.
      
#### 2. Percentage of canceled orders - процент отмененных заказов во всех странах за всё время продаж

 ```sql
   SELECT 
       l.Country as Country,
       l.total_counts as total_counts,
       r.cancelled_orders as cancelled_orders,
       (cancelled_orders/total_counts)*100 as CR_cancelled
   FROM (
       SELECT
           Country,
           COUNT(InvoiceNo) AS total_counts
       FROM default.retail
       WHERE (toStartOfMonth(InvoiceDate) = '2011-09-01' OR toStartOfMonth(InvoiceDate) = '2011-10-01' OR toStartOfMonth(InvoiceDate) = '2011-11-01')
           AND Country != 'United Kingdom'
       GROUP BY Country) AS l
   LEFT JOIN (
               SELECT 
                   Country,
                   COUNT(InvoiceNo) AS cancelled_orders
      FROM default.retail
       WHERE (toStartOfMonth(InvoiceDate) = '2011-09-01' OR toStartOfMonth(InvoiceDate) = '2011-10-01' OR toStartOfMonth(InvoiceDate) = '2011-11-01')
           AND Country != 'United Kingdom'
           AND Quantity < 0
       GROUP BY Country
               ) AS r
   ON l.Country=r.Country
   WHERE cancelled_orders > 0
   ORDER BY Country ASC
   ```
Наибольший CR отмененных заказов у USA, Malta, Czech Republic

#### 3. Average check in all countries for the last 3 months - средний чек в каждой стране за последние три месяца в сравнении с общим средним чеком за 3 месяца
```sql
WITH    (SELECT 
            AVG(check_for_date) AS total_avg_check
        FROM (
        SELECT 
            Country,
            AVG(check) AS check_for_date
        FROM (   
                SELECT 
                    Country,
                    InvoiceNo,
                    SUM(Quantity*UnitPrice) as check
                FROM default.retail
                WHERE Quantity > 0
                group by InvoiceNo, Country )
        GROUP BY Country
        ORDER BY Country ASC)) AS total_countries_avg_check

SELECT 
    Country,
    ROUND(AVG(total_price),0) as AOU,
    total_countries_avg_check
FROM (
        SELECT 
            Country,
            InvoiceNo,
            SUM(Quantity*UnitPrice) as total_price
        FROM default.retail
        WHERE Quantity > 0 AND (toStartOfMonth(InvoiceDate) = '2011-09-01' OR toStartOfMonth(InvoiceDate) = '2011-10-01' OR toStartOfMonth(InvoiceDate) = '2011-11-01') 
        GROUP BY InvoiceNo, Country )
GROUP BY Country
ORDER BY Country ASC
```
Большая часть стран имеют средний чек ниже общего среднего чека.

#### 4. Total revenue for the 3 last month - общая выручка за последние три месяца во всех странах за исключением UK
```sql
SELECT
    SUM(total) as total_revenue
FROM (
    SELECT 
        (Quantity*UnitPrice) as total
    FROM default.retail
    WHERE Quantity > 0 
        AND (toStartOfMonth(InvoiceDate) = '2011-09-01' OR toStartOfMonth(InvoiceDate) = '2011-10-01' OR toStartOfMonth(InvoiceDate) = '2011-11-01')
        AND Country != 'United Kingdom'
    )
```
#### 5. ANTI-TOP 3 countries (correlation canceled orders to success orders) - 3 страны с самым большим процентом отмененных заказов в сравнении с общим  количеством заказов
```sql
SELECT 
    l.Country,
    SUM(l.order_counts) as success_orders,
    SUM(r.cancelled_counts) as canceled_orders
FROM (
        SELECT
            Country,
            COUNT(InvoiceNo) AS order_counts
        FROM default.retail
        WHERE Quantity > 0 AND (toStartOfMonth(InvoiceDate) = '2011-09-01' OR toStartOfMonth(InvoiceDate) = '2011-10-01' OR toStartOfMonth(InvoiceDate) = '2011-11-01')
        GROUP BY Country --InvoiceDate
        ) as l
FULL JOIN  (
            SELECT
                Country,
                COUNT(InvoiceNo) AS cancelled_counts
            FROM default.retail
            WHERE Quantity < 0 AND (toStartOfMonth(InvoiceDate) = '2011-09-01' OR toStartOfMonth(InvoiceDate) = '2011-10-01' OR toStartOfMonth(InvoiceDate) = '2011-11-01')
            GROUP BY Country --, InvoiceDate
            ) AS r
ON l.Country = r.Country
where Country = 'Czech Republic' OR Country = 'Malta' OR Country = 'USA'
GROUP BY Country 
ORDER BY canceled_orders DESC
```

8. #### Average check in the UK for the entire period - средний чек в UK за весь период (детализация крупнейшего продавца) 
Запрос в Redash имеет фильтр по всем странам, здесь отображен полный запрос. Однако в .pdf файле визаулизация только для UK
```sql
WITH    (SELECT 
            AVG(check_for_date) AS total_avg_check
        FROM (
        SELECT 
            InvoiceDate,
            AVG(check) AS check_for_date
        FROM (   
                SELECT 
                    toStartOfMonth(InvoiceDate) AS InvoiceDate,
                    InvoiceNo,
                    SUM(Quantity*UnitPrice) as check
                FROM default.retail
                WHERE Quantity > 0
                group by InvoiceNo, InvoiceDate )
        GROUP BY InvoiceDate
        ORDER BY InvoiceDate ASC)) AS total_avg

SELECT 
    Country as "Country::filter",
    InvoiceDate,
    AVG(total_price) as AOU,
    total_avg
FROM (
        SELECT 
            Country,
            toStartOfMonth(InvoiceDate) as InvoiceDate,
            InvoiceNo,
            SUM(Quantity*UnitPrice) as total_price
        FROM default.retail
        WHERE Quantity > 0 AND InvoiceDate < '2011-12-01'
        GROUP BY InvoiceNo, InvoiceDate, Country )
GROUP BY InvoiceDate, Country
ORDER BY InvoiceDate ASC
```



