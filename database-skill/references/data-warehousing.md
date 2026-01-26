<overview>
Data warehousing is the practice of collecting, storing, and managing data from various sources for analytical purposes. Modern data warehouses (2024-2025) are predominantly cloud-based, with architectures that separate compute from storage. This reference covers data warehouse architecture patterns and best practices.
</overview>

<architecture_types>
<type name="traditional">
**Traditional Data Warehouse**

On-premises or cloud-hosted with coupled storage/compute:
- Amazon Redshift (original)
- Teradata
- Oracle Exadata

Characteristics:
- Provisioned capacity
- Storage and compute scale together
- Suitable for predictable workloads
</type>

<type name="cloud-native">
**Cloud-Native Data Warehouse**

Separate storage and compute, serverless options:
- Snowflake
- Google BigQuery
- Amazon Redshift Serverless
- Databricks SQL

Characteristics:
- Pay-per-use
- Independent scaling of storage/compute
- Multi-cluster support
- Near-zero administration
</type>

<type name="lakehouse">
**Lakehouse Architecture**

Combines data lake flexibility with warehouse capabilities:
- Databricks
- Delta Lake
- Apache Iceberg

Characteristics:
- Open file formats (Parquet, Delta, Iceberg)
- Schema enforcement on raw data
- ACID transactions on data lake
- Unified batch and streaming
</type>
</architecture_types>

<schema_design>
<pattern name="star-schema">
**Star Schema**

Central fact table surrounded by dimension tables:

```sql
-- Fact table: Measures and foreign keys
CREATE TABLE fact_sales (
    sale_id BIGINT PRIMARY KEY,
    date_key INT REFERENCES dim_date(date_key),
    product_key INT REFERENCES dim_product(product_key),
    customer_key INT REFERENCES dim_customer(customer_key),
    store_key INT REFERENCES dim_store(store_key),

    -- Measures
    quantity INT,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(12,2),
    discount_amount DECIMAL(10,2)
);

-- Dimension tables: Descriptive attributes
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,
    full_date DATE,
    day_of_week VARCHAR(10),
    month VARCHAR(10),
    quarter VARCHAR(10),
    year INT,
    is_weekend BOOLEAN,
    is_holiday BOOLEAN
);

CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_id VARCHAR(50),
    product_name VARCHAR(255),
    category VARCHAR(100),
    subcategory VARCHAR(100),
    brand VARCHAR(100)
);
```

Benefits:
- Simple to understand and query
- Optimized for aggregations
- Denormalized for read performance
</pattern>

<pattern name="snowflake-schema">
**Snowflake Schema**

Normalized dimension tables:

```sql
-- Normalized: Category is separate table
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_name VARCHAR(255),
    category_key INT REFERENCES dim_category(category_key)
);

CREATE TABLE dim_category (
    category_key INT PRIMARY KEY,
    category_name VARCHAR(100),
    department_key INT REFERENCES dim_department(department_key)
);
```

Use when:
- Dimension data is large and repeated
- Need to reduce storage
- Updates to dimension attributes are frequent
</pattern>

<pattern name="data-vault">
**Data Vault**

Hub-Link-Satellite pattern for flexibility:

```sql
-- Hub: Business keys
CREATE TABLE hub_customer (
    customer_hk BIGINT PRIMARY KEY,  -- Hash key
    customer_bk VARCHAR(50),          -- Business key
    load_date TIMESTAMP,
    record_source VARCHAR(100)
);

-- Link: Relationships
CREATE TABLE link_customer_order (
    link_hk BIGINT PRIMARY KEY,
    customer_hk BIGINT,
    order_hk BIGINT,
    load_date TIMESTAMP
);

-- Satellite: Descriptive attributes
CREATE TABLE sat_customer (
    customer_hk BIGINT,
    load_date TIMESTAMP,
    end_date TIMESTAMP,
    name VARCHAR(255),
    email VARCHAR(255),
    address TEXT,
    PRIMARY KEY (customer_hk, load_date)
);
```

Benefits:
- Handles changing sources
- Full audit trail
- Flexible for evolving requirements
- Better for raw data vault layer
</pattern>
</schema_design>

<etl_vs_elt>
**ETL vs ELT**

<approach name="etl">
**ETL (Extract, Transform, Load)**
- Transform data before loading
- External transformation engine
- Better for on-premises, limited warehouse compute

```
Source → Extract → Transform (external) → Load → Warehouse
```
</approach>

<approach name="elt">
**ELT (Extract, Load, Transform)**
- Load raw data, transform in warehouse
- Leverage warehouse compute power
- Modern cloud warehouse approach (2024-2025)

```
Source → Extract → Load (raw) → Transform (in warehouse) → Curated
```

Tools: dbt, Dataform, Matillion
</approach>

**ELT Advantages:**
- Leverage MPP (Massively Parallel Processing)
- Raw data preserved for reprocessing
- Easier to iterate on transformations
- Better for cloud data warehouses
</etl_vs_elt>

<partitioning>
**Partitioning Strategies**

```sql
-- Range partitioning (most common for time-series)
CREATE TABLE fact_sales (
    sale_date DATE,
    product_id INT,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (sale_date);

-- Create partitions
CREATE TABLE fact_sales_2024_q1 PARTITION OF fact_sales
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

-- BigQuery: Partitioned table
CREATE TABLE dataset.fact_sales
PARTITION BY DATE(sale_timestamp)
OPTIONS (partition_expiration_days = 365);

-- Snowflake: Automatic clustering
ALTER TABLE fact_sales CLUSTER BY (sale_date, region);
```

**Best Practices:**
- Partition by most common filter column (usually date)
- Keep partition sizes reasonable (not too small)
- Prune partitions in queries
- Set retention policies for old partitions
</partitioning>

<best_practices>
**Data Warehouse Best Practices (2024-2025)**

<practice name="zones">
**Data Zones**

```
Bronze/Raw → Silver/Cleansed → Gold/Business
```

| Zone | Purpose | Characteristics |
|------|---------|-----------------|
| Bronze/Raw | Landing zone | Source data as-is, append-only |
| Silver/Cleansed | Conforming | Cleaned, typed, deduplicated |
| Gold/Business | Presentation | Aggregated, business-ready |
</practice>

<practice name="slowly-changing">
**Slowly Changing Dimensions (SCD)**

```sql
-- Type 1: Overwrite (no history)
UPDATE dim_customer SET address = 'New Address' WHERE customer_id = 123;

-- Type 2: Add row (full history)
INSERT INTO dim_customer (customer_key, customer_id, address, effective_date, end_date, is_current)
VALUES (999, 123, 'New Address', '2024-01-15', '9999-12-31', true);
UPDATE dim_customer SET end_date = '2024-01-14', is_current = false
WHERE customer_id = 123 AND is_current = true;

-- Type 3: Add column (limited history)
ALTER TABLE dim_customer ADD COLUMN previous_address VARCHAR(255);
UPDATE dim_customer SET previous_address = address, address = 'New Address';
```
</practice>

<practice name="incremental">
**Incremental Processing**

```sql
-- Track last processed
SELECT MAX(updated_at) FROM target_table;

-- Load only new/changed
INSERT INTO target_table
SELECT * FROM source_table
WHERE updated_at > @last_processed;
```
</practice>

<practice name="quality">
**Data Quality Checks**

```sql
-- Row count validation
SELECT COUNT(*) FROM staging_table;

-- Null checks
SELECT COUNT(*) FROM staging_table WHERE required_column IS NULL;

-- Referential integrity
SELECT s.* FROM fact_sales s
LEFT JOIN dim_product p ON s.product_key = p.product_key
WHERE p.product_key IS NULL;

-- Range validation
SELECT * FROM fact_sales WHERE total_amount < 0;
```
</practice>
</best_practices>

<cloud_platforms>
**Platform Comparison (2024-2025)**

| Feature | Snowflake | BigQuery | Redshift | Databricks |
|---------|-----------|----------|----------|------------|
| Pricing model | Credits | On-demand/slots | Per-node/serverless | DBUs |
| Separation | Storage/Compute | Yes | Limited | Yes |
| Format | Proprietary | Capacitor | Columnar | Delta/Parquet |
| Streaming | Snowpipe | Real-time | Kinesis | Structured Streaming |
| ML integration | Snowpark | BigQuery ML | Redshift ML | MLflow |
| Semi-structured | VARIANT | JSON/ARRAY | SUPER | Any |
</cloud_platforms>

<real_time>
**Real-Time Data Warehousing (2025 Trend)**

Modern warehouses support real-time/near-real-time:

```sql
-- Snowflake: Snowpipe for continuous loading
CREATE PIPE my_pipe AS
COPY INTO my_table FROM @my_stage;

-- BigQuery: Streaming inserts
-- INSERT with tabledata.insertAll API

-- Redshift: Streaming ingestion
CREATE MATERIALIZED VIEW mv_streaming
AUTO REFRESH YES
AS SELECT * FROM kinesis_stream;
```

Technologies:
- Apache Kafka for streaming
- Apache Flink for stream processing
- Materialized views for real-time aggregations
</real_time>
