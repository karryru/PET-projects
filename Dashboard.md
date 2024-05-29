## Визуализация данных оформлена в Redash 
### Дашборд прикреплен в виде .pdf
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


5. #### Average check in all countries for the last 3 months - средний чек в каждой стране за последние три месяца в сравнении с общим средним чеком за 3 месяца
6. #### Total revenue for the 3 last month - общая выручка за последние три месяца во всех странах за исключением UK
7. #### ANTI-TOP 3 countries (correlation canceled orders to success orders) - 3 страны с самым большим процентом отмененных заказов в сравнении с общим  количеством заказов
8. #### Average check in the UK for the entire period - средний чек в UK за весь период (детализация крупнейшего продавца) 




