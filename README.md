# Northwind Public Database 
# 1- Customer Segmentation
- RFM Analysis:
  - Recency: Days since last order
  - Frequency: Total number of orders
  - Monetary Value: Total amount spent

```sql
 create VIEW RFM_Analysis as 
select o.customerid
, max(date(o.OrderDate)) LastOrder
, julianday(date('now'))- julianday(max(date(o.OrderDate))) Days_Since_lastOrders
, count( DISTINCT o.OrderID) TotalOrders
, cast((sum(unitprice*quantity*(1-discount))) as int) TotalAmountSpent
from  orders o 
left join 'Order Details' od 
on o.OrderID=od.orderid
group by 1
order by 1
```

- Customer categories
```sql
create view Customer_categories as 
select customerid
, case 
  when Days_Since_lastOrders < 450 and TotalOrders> 190 and TotalAmountSpent > 5000000 
  then 'Champions'
  when TotalOrders BETWEEN 160 and 190 or TotalAmountSpent BETWEEN 4500000 and 5000000 
  then 'Potential Loyalists'
  else 'At Risk' 
  end Customer_Segments
from RFM_Analysis
order by 2
```

-show number of customers for each category
```sql
select Customer_Segments
,count( Customer_Segments) CustomersNumber
from Customer_categories 
group by Customer_Segments
```


 
