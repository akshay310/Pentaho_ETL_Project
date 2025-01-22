
Project Title: Enhanced Customer ETL

Project Goal: Extract customer and order data from CSV files, transform it (standardize names, create unique IDs, calculate total order value), and load it into Microsoft Excel

1. Data Sources:

Customers.csv: 
Orders.csv:

2. Data Warehouse Schema:

Customer Dimension: CustomerDimension (CustomerKey (INT, PRIMARY KEY), CustomerID (INT), Name (VARCHAR), City (VARCHAR), Country (VARCHAR), JoinDate (DATE))
Order Fact: OrderFact (OrderKey (INT, PRIMARY KEY), CustomerID (INT), OrderDate (DATE), Quantity (INT), UnitPrice (NUMERIC), TotalOrderValue (NUMERIC))
3. Set up the Environment: 

Install Pentaho PDI. If you are working with a database from PostgreSQL or MySQLServer, then install that particular software and create database connections to Pentaho

4. Develop ETL Transformations:

   4 a) Staging Layer

A. Extract (CSV Input):

Create two "CSV file input" steps, one for Customers.csv and one for Orders.csv. It is recommended that you transform both in separate files
B. Transform (Customers):

Standardize Names: "String operations" step with "InitCap".
Handle Nulls (JoinDate): "Value Mapper" step to replace null JoinDate values with a default date (e.g., '1900-01-01').
String to Date Conversion: "Select Values" step to convert JoinDate to Date Data Type using format yyyy-MM-dd.
Generate Surrogate Key: "Sequence" step for CustomerKey.

C. Transform (Orders):

Calculate Total Order Value: "Calculator" step: TotalOrderValue = Quantity * UnitPrice.
String to Date Conversion: "Select Values" step to convert OrderDate to Date Data Type using format yyyy-MM-dd.
Generate Surrogate Key: "Sequence" step for OrderKey.

D. Load (Table Output):

Two "Table output" steps, one for CustomerDimension and one for OrderFact.
