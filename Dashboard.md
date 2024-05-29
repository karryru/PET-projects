## Визуализация данных оформлена в Redash 
### Дашборд прикреплен в виде .pdf
1. #### Revenue - выручка по странам (без UK) за последние три месяца продаж для отображения актуального состояния бизнеса
2. #### Percentage of canceled orders - 
![image](https://github.com/karryru/retails/assets/170441289/348269ea-571f-4c35-8c8b-846c93f66da2)


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
