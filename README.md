# Northwind Public Database (SQLITE)
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

# Customer categories
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
![number of customers for each category](https://github.com/Saragamil3/Northwind-database-Sales-Analysis/blob/main/Screenshot%202025-05-03%20174518.png)

- Order Value:
   High-Value, Medium-Value, Low-Value customers based on their avarage order revenue value
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

# customers segmentation 
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
![customers segmentation based on their avarage order revenue value](https://github.com/Saragamil3/Northwind-database-Sales-Analysis/blob/main/Picture5.png)

# 2- Product Analysis:
- The top 10 revenue generator products
  
```sql
#High Revenue Value: Identify the top 10 revenue generator products.
create view products_with_high_revenue as 
select p.productname
,cast(sum(od.unitprice*od.quantity*(1-od.discount)) as int) as TotalRevenue
from Products p inner join 'Order Details' od
on p.ProductID= od.productid 
group by 1
order by TotalRevenue DESC
limit 10
```
![The top 10 revenue generator products](https://github.com/Saragamil3/Northwind-database-Sales-Analysis/blob/main/Picture2.png)

- The top 10 most frequently ordered products
  
```sql
--High Sales Volume: Determine the top 10 most frequently ordered products
create view products_with_high_sales
as
select p.productname 
,count( distinct od.orderid) TotalOrders
from Products p inner join 'Order Details' od
on p.ProductID= od.productid 
group by 1
order by  TotalOrders DESC
limit 10
```
![Determine the top 10 most frequently ordered products](https://github.com/Saragamil3/Northwind-database-Sales-Analysis/blob/main/Picture1.png)

- Slow Movers
```sql
--Slow Movers: Identify products with low sales volume. (5 product)
create view products_with_low_sales_volume as
select p.productname
,count(od.orderid) TotalOrders
from Products p inner join 'Order Details' od
on p.ProductID= od.productid 
group by 1
order by  TotalOrders ASC
limit 5
```
![products with low sales volume](https://github.com/Saragamil3/Northwind-database-Sales-Analysis/blob/main/Picture3.png)

 # 3-Order Analysis:
- Seasonality
```sql
--Seasonality: Identify any seasonal fluctuations in order volume
select strftime('%Y',orderdate) Years
, count(DISTINCT orderid) NumberOrders
from orders 
group by 1 
```
![Seasonality](https://github.com/Saragamil3/Northwind-database-Sales-Analysis/blob/main/Picture4.png)

- The most popular order days
 ```sql
 -- 2-Day-of-the-Week Analysis: Determine the most popular order days
CREATE view popular_order_days as 
select 
 case strftime('%w',orderdate) 
 when '0' then 'Saturday'
 when '1' then 'Sunday'
 when '2' then 'Monday'
 WHEN '3' THEN 'Tuesday'
 WHEN '4' THEN 'Wednesday'
 WHEN '5' THEN 'Thursday'
 WHEN '6' THEN 'Friday'
 end Day_of_the_week
, count(DISTINCT orderid) NumberOrders
from orders 
group by  Day_of_the_week
order by NumberOrders DESC
limit 2
 ```
![The most popular order days](https://github.com/Saragamil3/Northwind-database-Sales-Analysis/blob/main/Picture5.png)

- Distribution of order quantities
```sql
--Order Size Analysis: Analyze the distribution of order quantities
select DISTINCT orderid
,sum(quantity) Units_Sold
from 'Order Details'
group by 1
order by Units_Sold DESC
```
# 4-Employee Performance
- Total Revenue Generated
- Number of orders processed
- Average order value 
```sql
create view Employee_Performance as 
select concat(e.FirstName,' ', e.LastName) FullName
,cast(sum(od.unitprice*od.quantity*(1-od.discount)) as int) TotalRevenue
,count(DISTINCT o.OrderID) TotalOrders
, cast(sum(od.unitprice*od.quantity*(1-od.discount)) / count(DISTINCT o.OrderID) as int) Average_order_value 
from Employees e inner join orders o 
on e.EmployeeID=o.EmployeeID inner join 'Order Details' od 
on o.OrderID=od.orderid 
group by FullName
```
![Employee Performance](https://github.com/Saragamil3/Northwind-database-Sales-Analysis/blob/main/Picture6.png)
