
# SQL Subqueries - Lab

## Introduction

Now that you've seen how subqueries work, it's time to get some practice writing them! Not all of the queries will require subqueries, but all will be a bit more complex and require some thought and review about aggregates, grouping, ordering, filtering, joins and subqueries. Good luck!  

## Objectives

You will be able to:

* Write subqueries to decompose complex queries

## CRM Database Schema

Once again, here's the schema for the CRM database you'll continue to practice with.

<img src="images/Database-Schema.png" width="600">

## Connect to the Database

As usual, start by importing the necessary packages and connecting to the database **data.sqlite**.


```python
# Your code here; import the necessary packages
```


```python
# Your code here; create the connection and cursor
```


```python
# __SOLUTION__ 
import sqlite3
import pandas as pd
```


```python
# __SOLUTION__ 
conn = sqlite3.Connection('data.sqlite')
cur = conn.cursor()
```

## Write an Equivalent Query using a Subquery

```SQL
SELECT customerNumber,
       contactLastName,
       contactFirstName
       FROM customers
       JOIN orders 
       USING(customerNumber)
       WHERE orderDate = '2003-01-31';
```


```python
# Your code here; use a subquery. No join is necessary 
```


```python
# __SOLUTION__ 
cur.execute("""SELECT customerNumber, contactLastName, contactFirstName
               FROM customers
               WHERE customerNumber IN (SELECT customerNumber 
                                        FROM orders 
                                        WHERE orderDate = '2003-01-31');
                                        """)
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>customerNumber</th>
      <th>contactLastName</th>
      <th>contactFirstName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>141</td>
      <td>Freyre</td>
      <td>Diego</td>
    </tr>
  </tbody>
</table>
</div>



## Select the Total Number of Orders for Each Product Name

Sort the results by the total number of items sold for that product.


```python
# Your code here
```


```python
# __SOLUTION__ 
cur.execute("""SELECT productName, COUNT(orderNumber) as numberOrders, SUM(quantityOrdered) as totalUnitsSold
               FROM products
               JOIN orderdetails
               USING (productCode)
               GROUP BY productName
               ORDER BY totalUnitsSold desc;
               """)
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
print(df.shape)
df.tail()
```

    (109, 3)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>productName</th>
      <th>numberOrders</th>
      <th>totalUnitsSold</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>104</th>
      <td>1999 Indy 500 Monte Carlo SS</td>
      <td>25</td>
      <td>855</td>
    </tr>
    <tr>
      <th>105</th>
      <td>1911 Ford Town Car</td>
      <td>25</td>
      <td>832</td>
    </tr>
    <tr>
      <th>106</th>
      <td>1936 Mercedes Benz 500k Roadster</td>
      <td>25</td>
      <td>824</td>
    </tr>
    <tr>
      <th>107</th>
      <td>1970 Chevy Chevelle SS 454</td>
      <td>25</td>
      <td>803</td>
    </tr>
    <tr>
      <th>108</th>
      <td>1957 Ford Thunderbird</td>
      <td>24</td>
      <td>767</td>
    </tr>
  </tbody>
</table>
</div>



## Select the Product Name and the  Total Number of People Who Have Ordered Each Product

Sort the results in descending order.

### A quick note on the SQL  `SELECT DISTINCT` statement:

The `SELECT DISTINCT` statement is used to return only distinct values in the specified column. In other words, it removes the duplicate values in the column from the result set.

Inside a table, a column often contains many duplicate values; and sometimes you only want to list the unique values. If you apply the `DISTINCT` clause to a column that has `NULL`, the `DISTINCT` clause will keep only one NULL and eliminates the other. In other words, the DISTINCT clause treats all `NULL` “values” as the same value.


```python
# Your code here:
# Hint: because one of the tables we'll be joining has duplicate customer numbers, you should use DISTINCT
```


```python
# __SOLUTION__ 
cur.execute("""SELECT productName, COUNT(DISTINCT customerNumber) AS numPurchasers
               FROM products
               JOIN orderdetails
               USING(productCode)
               JOIN orders
               USING(orderNumber)
               GROUP BY productName
               ORDER BY numPurchasers DESC;
               """)
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>productName</th>
      <th>numPurchasers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1992 Ferrari 360 Spider red</td>
      <td>40</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1934 Ford V8 Coupe</td>
      <td>27</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1952 Alpine Renault 1300</td>
      <td>27</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1972 Alfa Romeo GTA</td>
      <td>27</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Boeing X-32A JSF</td>
      <td>27</td>
    </tr>
  </tbody>
</table>
</div>



## Select the Employee Number, First Name, Last Name, City (of the office), and Office Code of the Employees Who Sold Products Which Have Been Ordered by Less Then 20 people.

This problem is a bit tougher. To start, think about how you might break the problem up. Be sure that your results only list each employee once.


```python
# Your code here
```


```python
# __SOLUTION__ 
cur.execute("""SELECT DISTINCT employeeNumber, officeCode, o.city, firstName, lastName
               FROM employees e
               JOIN offices o
               USING(officeCode)
               JOIN customers c
               ON e.employeeNumber = c.salesRepEmployeeNumber
               JOIN orders
               USING(customerNumber)
               JOIN orderdetails
               USING(orderNumber)
               WHERE productCode IN (SELECT productCode
                                            FROM products
                                            JOIN orderdetails
                                            USING(productCode)
                                            JOIN orders
                                            USING(orderNumber)
                                            GROUP BY productCode
                                            HAVING COUNT(DISTINCT customerNumber) < 20);
                                            """)
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>employeeNumber</th>
      <th>officeCode</th>
      <th>city</th>
      <th>firstName</th>
      <th>lastName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1370</td>
      <td>4</td>
      <td>Paris</td>
      <td>Gerard</td>
      <td>Hernandez</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1501</td>
      <td>7</td>
      <td>London</td>
      <td>Larry</td>
      <td>Bott</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1337</td>
      <td>4</td>
      <td>Paris</td>
      <td>Loui</td>
      <td>Bondur</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1166</td>
      <td>1</td>
      <td>San Francisco</td>
      <td>Leslie</td>
      <td>Thompson</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1286</td>
      <td>3</td>
      <td>NYC</td>
      <td>Foon Yue</td>
      <td>Tseng</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1612</td>
      <td>6</td>
      <td>Sydney</td>
      <td>Peter</td>
      <td>Marsh</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1611</td>
      <td>6</td>
      <td>Sydney</td>
      <td>Andy</td>
      <td>Fixter</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1401</td>
      <td>4</td>
      <td>Paris</td>
      <td>Pamela</td>
      <td>Castillo</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1621</td>
      <td>5</td>
      <td>Tokyo</td>
      <td>Mami</td>
      <td>Nishi</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1323</td>
      <td>3</td>
      <td>NYC</td>
      <td>George</td>
      <td>Vanauf</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1165</td>
      <td>1</td>
      <td>San Francisco</td>
      <td>Leslie</td>
      <td>Jennings</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1702</td>
      <td>4</td>
      <td>Paris</td>
      <td>Martin</td>
      <td>Gerard</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1216</td>
      <td>2</td>
      <td>Boston</td>
      <td>Steve</td>
      <td>Patterson</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1188</td>
      <td>2</td>
      <td>Boston</td>
      <td>Julie</td>
      <td>Firrelli</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1504</td>
      <td>7</td>
      <td>London</td>
      <td>Barry</td>
      <td>Jones</td>
    </tr>
  </tbody>
</table>
</div>



## Select the Employee Number, First Name, Last Name, and Number of Customers for Employees Whose Customers Have an Average Credit Limit of Over 15K


```python
# Your code here
```


```python
# __SOLUTION__ 
cur.execute("""SELECT employeeNumber, firstName, lastName, COUNT(customerNumber) AS numCustomers
               FROM employees e
               JOIN customers c
               ON e.employeeNumber = c.salesRepEmployeeNumber
               GROUP BY employeeNumber, firstName, lastName
               HAVING AVG(creditLimit) > 15000;
               """)
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>employeeNumber</th>
      <th>firstName</th>
      <th>lastName</th>
      <th>numCustomers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1165</td>
      <td>Leslie</td>
      <td>Jennings</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1166</td>
      <td>Leslie</td>
      <td>Thompson</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1188</td>
      <td>Julie</td>
      <td>Firrelli</td>
      <td>6</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1216</td>
      <td>Steve</td>
      <td>Patterson</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1286</td>
      <td>Foon Yue</td>
      <td>Tseng</td>
      <td>7</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1323</td>
      <td>George</td>
      <td>Vanauf</td>
      <td>8</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1337</td>
      <td>Loui</td>
      <td>Bondur</td>
      <td>6</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1370</td>
      <td>Gerard</td>
      <td>Hernandez</td>
      <td>7</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1401</td>
      <td>Pamela</td>
      <td>Castillo</td>
      <td>10</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1501</td>
      <td>Larry</td>
      <td>Bott</td>
      <td>8</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1504</td>
      <td>Barry</td>
      <td>Jones</td>
      <td>9</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1611</td>
      <td>Andy</td>
      <td>Fixter</td>
      <td>5</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1612</td>
      <td>Peter</td>
      <td>Marsh</td>
      <td>5</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1621</td>
      <td>Mami</td>
      <td>Nishi</td>
      <td>5</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1702</td>
      <td>Martin</td>
      <td>Gerard</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>



## Summary

In this lesson, you got to practice some more complex SQL queries, some of which required subqueries. There's still plenty more SQL to be had though; hope you've been enjoying some of these puzzles!
