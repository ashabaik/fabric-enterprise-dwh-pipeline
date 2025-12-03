# Enterprise Data Warehouse ETL Pipeline

## 📋 Project Overview

A production-grade, end-to-end data warehouse pipeline built with **Microsoft Fabric** that orchestrates the complete data flow from source systems to a dimensional data warehouse. This solution implements a multi-stage ETL process with automated ingestion, transformation, loading, and audit tracking.

**Built for:** Large-scale e-commerce data warehouse operations  
**Processing Volume:** Millions of daily transactions across customer, product, sales, and inventory domains

## 🎯 Business Problem

Enterprise organizations need a reliable, scalable solution to:
- Ingest data from multiple source systems into a centralized data warehouse
- Transform raw data into business-ready dimensional models (star schema)
- Load both dimension and fact tables in the correct sequence
- Track data refresh cycles for audit and SLA compliance
- Ensure data consistency and quality across all stages

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Source Systems (OLTP)                        │
│          (SQL Server, APIs, File Systems, etc.)                  │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│   STAGE 1: Full Ingestion (Src to Landing)                      │
│   • Extract data from multiple sources                           │
│   • Raw data loaded into Landing Zone                           │
│   • No transformations applied                                   │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              Landing Area (Staging Layer)                        │
│   • Raw, unprocessed data                                        │
│   • Temporary storage for ETL processing                         │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│   STAGE 2: Dimension Loading (Landing to DWH DIM)               │
│   • Transform and load dimension tables                          │
│   • Apply business rules and data quality checks                │
│   • SCD Type 1/2 handling                                        │
│   • Load: DimCustomer, DimProduct, DimDate, etc.                │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│   STAGE 3: Fact Loading (Landing to DWH FACT)                   │
│   • Transform and load fact tables                               │
│   • Join with dimension keys                                     │
│   • Aggregate measures and metrics                               │
│   • Load: FactSales, FactInventory, FactOrders, etc.            │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│   STAGE 4: Audit & Monitoring (Timestamp Recording)             │
│   • Record last refresh timestamp                                │
│   • Update audit table: dbo.LAST_REFRESH                        │
│   • Enable SLA tracking and monitoring                           │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│            Data Warehouse (Production)                           │
│   Ready for BI Dashboards, Analytics, and Reporting             │
└─────────────────────────────────────────────────────────────────┘
```

## ✨ Key Features

### 🔄 Pipeline Orchestration
- **Sequential execution** with dependency management
- **Error handling** at each stage with retry logic
- **Parallel processing** where possible for performance optimization
- **Conditional execution** based on previous stage outcomes

### 📊 Data Warehouse Design
- **Star schema** with dimension and fact tables
- **Slowly Changing Dimensions (SCD)** support
- **Surrogate keys** for dimensional integrity
- **Data quality validations** at each stage

### 🛡️ Production-Ready Features
- Comprehensive logging and monitoring
- Audit trail with timestamp tracking
- Failure notifications and alerting
- Data lineage and impact analysis
- Performance optimization and indexing

### 🎯 Business Value
- **Centralized analytics** - Single source of truth for business intelligence
- **Real-time insights** - Fresh data for decision-making
- **Scalability** - Handles growing data volumes efficiently
- **Data governance** - Audit tracking and compliance support

## 🔧 Technical Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Orchestration** | Microsoft Fabric Data Factory | Pipeline coordination and scheduling |
| **Storage** | Microsoft Fabric Lakehouse | Landing zone and staging storage |
| **Processing** | PySpark Notebooks | Data transformations and business logic |
| **Data Warehouse** | Microsoft Fabric Warehouse | Production dimensional model |
| **Monitoring** | Fabric Monitoring Tools | Pipeline execution tracking |

## 📁 Pipeline Components

### 1. **Full_Ingestion Pipeline** (Src to Landing)
**Purpose:** Extract raw data from source systems

**Activities:**
- Connect to source databases (SQL Server, APIs, files)
- Copy data to Landing zone without transformations
- Validate data extraction completeness
- Log ingestion metrics

**Key Technologies:**
- Copy Data activities
- Parameterized connections
- Incremental/full load logic

### 2. **Full Load Landing to DWH (DIM)** Pipeline
**Purpose:** Load and transform dimension tables

**Activities:**
- Read from Landing zone
- Apply data quality rules and transformations
- Handle Slowly Changing Dimensions (SCD Type 1/2)
- Load into dimension tables:
  - `DimCustomer` - Customer master data
  - `DimProduct` - Product catalog
  - `DimDate` - Date dimension
  - `DimLocation` - Geographic data
  - `DimEmployee` - Employee information

**Key Technologies:**
- PySpark for complex transformations
- SQL for data quality checks
- Lookups for surrogate key assignment

### 3. **Full Load Landing to DWH (FACT)** Pipeline
**Purpose:** Load fact tables with business transactions

**Activities:**
- Read transactional data from Landing
- Join with dimension tables for key lookups
- Calculate derived measures and aggregations
- Load into fact tables:
  - `FactSales` - Sales transactions
  - `FactOrders` - Order details
  - `FactInventory` - Inventory movements
  - `FactRevenue` - Revenue metrics

**Key Technologies:**
- Star schema joins
- Aggregate functions
- Incremental load patterns

### 4. **Audit & Timestamp Recording**
**Purpose:** Track data refresh cycles

**Activities:**
- Query system date/time
- Apply timezone offset (UTC+3)
- Update `dbo.LAST_REFRESH` audit table
- Enable monitoring dashboards

## 🚀 Pipeline Execution Flow

```
START
  │
  ├─► [Src to Landing] ──Success──► [Landing to DWH DIM]
  │                                         │
  │                                    On Complete
  │                                         │
  │                                         ▼
  │                              [Landing to DWH FACT]
  │                                         │
  │                                     Success
  │                                         │
  │                                         ▼
  │                                   [Record Timestamp]
  │                                         │
  └─────────────────────────────────────►  END
```

### Dependency Logic:
1. **Stage 1 → Stage 2:** Proceeds only on successful ingestion
2. **Stage 2 → Stage 3:** Proceeds on any completion (success/failure) to ensure fact loading attempts
3. **Stage 3 → Stage 4:** Proceeds only on successful fact loading for accurate audit

## 📊 Data Warehouse Schema

### Dimension Tables (DIM):
```sql
-- Example: Customer Dimension
DimCustomer (
    CustomerKey INT PRIMARY KEY,      -- Surrogate key
    CustomerID VARCHAR(50),            -- Business key
    CustomerName VARCHAR(200),
    Email VARCHAR(100),
    RegistrationDate DATE,
    CustomerType VARCHAR(50),
    IsActive BIT,
    -- SCD Type 2 columns
    EffectiveDate DATE,
    ExpiryDate DATE,
    IsCurrent BIT
)
```

### Fact Tables (FACT):
```sql
-- Example: Sales Fact
FactSales (
    SalesKey BIGINT PRIMARY KEY,
    DateKey INT FOREIGN KEY,
    CustomerKey INT FOREIGN KEY,
    ProductKey INT FOREIGN KEY,
    OrderID VARCHAR(50),
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    TotalAmount DECIMAL(10,2),
    DiscountAmount DECIMAL(10,2),
    NetAmount DECIMAL(10,2)
)
```

### Audit Table:
```sql
-- Last Refresh Tracking
dbo.LAST_REFRESH (
    RefreshID INT IDENTITY PRIMARY KEY,
    PipelineName VARCHAR(100),
    RefreshTimestamp DATETIME,
    Status VARCHAR(50),
    RecordsProcessed BIGINT
)
```

## 🔒 Security & Governance

- **Role-based access control** for pipeline execution
- **Sensitive data masking** in non-production environments
- **Audit logging** for compliance requirements
- **Data encryption** at rest and in transit
- **Connection string security** via Azure Key Vault

## 📈 Performance Optimizations

- **Partitioning strategies** for large fact tables
- **Indexing** on frequently queried columns
- **Parallel execution** where dependencies allow
- **Incremental loading** to minimize data movement
- **Columnstore indexes** for analytical queries

## 🎓 Learning Outcomes

This project demonstrates:
- ✅ Enterprise-scale ETL pipeline design
- ✅ Microsoft Fabric ecosystem proficiency
- ✅ Dimensional modeling (Kimball methodology)
- ✅ Pipeline orchestration and dependency management
- ✅ Data quality and governance implementation
- ✅ Production deployment best practices

## 📫 Contact

**Ahmed Mohamed**
- GitHub: [@ashabaik](https://github.com/ashabaik)
- LinkedIn: www.linkedin.com/in/ahmed-mohamed-280481273
- Email: shabaik1996@gmail.com

---

## 📝 License

This project is for educational and portfolio purposes.

---

**⭐ If you found this helpful, please star this repository!**
