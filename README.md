# E-Commerce Data Engineering Project Architecture

## Overview
This project implements an end-to-end data pipeline for e-commerce analytics using Databricks, processing raw data from S3 bucket into a dimensional model for analytics.

---

## Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            DATA SOURCES (S3)                             │
│                      s3://orders-ecommerce-de/raw/                       │
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │customers.csv│   │products.csv  │  │sales_reps.csv│  │transactions.csv││
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                      PROCESSING LAYER (Databricks)                       │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                     Data Processing Notebooks                      │  │
│  │                                                                    │  │
│  │  • customers_data_processing    - PySpark transformations          │  │
│  │  • products_data_processing     - PySpark transformations          │  │
│  │  • sales_data_processing        - PySpark transformations          │  │
│  │  • transactions_data_processing - PySpark transformations          │  │
│  │  • data_modeling                - Dimensional modeling             │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                        DATA WAREHOUSE Catalog                            │
│                          Catalog: ecommerce                              │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                       Gold Layer (gold schema)                     │  │
│  │                                                                    │  │
│  │  Dimension Tables:                                                 │  │
│  │    • dim_date        (Date dimension: 2023-2025)                   │  │
│  │    • dim_customers   (Customer dimension)                          │  │
│  │    • dim_products    (Product dimension)                           │  │
│  │    • dim_sales_reps  (Sales representative dimension)              │  │
│  │                                                                    │  │
│  │  Fact Tables:                                                      │  │
│  │    • fact_transactions (Transactional fact table)                  │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                      ANALYTICS & VISUALIZATION                           │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │             E-Commerce Transactions Analysis Dashboard             │  │
│  │                                                                    │  │
│  │  • Sales trends and metrics                                        │  │
│  │  • Customer analytics                                              │  │
│  │  • Transaction insights                                            │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘

```

---

## Data Flow

### 1. **Source Layer**
- **Location**		: s3://orders-ecommerce-de/raw/
- **Format**		  : CSV files
- **Datasets**:
- customers.csv 	 - Customer master data
- products.csv 	   - Product information
- sales_reps.csv 	 - Sales representative information
- transactions.csv - Transaction records

### 2. **Processing Layer**

#### customers_data_processing
- Reads				: s3://orders-ecommerce-de/raw/customers.csv
- Transforms	: Customer data cleaning, standardization, deduplication
- Writes			: ecommerce.gold.dim_customers

#### products_data_processing
- Reads				: s3://orders-ecommerce-de/raw/products.csv
- Transforms	: Product data enrichment and categorization
- Writes			: ecommerce.gold.dim_products

#### sales_data_processing
- Reads				: s3://orders-ecommerce-de/raw/sales_reps.csv
- Transforms	: Sales rep data processing
- Writes			: ecommerce.gold.dim_sales_reps

#### transactions_data_processing
- Reads				: s3://orders-ecommerce-de/raw/transactions.csv
- Transforms	: Transaction validation, enrichment, metric calculations
- Writes			: ecommerce.gold.fact_transactions
- Dependencies: Uses utilities from /setup/utilities

#### data_modeling
- Creates dimensional model structures
- Generates ecommerce.gold.dim_date (2023-01-01 to 2025-12-31)
- Includes date_id, year, month, day, quarter, day_type etc

### 3. **Storage Layer**

**Catalog** : ecommerce  
**Schema**  : gold  
**Format**  : Delta Lake

**Tables**:
-  dim_date 					- Date dimension table
-  dim_customers 			- Customer dimension
-  dim_products 			- Product dimension
-  dim_sales_reps 		- Sales representative dimension
-  fact_transactions 	- Transaction fact table

### 4. **Analytics Layer**

**Dashboard**: E-Commerce Transactions Analysis
- Connects to gold layer tables
- Provides insights and KPIs
- Interactive visualizations

---

## Technology Stack
┌───────────────────────────────────────────────┐
| Component 					| Technology 					 		|
|---------------------|-------------------------|
| Cloud Storage 			| AWS S3 								 	|
| Data Processing 		| Databricks (PySpark) 		|
| Data Warehouse 			| Unity catalog						|
| Orchestration 			| Databricks Notebooks 		|
| Analytics 					| Databricks Dashboards   |
| Language 						| Python, SQL 						|
└───────────────────────────────────────────────┘
---

## Design Patterns

### Dimensional Modeling
- **Star Schema**         : Fact and dimension tables for optimized analytics
- **Conformed Dimensions**: Shared dimensions across fact tables

### Data Quality
- Schema validation at ingestion
- Data type enforcement
- Deduplication logic
- Null handling strategies

---

## Project Structure

```
de_project/
├── customers_data_processing            (Customer ETL)
├── products_data_processing             (Product ETL)
├── sales_data_processing                (Sales rep ETL)
├── transactions_data_processing         (Transaction ETL)
├── data_modeling                        (Dimensional model setup)
└── E-Commerce Transactions Analysis     (BI reports)
```

---

## Data Refresh Strategy

**Current** : Manual notebook execution  
**Recommended Next Steps**:
1. Schedule notebooks as Databricks Jobs
2. Implement incremental loading for transactions
3. Set up data quality monitoring
4. Add alerting for pipeline failures

---

## Future Enhancements

1. **Incremental loading   : Implement incremental loading for transactions
2.- **SCD Type 1**   	     : Implement type 1 dimensions (can be extended to Type 2 if needed)
3. **Orchestration** 	     : Migrate to Databricks Jobs
4. **Data Quality**  	     : Implement live table Expectations
5. **Scheduling**	   			 : Schedule notebooks as Databricks Jobs

---

## Maintenance

- **Owner**       : kharitejareddy997@gmail.com
- **Last Updated**: 2026-06-22
