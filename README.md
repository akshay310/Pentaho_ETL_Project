
Project Title: Enhanced Customer ETL

Project Goal: Extract customer and order data from CSV files, transform it (standardize names, create unique IDs, calculate total order value), and load it into Microsoft Excel

1. Data Sources:

Customers.csv

Orders.csv

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

4 b) Core Layer

1. Input Steps (CSV File Input):

Customer Data:
Add a "CSV file input" step.
Configure it to read your cleaned customer data CSV file.
Set the delimiter, enclosure, and other necessary settings.
Name this step something descriptive, like "Customer Input".
Order Data:
Add another "CSV file input" step.
Configure it to read your cleaned order data CSV file.
Set the appropriate settings.
Name this step "Order Input".
2. Sort Steps (Sort Rows):

Crucially, you must sort both input streams by the join key (CustomerID) before the merge join.
Customer Sort:
Add a "Sort rows" step.
Connect it to the "Customer Input" step.
Sort by the CustomerID field in ascending order.
Name this step "Sort Customers".
Order Sort:
Add another "Sort rows" step.
Connect it to the "Order Input" step.
Sort by the CustomerID field in ascending order.
Name this step "Sort Orders".
3. Merge Join Step:

Add a "Merge join" step.
Connect the "Sort Customers" step to the first input of the "Merge join" step (the "main stream").
Connect the "Sort Orders" step to the second input of the "Merge join" step (the "lookup stream").
Configure the "Merge join" step:
Join Type: Choose "Inner Join". This will only output rows where there's a match in both datasets.
Key 1: CustomerID (from the customer stream).
Key 2: CustomerID (from the order stream).
4. Select Values Step:

Add a "Select values" step.
Connect it to the "Merge join" step.
In the "Select values" step, select only the desired output fields and their correct names:
OrderKey
CustomerID
CustomerName
Country
OrderDate
Quantity
Price
TotalOrdervalue
5. Output Step (e.g., Text file output, Table output):

Add an output step, such as "Text file output" (to write to a CSV file) or "Table output" (to write to a database table).
Connect it to the "Select values" step.
Configure the output step according to your needs (filename, database connection, table name, etc.).
Transformation Flow:

Customer Input --> Sort Customers -->
                                       Merge Join --> Select Values --> Output
Order Input    --> Sort Orders    -->
