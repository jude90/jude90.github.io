---
title: "Neo4j 笔记4 Data Import & etl" 
layout: post
date: 2016-01-14 03:06:26
---


这次的笔记介绍一下 关系型数据库的模型 到 图数据库模型 的转换过程, 内容整理自neo4j 的官网。[Import Data Into Neo4j](http://neo4j.com/developer/guide-importing-data-and-etl/)

将关系型数据转换成图数据，可以遵循的原则是：

- 一条记录( record ) 是 一个节点( node )
- 一个表( table ) 是一个 标签名 ( label )

我们使用的数据集是[ NorthWind dataset](http://code.google.com/p/northwindextended/downloads/detail?name=northwind.postgre.sql&can=2&q=)， 其中包含了丰富的表连接。非常适合转化成图数据库。

假设这些表已经存在我们的PostgresSQL 里面，数据库名为northwind，执行`psql -d northwind < export_csv.sql` 把它们 导出为csv 文件。
**export_csv.sql** :

```sql
COPY (SELECT * FROM customers) TO '/tmp/customers.csv' WITH CSV header;

COPY (SELECT * FROM suppliers) TO '/tmp/suppliers.csv' WITH CSV header;

COPY (SELECT * FROM products)  TO '/tmp/products.csv' WITH CSV header;

COPY (SELECT * FROM employees) TO '/tmp/employees.csv' WITH CSV header;

COPY (SELECT * FROM categories) TO '/tmp/categories.csv' WITH CSV header;

COPY (SELECT * FROM orders

      LEFT OUTER JOIN order_details ON order_details.OrderID = orders.OrderID) TO '/tmp/orders.csv' WITH CSV header;
```


下面就需要一个Cypher 脚本，执行 [LOAD CSV](http://neo4j.com/docs/stable/query-load-csv.html) 命令，将csv 文件导入 neo4j， 建立节点和关系。

**import_csv.cypher**
首先创建节点：

```
// Create customers

USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:customers.csv" AS row

CREATE (:Customer {companyName: row.CompanyName, customerID: row.CustomerID, fax: row.Fax, phone: row.Phone});

// Create products

USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:products.csv" AS row

CREATE (:Product {productName: row.ProductName, productID: row.ProductID, unitPrice: toFloat(row.UnitPrice)});


// Create suppliers

USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:suppliers.csv" AS row

CREATE (:Supplier {companyName: row.CompanyName, supplierID: row.SupplierID});

// Create employees

USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:employees.csv" AS row

CREATE (:Employee {employeeID:row.EmployeeID,  firstName: row.FirstName, lastName: row.LastName, title: row.Title});


// Create categories

USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:categories.csv" AS row

CREATE (:Category {categoryID: row.CategoryID, categoryName: row.CategoryName, description: row.Description});

USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:orders.csv" AS row

MERGE (order:Order {orderID: row.OrderID}) ON CREATE SET order.shipName =  row.ShipName;
```

然后创建索引，这可以在创建 关系(relationship ) 时提高节点(node )查询速度。

```
CREATE INDEX ON :Product(productID);

CREATE INDEX ON :Product(productName);

CREATE INDEX ON :Category(categoryID);

CREATE INDEX ON :Employee(employeeID);

CREATE INDEX ON :Supplier(supplierID);

CREATE INDEX ON :Customer(customerID);

CREATE INDEX ON :Customer(customerName);

```

在创建了节点和 索引之后，就可以建立关系 ( relationship )。
订单 与产品及 雇员间的关系

```
USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:orders.csv" AS row

MATCH (order:Order {orderID: row.OrderID})

MATCH (product:Product {productID: row.ProductID})

MERGE (order)-[pu:PRODUCT]->(product)

ON CREATE SET pu.unitPrice = toFloat(row.UnitPrice), pu.quantity = toFloat(row.Quantity);


USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:orders.csv" AS row

MATCH (order:Order {orderID: row.OrderID})

MATCH (employee:Employee {employeeID: row.EmployeeID})

MERGE (employee)-[:SOLD]->(order);


USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:orders.csv" AS row

MATCH (order:Order {orderID: row.OrderID})

MATCH (customer:Customer {customerID: row.CustomerID})

MERGE (customer)-[:PURCHASED]->(order);
```

建立产品、供应商、类别 之间的关系。

```
USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:products.csv" AS row

MATCH (product:Product {productID: row.ProductID})

MATCH (supplier:Supplier {supplierID: row.SupplierID})

MERGE (supplier)-[:SUPPLIES]->(product);


USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:products.csv" AS row

MATCH (product:Product {productID: row.ProductID})

MATCH (category:Category {categoryID: row.CategoryID})

MERGE (product)-[:PART_OF]->(category);

```

最后创建 雇员之间 的层级关系

```
USING PERIODIC COMMIT

LOAD CSV WITH HEADERS FROM "file:employees.csv" AS row

MATCH (employee:Employee {employeeID: row.EmployeeID})

MATCH (manager:Employee {employeeID: row.ReportsTo})

MERGE (employee)-[:REPORTS_TO]->(manager);
```

为了数据的完整性和查询优化，最后在 Order 标签类上创建一个 `unique constraint`:
CREATE CONSTRAINT ON (o:Order) ASSERT o.orderID IS UNIQUE;

执行以下命令来 运行Cypher脚本：`neo4j-shell -path northwind.db -file import_csv.cypher.`

关系型数据库模型 里面的 外键 在 图数据中变成了 一个 关系。

关系型数据库的每一条记录 既可以包含 各种类型的属性 ，也会包含 数据模型之间一对一 、 一对多、 多对多 的关系。同一个关系型数据库表 ，可以拆分成 图数据库中的节点(node )和关系(relationship )。甚至可以把 记录的一部分属性 赋值给 关系( relationship ).
建立图数据库的步骤是先节点再 关系。

创建 节点的 CREATE 从句紧跟 LOAD CSV 从句。 每一行数据被赋值给 row 变量。row的一部分属性成为了节点的属性。 

创建节点之后，才可以 创建 关系。建立关系 其实就是遍历 那些包含了 一对一、 一对多、 多对多关系的 表，利用每一条记录 的外键 组合，用MATCH命令 匹配 到 相对应的两个节点，然后用MERGE 命令创建两节点之间的关系。