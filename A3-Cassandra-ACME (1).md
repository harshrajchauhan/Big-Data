# TEAM ACME

# Project Introduction:

Our project aims to design a relational database management system (RDBMS) schema tailored to the e-commerce domain. This schema will encompass various entities, relationships, and attributes relevant to e-commerce operations, including customers, orders, products, and transactions. Additionally, we have explored the significance of translating RDBMS schemas into Cassandra schemas for big data applications.


## Objectives:

- Develop a comprehensive RDBMS schema for managing e-commerce data.
- Analyze essential queries required for efficient application functionality.
- Translate the RDBMS schema into a Cassandra schema, considering Cassandra's data model and design principles.
- Discuss the challenges and considerations encountered during the translation process.
- Reflect on the trade-offs of using Cassandra over traditional RDBMS for our e-commerce application.


## Significance:

- To fully utilize big data technologies in e-commerce applications, RDBMS schemas must be translated into Cassandra schemas. 
- Massive volumes of e-commerce data, including customer records, order details, and product information, are ideally suited for Cassandra's distributed architecture, scalability, and fault tolerance. 
- We can use Cassandra's capabilities to create reliable, scalable, and high-performing e-commerce apps if we comprehend the nuances of this translation process.


# Analyze the RDBMS Schema

Our RDBMS schema for the e-commerce domain comprises several interconnected tables representing entities such as customers, orders, products, and transactions. Below are the key components of our RDBMS schema:


## Tables:

1. **Customers Table**: The table contains information about customers who make purchases on the e-commerce platform.

| Column Name   | Data Type | Description                              |
|---------------|-----------|------------------------------------------|
| customer_id     | INT       | Primary key; Unique identifier for the customer. |
| name         | VARCHAR(100) | Name of the customer.                   |
| email   | VARCHAR(100)      | Email address of the customer.              |
| address | VARCHAR(255) | Address of the customer. |

```sql
   CREATE TABLE Customers (
       customer_id INT PRIMARY KEY,
       name VARCHAR(100),
       email VARCHAR(100),
       address VARCHAR(255)
   );
```
<br>

2. **Orders Table**: The table stores details about orders placed by customers.

| Column Name | Data Type   | Description                                                   |
|-------------|-------------|---------------------------------------------------------------|
| order_id    | INT         | Primary key; Unique identifier for the order.                 |
| customer_id | INT         | Foreign key referencing the customer who placed the order.    |
| order_date  | DATETIME    | Date and time when the order was placed.                      |
| total_amount| DECIMAL(10,2)| Total amount of the order.                                   |

```sql
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATETIME,
    total_amount DECIMAL(10,2)
);
```
<br>

3. **Products Table**: The table stores information about products available for purchase.

| Column Name | Data Type   | Description                                    |
|-------------|-------------|------------------------------------------------|
| product_id  | INT         | Primary key; Unique identifier for the product.|
| name        | VARCHAR(100)| Name of the product.                           |
| price       | DECIMAL(10,2)| Price of the product.                          |
| description | TEXT        | Description of the product.                    |

```sql
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2),
    description TEXT
);
```
<br>

4. **Transactions Table**: The table represents individual transactions for each order.

| Column Name     | Data Type   | Description                                                   |
|-----------------|-------------|---------------------------------------------------------------|
| transaction_id  | INT         | Primary key; Unique identifier for the transaction.           |
| order_id        | INT         | Foreign key referencing the order associated with the transaction. |
| product_id      | INT         | Foreign key referencing the product purchased in the transaction. |
| quantity        | INT         | Quantity of the product purchased.                            |
| transaction_date| DATETIME    | Date and time when the transaction occurred.                  |

```sql
CREATE TABLE Transactions (
    transaction_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    transaction_date DATETIME
);
```
<br>

# Relationships
### One-to-Many Relationship between Customers and Orders:
- Each customer can place multiple orders.
### One-to-Many Relationship between Orders and Transactions:
- Each order can have multiple transactions, representing the purchase of multiple products.
### Many-to-One Relationship between Products and Transactions:
- Each product can be included in multiple transactions.
<br>

# Assumptions
- Each customer is uniquely identified by their customer ID.
- Each order is uniquely identified by its order ID.
- Each product is uniquely identified by its product ID.
- Each transaction is uniquely identified by its transaction ID.
<br>

# Query Analysis
Essential queries for our e-commerce application include:

- Retrieve all orders placed by a specific customer.
- Retrieve details of all transactions associated with a particular order.
- Retrieve all products purchased in a given time period.
- Retrieve total sales for a specific product.
- Retrieve customer details based on their email address.
<br>

# Design Cassandra Schema

## Query-First Design:
Based on the identified queries, we have designed our Cassandra schema to optimize query performance and data distribution.

## Orders_By_Customer:

```cql
CREATE TABLE Orders_By_Customer (
    customer_id INT,
    order_id INT,
    order_date TIMESTAMP,
    total_amount DECIMAL,
    PRIMARY KEY (customer_id, order_id)
);
```
**Partition Key:** customer_id <br>
**Clustering Column:** order_id <br>
**Description:** This table stores orders grouped by customer. Each partition represents orders placed by a specific customer, and within each partition, orders are sorted by order_id.

## Transactions_By_Order:

```cql
CREATE TABLE Transactions_By_Order (
    order_id INT,
    transaction_id INT,
    product_id INT,
    quantity INT,
    transaction_date TIMESTAMP,
    PRIMARY KEY (order_id, transaction_id)
);
```
**Partition Key:** order_id <br>
**Clustering Column:** transaction_id <br>
**Description:** This table stores transactions associated with each order. Each partition represents transactions for a specific order, and within each partition, transactions are sorted by transaction_id.

## Products_By_Date:

```cql
CREATE TABLE Products_By_Date (
    product_id INT,
    transaction_date TIMESTAMP,
    quantity INT,
    PRIMARY KEY (product_id, transaction_date)
) WITH CLUSTERING ORDER BY (transaction_date DESC);
```
**Partition Key:** product_id <br>
**Clustering Column:** transaction_date <br>
**Description:** This table stores the quantity of each product over time. Each partition represents a specific product, and within each partition, data is sorted by transaction_date in descending order.

## Customer_By_Email:

```cql
CREATE TABLE Customer_By_Email (
    email VARCHAR,
    customer_id INT,
    name VARCHAR,
    address VARCHAR,
    PRIMARY KEY (email)
);
```
**Partition Key:** email <br>
**Description:** This table stores customer information indexed by email.
Each partition represents a unique email address, allowing efficient 
retrieval of customer details based on email.

<br>

# Secondary Indexes

```cql
CREATE INDEX order_date_index ON Orders_By_Customer (order_date);
CREATE INDEX product_id_index ON Transactions_By_Order (product_id);
CREATE INDEX transaction_date_index ON Products_By_Date (transaction_date);
CREATE INDEX customer_id_index ON Customer_By_Email (customer_id);
```

Purpose:
- `order_date_index`: Allows efficient querying of orders by order date, enabling queries such as retrieving orders placed within a specific time range.
- `product_id_index`: Facilitates queries involving product IDs, such as retrieving transactions for a specific product.
- `transaction_date_index`: Enables efficient retrieval of product quantities based on transaction dates, supporting queries for product trends over time.
- `customer_id_index`: Allows efficient querying of customer details by customer ID, complementing the primary partition key based on email.

# Design Considerations

Our design process was guided by several key considerations to ensure optimal performance and scalability:

1. **Partition Keys**: We carefully selected partition keys to evenly distribute data across Cassandra nodes, preventing hotspots and ensuring efficient data retrieval.
2. **Clustering Columns**: Clustering columns were chosen to facilitate data sorting within partitions, enhancing query performance.
3. **Denormalization**: Denormalization was employed strategically to minimize the need for joins and optimize query performance, despite leading to some data duplication.
4. **Secondary Indexes**: Secondary indexes were employed judiciously to support efficient querying, considering the potential performance overhead.
<br>

# Discussion

Our team encountered several challenges during the schema translation process:

1. **Data Modeling Differences**: Adapting to Cassandra's data model required a paradigm shift from traditional RDBMS thinking, especially in understanding partitioning and clustering.
2. **Query Optimization**: Optimizing queries for Cassandra's distributed nature necessitated careful consideration of partitioning strategies and denormalization techniques.
3. **Data Duplication**: Denormalization led to increased data duplication, forcing us to strike a balance between redundancy and query performance.
<br>

# Trade-offs of Using Cassandra

While Cassandra offers significant advantages over traditional RDBMS, such as scalability and fault tolerance, it comes with its own set of trade-offs:

1. **Scalability**: Cassandra excels at handling massive data volumes and scaling horizontally, making it ideal for big data applications.
2. **Availability**: With its distributed architecture, Cassandra ensures high availability even in the face of node failures, enhancing system reliability.
3. **Consistency**: However, Cassandra sacrifices some consistency guarantees for availability and partition tolerance, requiring careful consideration and application-level handling.
<br>

# Conclusion

In conclusion, our Cassandra schema design optimally addresses the requirements of our e-commerce application, leveraging Cassandra's strengths in scalability, fault tolerance, and high availability. By adopting a "Query First" approach and carefully considering design considerations and trade-offs, we've developed a schema poised for performance and scalability in a big data environment, ensuring the seamless operation of our e-commerce platform in the face of growing data volumes and user demands.