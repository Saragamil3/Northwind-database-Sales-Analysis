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

# show number of customers for each category
select Customer_Segments
,count( Customer_Segments) CustomersNumber
from Customer_categories 
group by Customer_Segments
```

- Order Value:
 - High-Value, Medium-Value, Low-Value customers based on their avarage order revenue value
```sql
create view avarage_order_revenue as
select o.customerid
,count(distinct od.OrderID) TotalOrders
,cast((sum(unitprice*quantity*(1-discount))) as int) TotalAmountSpent
,cast((sum(unitprice*quantity*(1-discount))) as int) /count( DISTINCT od.OrderID) avarage_order_revenue
from orders o inner join  'Order Details' od 
on o.OrderID=od.orderid
group by 1
order by  avarage_order_revenue
```
-- customers segmentation 
```sql
create view CustomerCategories as 
select customerid
,avarage_order_revenue 
, case 
  when avarage_order_revenue < 26000 then 'Low-Value'
  when avarage_order_revenue > 29000 then 'High-Value'
  else ' Medium-Value'
  end CustomerCategory
from avarage_order_revenue
order by 3 DESC

# Number of customers per category 
select CustomerCategory
, count(CustomerCategory) CustomersNumber
from CustomerCategories
group by 1
```


 
