# 전체 아키텍처 (2025 Modern Data Stack)

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATA SOURCES (Raw Data)                      │
│  CSV Files, APIs, Databases, Event Streams, External Systems    │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              🥉 BRONZE LAYER (Delta Lake - Raw)                  │
│  • Raw data ingestion (as-is)                                   │
│  • Append-only, immutable                                       │
│  • Schema enforcement at write                                  │
└────────────────────────┬────────────────────────────────────────┘
                         │ dbt source/seed
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│         🥈 SILVER LAYER (Delta Lake - Cleaned)                   │
│  • Data quality validation                                      │
│  • Deduplication & normalization                                │
│  • Type casting & cleansing                                     │
│  • SCD Type 2 implementation                                    │
└────────────────────────┬────────────────────────────────────────┘
                         │ dbt + Ontology Naming Macro
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│          🥇 GOLD LAYER (Delta Lake - Business)                   │
│  • Star Schema (Dimensions + Facts)                             │
│  • Ontology-driven naming (dim_*, fct_*)                        │
│  • Unity Catalog tags (ontology URIs)                           │
│  • Optimized for analytics                                      │
└────────────┬──────────────────┬─────────────────────────────────┘
             │                  │
             ▼                  ▼
┌──────────────────────┐  ┌─────────────────────────────────────┐
│  📊 Power BI         │  │  🕸️ Neo4j Knowledge Graph          │
│  • Direct Lake       │  │  • Ontology structure (TBox)        │
│  • Semantic Model    │  │  • Instance data (ABox)             │
│  • Customer 360      │  │  • Semantic search                  │
│  • Real-time refresh │  │  • Recommendation engine            │
└──────────────────────┘  └─────────────────────────────────────┘
```

## 🎯 핵심 컴포넌트

### 1. Ontology Layer (Foundation)

**역할**: 도메인 지식의 표준화 및 의미 정의

```
┌─────────────────────────────────────────┐
│     OWL Ontology (ecommerce.owl)        │
├─────────────────────────────────────────┤
│  Classes:                               │
│    • Party → Person → Customer          │
│    • Product                            │
│    • Order → OrderItem                  │
│    • Payment, Review, Address           │
│                                         │
│  Object Properties:                     │
│    • placedBy, contains, refersTo       │
│    • paysFor, writtenBy, about          │
│                                         │
│  Data Properties:                       │
│    • customerId, orderId, orderDate     │
│    • totalAmount, rating                │
└─────────────────────────────────────────┘
         │
         ├─────────────────┬──────────────────┐
         ▼                 ▼                  ▼
    dbt Naming        Unity Catalog       Neo4j Graph
    Automation           Tagging           Modeling
```

### 2. Data Transformation Layer (dbt)

**역할**: ELT 패턴 기반 데이터 변환 자동화

```
dbt Project Structure:
├── models/
│   ├── staging/              ← Bronze → Silver
│   │   ├── stg_customers.sql      (원본 → 정제)
│   │   ├── stg_orders.sql
│   │   ├── stg_products.sql
│   │   └── stg_reviews.sql
│   │
│   ├── intermediate/         ← Silver → Silver
│   │   └── int_order_items.sql    (비즈니스 로직)
│   │
│   └── marts/                ← Silver → Gold
│       ├── dim_customer.sql       (Dimension)
│       ├── dim_product.sql
│       ├── fct_order.sql          (Fact)
│       └── fct_review.sql
│
├── macros/
│   └── enforce_ontology_naming.sql  ← 자동 네이밍
│
└── tests/
    ├── unique_customer_id.sql
    ├── not_null_order_date.sql
    └── relationships.yml
```

**Key Features**:
- ✅ Incremental models (대용량 처리)
- ✅ Tests (uniqueness, not_null, relationships)
- ✅ Documentation (auto-generated)
- ✅ Lineage visualization (DAG)

### 3. Governance Layer (Unity Catalog)

**역할**: 메타데이터 관리 및 데이터 거버넌스

```
Unity Catalog Hierarchy:
main (catalog)
├── ecommerce_bronze (schema)
│   ├── raw_customers
│   ├── raw_orders
│   └── raw_products
│
├── ecommerce_silver (schema)
│   ├── stg_customers
│   ├── stg_orders
│   └── stg_products
│
└── ecommerce_gold (schema)
    ├── dim_customer
    │   └── [TAG] ontology_class: Customer
    ├── dim_product
    │   └── [TAG] ontology_class: Product
    ├── fct_order
    │   └── [TAG] ontology_class: Order
    └── fct_review
        └── [TAG] ontology_class: Review
```

**Tagging Strategy**:
```python
# 자동 태깅 로직
if "customer" in table_name.lower():
    tag = "https://github.com/sechan9999/ecommerce-ontology#Customer"
elif "order" in table_name.lower():
    tag = "https://github.com/sechan9999/ecommerce-ontology#Order"
# ...
```

### 4. Knowledge Graph Layer (Neo4j)

**역할**: 그래프 기반 의미 검색 및 추천

```
Neo4j Graph Model:
(Customer)-[:PLACED]->(Order)-[:CONTAINS]->(OrderItem)-[:REFERS_TO]->(Product)
    ↓                    ↓
[:WRITTEN]          [:PAID_BY]
    ↓                    ↓
(Review)-[:ABOUT]->(Product)   (Payment)
```

**Use Cases**:
1. 고객 여정 분석 (Customer Journey)
2. 제품 추천 (Collaborative Filtering)
3. 온톨로지 검증 (Reasoner)
4. 데이터 계보 추적 (Lineage)

### 5. Analytics Layer (Power BI)

**역할**: 비즈니스 인사이트 시각화

```
Power BI Architecture:
┌─────────────────────────────────────────┐
│        Databricks Direct Lake           │
│  (실시간 Delta Lake 연동 - Zero Copy)    │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│         Semantic Model                  │
│  • Star Schema (dim + fct)              │
│  • Relationships (1:N, M:N)             │
│  • Calculated Columns/Measures          │
│  • Row-Level Security (RLS)             │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│      Customer 360 Dashboard             │
│  • 고객 세그먼트 분석                    │
│  • 주문 트렌드 (시계열)                  │
│  • 제품 성과 분석                        │
│  • 리뷰 센티먼트 분석                    │
└─────────────────────────────────────────┘
```

## 🔄 Data Flow (End-to-End)

### Batch Processing Flow

```
1. Data Ingestion (Daily)
   CSV/API → Databricks → Bronze Layer (Delta)
   
2. Data Quality & Cleansing
   Bronze → dbt staging → Silver Layer
   
3. Business Logic Application
   Silver → dbt marts → Gold Layer
   
4. Metadata Enrichment
   Python Script → Unity Catalog Tags
   
5. Analytics Serving
   Gold → Power BI Direct Lake
   Gold → Neo4j (manual sync)
```

### Real-time Processing Flow (Optional)

```
Event Stream (Kafka)
   ↓
Databricks Structured Streaming
   ↓
Delta Lake (Bronze) - Auto Loader
   ↓
dbt Incremental Models (Silver/Gold)
   ↓
Power BI Auto-Refresh
```

## 🔐 Security & Access Control

### Unity Catalog Permissions

```sql
-- Catalog 레벨
GRANT USE CATALOG ON CATALOG main TO `data_engineers`;

-- Schema 레벨
GRANT SELECT ON SCHEMA main.ecommerce_gold TO `bi_analysts`;
GRANT ALL PRIVILEGES ON SCHEMA main.ecommerce_silver TO `data_engineers`;

-- Table 레벨
GRANT SELECT ON TABLE main.ecommerce_gold.dim_customer TO `marketing_team`;
```

### Row-Level Security (Power BI)

```dax
-- RLS Filter
[Region] = USERNAME()
```

## 📊 Performance Optimization

### Delta Lake Optimization

```sql
-- Z-Ordering (자주 필터링되는 컬럼)
OPTIMIZE main.ecommerce_gold.fct_order
ZORDER BY (customer_id, order_date);

-- Vacuum (오래된 버전 삭제)
VACUUM main.ecommerce_gold.fct_order RETAIN 168 HOURS;
```

### dbt Incremental Strategy

```sql
{{ config(
    materialized='incremental',
    unique_key='order_id',
    incremental_strategy='merge'
) }}

select * from {{ ref('stg_orders') }}
{% if is_incremental() %}
where order_date > (select max(order_date) from {{ this }})
{% endif %}
```

## 🎯 Data Quality Framework

### dbt Tests

```yaml
# schema.yml
models:
  - name: dim_customer
    tests:
      - dbt_expectations.expect_table_row_count_to_be_between:
          min_value: 1000
          max_value: 1000000
    columns:
      - name: customer_id
        tests:
          - unique
          - not_null
      - name: email
        tests:
          - dbt_expectations.expect_column_values_to_match_regex:
              regex: '^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\\.[a-zA-Z0-9-.]+$'
```

### Great Expectations Integration

```python
# Databricks Notebook
import great_expectations as ge

df = spark.table("main.ecommerce_gold.fct_order")
ge_df = ge.from_spark(df)

ge_df.expect_column_values_to_not_be_null("order_id")
ge_df.expect_column_values_to_be_between("total_amount", min_value=0)
```

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/dbt-ci-cd.yml
name: dbt CI/CD

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  dbt-run:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up dbt
        run: pip install dbt-databricks
      
      - name: dbt deps
        run: dbt deps
      
      - name: dbt run (slim CI)
        if: github.event_name == 'pull_request'
        run: dbt run --select state:modified+
      
      - name: dbt test
        run: dbt test
      
      - name: dbt docs generate
        run: dbt docs generate
```

## 📈 Monitoring & Observability

### Databricks Monitoring

- **Query History**: Unity Catalog 쿼리 로그
- **Cluster Metrics**: CPU, Memory, Disk usage
- **Job Runs**: 성공/실패 추적

### dbt Cloud Monitoring

- **Model Timing**: 느린 모델 식별
- **Test Results**: 데이터 품질 추적
- **Lineage**: 영향도 분석

## 🚀 Scalability Considerations

### Current Scale
- **Data Volume**: ~1GB (샘플)
- **Records**: ~100K orders, 10K customers
- **Refresh Frequency**: Daily batch

### Future Scale (Production)
- **Data Volume**: 10TB+ (3년치)
- **Records**: 100M+ orders, 10M+ customers
- **Refresh Frequency**: Near real-time (5분)

**Optimization Strategies**:
1. Partitioning by date (`order_date`)
2. Liquid Clustering (Delta 3.0+)
3. Materialized Views
4. Auto-scaling clusters
5. Photon Engine

---

**Last Updated**: 2025-11-18  
**Version**: 1.0.0
