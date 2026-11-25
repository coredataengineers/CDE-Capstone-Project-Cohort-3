# CDE-Capstone-Project-Cohort-3

# BUSINESS SCENARIO
**Client**: CDEBite  
**Headquarters**: Valencia, Spain (Operating Markets: Lisbon, Porto, Braga, and Coimbra, Portugal)  
**Engineering Role**: Consulting Data Engineer   
**Objective**: Unified Multi-Source Ingestion & Dimensional Analytics Modeling in the On-Demand Food Delivery Domain

Headquartered in Valencia, Spain, CDEBite is an on-demand food delivery platform that connects customers browsing menus on its mobile and web apps with local restaurants and couriers across Lisbon, Porto, Braga, and Coimbra, Portugal. As order volume has scaled rapidly, operational data has become fragmented across disconnected tools: transactional records in Supabase PostgreSQL, high-frequency clickstream and courier telemetry in AWS S3 Parquet partitions, and commercial/operational trackers maintained manually across four Google Sheets.

Because these systems evolved independently without unified data modeling or governance, leadership faces critical blind spots:

- Unclear Order Profitability: Finance and operations cannot accurately calculate net contribution margin per order when payment retries, promotional discounts, changing restaurant commission rates, courier shift and bonus rates, and support refunds live across separate systems and manual sheets.
- Unexplained Delivery Delays: When deliveries run late, or orders are cancelled, teams cannot determine whether delays stem from restaurant preparation times, courier transit times, shift coverage gaps, or peak-hour congestion across cities and cuisines.
- Unverified Marketing & Funnel Performance: Marketing lacks visibility into which promotional codes drive repeat orders versus one-time discount usage, as well as where users drop off across the app-to-checkout funnel.
- Data Reliability & Reconciliation Friction: Manual spreadsheet formatting inconsistencies, duplicate GPS/payment retries, late-arriving event partitions, and changing partner contract terms create reporting discrepancies and slow down cross-functional decisions.


You have been brought in as a `Data Engineer` to design and implement a `Unified Data Platform ` that solves these problems. To solve this problem, you need to build a physical data platform that combines various tools and technologies.

### Data Sources
The following eleven entities span CDEBite’s transactional database, partitioned event/telemetry Parquet files in S3, and four operational Google Sheets. Engineers should expect real-world grime: varying timestamps and date formats, nullable fields, duplicate retries, casing/whitespace noise, and late-arriving partitions. 

They are as follows:

### 1. Supabase (PostgreSQL Transactional Tables)

- `Description:` Food Delivery Platform transactional database, including tables such as  Users, Order_items etc.
- `Format:` Database Table
- `Data Location:` In a Transactional Postgres Database
  - Database credentials located here: [SSM Paramter]() in AWS
- **Tables**
   - `Users` - user_id (PK), full_name, email, phone_number, city, signup_at, account_status, updated_at
   - `Restaurants` - restaurant_id (PK), restaurant_name, cuisine_type, city, street_address, latitude, longitude, avg_prep_time_mins, is_active, created_at, updated_at
   - `Orders` - order_id (PK), user_id, restaurant_id, courier_id, promo_code, order_status, subtotal_amount, delivery_fee, discount_amount, total_amount, placed_at, preparing_at, picked_up_at, delivered_at, cancelled_at, updated_at
   - `Order_Items` - order_item_id (PK), order_id, item_name, item_category, quantity, unit_price, line_total, created_at
   - `Payments` - payment_id (PK), order_id, payment_method, payment_status, amount, currency, provider_txn_ref, attempt_number, created_at, updated_at

  
### 2. AWS S3 (Partitioned Parquet Event & Telemetry Streams)

- `Description:` Food Delivery Platform Telemetry data tracking delivery driver information, and platform events.
- `Format:` Parquet 
- `Data Location:` In an S3 Data Lake on [AWS HERE]()
- **Entities**
   - `App_Events` - event_id (PK), session_id, user_id, order_id, restaurant_id, event_name, event_timestamp, ingested_at, promo_code, device_platform, app_version, city, event_properties (nested JSON/struct), year, month, day
   - `Driver_gps_pings` - ping_id (PK), courier_id, order_id, ping_timestamp, received_at, latitude, longitude, speed_kmh, heading_deg, accuracy_m, courier_status, battery_pct, year, month, day


### 3. Google Sheets (4 Operational & Commercial Trackers)

- `Description:` Marketing and Operations google sheets for tracking customer interaction and business performance.
- `Format:` Google Spreadsheet
- `Data Location:` In Spreadsheets [HERE]()
- `Frequency:` These are Static Datasets


### Note: 
- To retrieve all Data stored on AWS, use your existing credentials to login and ensure you are in the Stockholm Region.
  - However, the expectation is that your pipeline must be designed to extract the objects from s3.
  - Full Permissions on s3 and SSM have been given to your user along with permissions to create Access and Secret Key.

#### !DO NOT EXPOSE/PUSH THE KEYS ON GITHUB.

The Google Sheets Operational & Commercial Tracker Datasets provided above `MUST` be moved to your own `PRIVATE` google sheet account, this will allow you to set up all necessary `CREDENTIALS` like `SERVICE ACCOUNT` to be able to communicate with Google API to be able to programatically extract the data from the spread sheet.


### PROJECT OBJECTIVES
Implement a `Production-Grade` solution that covers the following:
- Data Ingestion (Raw Layer)
  - Develop an extraction layer that reads data from each source.
  - Store raw, unprocessed data in Cloud Object Storage (e.g., S3, GCS, Azure Blob) in Parquet format.
  - Include metadata tracking like the load time.
  - `Pay Attention` to the messy state of the data like
    - Column naming
    - User Emails
    - Many more
   
### Data Storage and Modeling:
Design a suitable `Data Warehouse` of choice like Snowflake, Redshift, BigQuery e.t.c to house these data in a unified and central storage to serve downstream use cases.
Since the output of your work will be consumed by downstream users for Analytics, ML, etc. You `MUST` ensure the data is properly transformed and modeled using dbt for easier consumption and querying.

### Data Cleaning and Enrichment
- Perform data cleaning and standardization on the raw data where necessary.
- Enrich the datasets by performing joins, aggregation, categorization etc. based on your understanding of the datasets and as you deemed necessary.
- Make sure the data is of high quality and can be trusted.

### Orchestration:
- `Apache Airflow` must be used for orchestrating the entire workflow, which includes:
  - Extracting the data from the different data sources and writing to a data lake.
  - Transforming the data to ensure high quality and trust.
  - Modelling the data for easier consumption and high performant querying.
  - Testing and validating the accuracy of the data.
- You must also follow best practices like:
  - Idempotency
  - Incremental processing
  - Implementing retries.
  - Failure notification/alert.
  - Etc.

### Containerization:
For reproducibility across environments, 
- You need to package all your code as a Docker image and push the image to a cloud-based container registry (e.g Docker Hub).
- This includes all of your Python code and DAG.

### CI/CD Pipeline:
`Continuous Integration` and `Continuous Deployment` (CI/CD) should be integrated to your Github Repository. It should include:
- `CI` that carries out checks on code linting to ensure code written follows best practices.
- `CD` to carry out the Build and Push of your python code to a Cloud based Container Registry.

### Infrastructure as Code (Terraform):
- All `Cloud Infrastructures` like IAM, Object storage, DB/DW Networking resource provisioning `MUST` be `Terraformed`.
- `Terraform State` File backend managed in the cloud using Object Storage.

### BONUS (NOT MANDATORY)
Derive any insights from the Data Set, and any recommendations on what `CDEBite` can do as a result of the data available.


### SUBMISSION REQUIREMENTS
- A `GitHub Repository` containing a well documented `README` of the entire project. It should contain important information like the:
  - Project overview/background.
  - Project structure.
  - Architecture diagram.
  - Choice of tools and technology.
  - How to set up the project.
  - e.t.c.

A `Power Point` or something similar for presentation of the entire project to the management.
Project submission will close on the `5th December, 2026`.

Github link should be submitted via the link [HERE]().

###  PLEASE REACH OUT ON SLACK IF CLARIFICATION NEEDED.
