# **UBER REAL-TIME DATA ENGINEERING PROJECT**

In this project, I engineered a high-throughput, end-to-end real-time streaming pipeline for a ride-booking service (similar to Uber) using a modern Pub/Sub (Producer/Consumer) architecture. My solution integrates live event streams with historical data to build a scalable Medallion Architecture on Azure.

1. Ingestion Strategy: Decoupling and Hybrid Paths
I developed a dual-path ingestion strategy to handle different data velocities:
- Real-Time Stream: I built a custom FastAPI web application that acts as the Producer, generating ride confirmation events. These events are published to Azure Event Hub, which I configured to serve as a managed Apache Kafka instance, ensuring events are stored in an ordered, reliable manner.
- Batch & Historical Load: To simulate a "lift and shift" migration of 2,000+ historical records, I utilized Azure Data Factory (ADF). I created a metadata-driven ADF pipeline that dynamically fetches mapping files (cities, payment methods) from a GitHub API and loads them into Azure Data Lake Storage (ADLS) Gen2.

2. Transformation: The Metadata-Driven Silver Layer
The core innovation of my pipeline is the One Big Table (OBT) approach in the Silver layer, designed for decentralized domain-specific modeling.
- Jinja2 SQL Templating: To make the code modular and future-proof, I implemented Jinja2 templating. Instead of hardcoding complex multi-table joins, I defined a configuration file; my Jinja template then dynamically renders the SQL query for the join, allowing new tables to be added without touching the core processing logic.
- Streaming Joins and Watermarking: In the Silver layer, I combined the streaming ride data with bulk loads using an append flow. To handle late-arriving data and manage state in memory, I applied Watermarking (e.g., a 3-minute delay) on the booking timestamps.

4. Advanced Modeling: Gold Layer and SCDs
In the Gold layer, I transformed the OBT into a robust Star Schema consisting of one Fact table and six Dimension tables (Passenger, Driver, Vehicle, Payment, Booking, and Location).
- Slowly Changing Dimensions (SCD) Type 2: I implemented SCD Type 2 for the location dimension to maintain a complete history of updates (e.g., city name changes).
- Spark Declarative Pipelines (SDP): I utilized the modern Auto-CDC API within SDP (formerly Delta Live Tables) to automate the complex logic of tracking historical changes using start_at and end_at timestamps.

5. Processing Engine and Governance
I leveraged Azure Databricks as the primary processing engine, using Unity Catalog to manage the three-level namespace (Catalog > Schema > Table) for better data governance
. The final pipeline is fully orchestrated, utilizing incremental processing to ensure only new events are transformed, which optimizes both cost and performance


![Project Architecture](https://github.com/anshlambagit/Uber_Data_Engineer_Project/blob/main/architecture.png)



