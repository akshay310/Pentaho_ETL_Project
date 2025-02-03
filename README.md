# Enhanced Customer ETL

## Project Goal
Extract customer and order data from CSV files, transform it (standardize names, create unique IDs, calculate total order value), and load it into Microsoft Excel.

## Project Details and Workflow

### 1. Data Sources:
- **Customers.csv**
- **Orders.csv**

### 2. Data Warehouse Schema:
#### Customer Dimension: `CustomerDimension`
| Column       | Data Type    | Description |
|-------------|-------------|-------------|
| CustomerKey | INT, PRIMARY KEY | Surrogate key for customers |
| CustomerID  | INT         | Original customer ID |
| Name        | VARCHAR     | Customer name |
| City        | VARCHAR     | Customer city |
| Country     | VARCHAR     | Customer country |
| JoinDate    | DATE        | Customer join date |

#### Order Fact: `OrderFact`
| Column       | Data Type    | Description |
|-------------|-------------|-------------|
| OrderKey    | INT, PRIMARY KEY | Surrogate key for orders |
| CustomerID  | INT         | Original customer ID |
| OrderDate   | DATE        | Date of order |
| Quantity    | INT         | Number of items in order |
| UnitPrice   | NUMERIC     | Price per unit |
| TotalOrderValue | NUMERIC | Computed total order value |

---

## 3. Set up the Environment:
- Install **Pentaho PDI**.
- If using a database (e.g., **PostgreSQL** or **MySQLServer**), install it and create database connections in Pentaho.

---

## 4. Develop ETL Transformations:

### **4a) Staging Layer**

#### **A. Extract (CSV Input):**
- Create two **"CSV file input"** steps:
  - One for `Customers.csv`
  - One for `Orders.csv`

#### **B. Transform (Customers):**
1. **Standardize Names:** Use **"String operations"** step with **"InitCap"**.
2. **Handle Nulls (JoinDate):** Use **"Value Mapper"** step to replace null `JoinDate` values with a default date (e.g., '1900-01-01').
3. **String to Date Conversion:** Use **"Select Values"** step to convert `JoinDate` to `Date` format (`yyyy-MM-dd`).
4. **Generate Surrogate Key:** Use **"Sequence"** step for `CustomerKey`.

#### **C. Transform (Orders):**
1. **Calculate Total Order Value:** Use **"Calculator"** step:
   ```
   TotalOrderValue = Quantity * UnitPrice
   ```
2. **String to Date Conversion:** Use **"Select Values"** step to convert `OrderDate` to `Date` format (`yyyy-MM-dd`).
3. **Generate Surrogate Key:** Use **"Sequence"** step for `OrderKey`.

#### **D. Load (Table Output):**
- Use **"Table output"** steps for both `CustomerDimension` and `OrderFact`.

---

### **4b) Core Layer**

#### **1. Input Steps (CSV File Input):**
- **Customer Data:**
  - Add a "CSV file input" step.
  - Configure it to read the cleaned customer data.
  - Set delimiter, enclosure, and required settings.
- **Order Data:**
  - Add another "CSV file input" step.
  - Configure it for the cleaned order data.

#### **2. Sort Steps (Sort Rows):**
- **Customer Sort:**
  - Add "Sort rows" step.
  - Connect to "Customer Input".
  - Sort by `CustomerID` (ascending order).
- **Order Sort:**
  - Add "Sort rows" step.
  - Connect to "Order Input".
  - Sort by `CustomerID` (ascending order).

#### **3. Merge Join Step:**
- Add **"Merge join"** step.
- Connect "Sort Customers" as the **main stream**.
- Connect "Sort Orders" as the **lookup stream**.
- Configure join type as **"Inner Join"** on `CustomerID`.

#### **4. Select Values Step:**
- Add **"Select values"** step.
- Choose required fields:
  ```
  OrderKey, CustomerID, CustomerName, Country, OrderDate, Quantity, Price, TotalOrderValue
  ```

#### **5. Output Step (Text file or Table output):**
- Use "Text file output" (CSV) or "Table output" (Database).
- Connect it to "Select values" step.

### **Transformation Flow:**
```
Customer Input --> Sort Customers -->
                                      Merge Join --> Select Values --> Output
Order Input    --> Sort Orders    -->
```

---

## **Use Case Implementation using Pentaho**

### **1. To find the customer who has the highest order value (TotalOrderValue):**
```sql
SELECT CustomerID, SUM(TotalOrderValue) AS TotalSpent
FROM OrderFact
GROUP BY CustomerID
ORDER BY TotalSpent DESC
LIMIT 1;
```

### **2. To find the total number of orders by each employee:**
```sql
SELECT CustomerID, COUNT(OrderKey) AS TotalOrders
FROM OrderFact
GROUP BY CustomerID;
```

### **3. To create a dashboard using Power BI:**
- Connect Power BI to the transformed **CustomerDimension** and **OrderFact** tables.
- Create visualizations such as:
  - **Total Orders per Customer**
  - **Total Revenue per Country**
  - **Monthly Order Trends**

---

## **Project Repository Structure:**
```
📂 Enhanced_Customer_ETL
│── 📂 data_sources
│   │── customers.csv
│   │── orders.csv
│
│── 📂 etl_scripts
│   │── customer_transformation.ktr
│   │── order_transformation.ktr
│
│── 📂 sql_queries
│   │── customer_highest_order.sql
│   │── total_orders_by_employee.sql
│
│── 📂 power_bi_dashboard
│   │── customer_orders.pbix
│
│── README.md
```

---

## **Conclusion:**
This project demonstrates how to efficiently extract, transform, and load customer and order data using **Pentaho PDI**. By integrating **Power BI**, users can derive valuable insights from the transformed data, improving decision-making and customer analysis.
