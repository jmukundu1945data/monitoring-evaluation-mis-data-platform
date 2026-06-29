## Monitoring & Evaluation (M&E) MIS Data Platform

This repository is intended to demonstrate a practical, field-tested approach to building robust MIS data systems for government and development partners.


## Overview

This repository demonstrates the design and implementation of a scalable Management Information System (MIS) data platform tailored for Monitoring & Evaluation (M&E) in public sector and donor-funded programmes.

The platform is built using a dimensional data modelling approach (star schema) to support structured, indicator-based reporting. It is designed to handle high-volume, multi-dimensional data commonly found in national systems such as health, education, and social protection programmes.

The model supports:
- Routine reporting across facilities, districts, and provinces
- Indicator tracking across multiple programme areas
- Disaggregation by age group and gender
- Time-series analysis using both calendar and programme-specific reporting periods (e.g. PEPFAR financial year)

The design reflects practical experience in building data systems that must balance:
- Data completeness and quality
- Performance and scalability
- Flexibility for evolving reporting requirements

It is suitable for integration into national MIS environments and analytics platforms used by government institutions and development partners.

![Architecture](assets/diagrams/architecture_overview.png)

## Data Model (ERD)

The following diagram illustrates the core MIS data architecture based on a star schema design.

![MIS ERD](assets/diagrams/erd_mis_star_schema.png)

### Design Summary

The model follows a star schema approach with a central fact table (`Data`) capturing indicator values, linked to multiple analytical dimensions:

- **Time** (calendar + PEPFAR financial year)
- **Geography** (Facility → District → Province hierarchy)
- **Program**
- **Data Elements (Indicators)**
- **Demographics** (Age Group, Gender)

This structure supports:
- Disaggregated reporting
- Indicator-based aggregation
- Scalable integration into BI tools such as Power BI
The design reflects over a decade of practical experience supporting large-scale health and development programmes, including indicator-based reporting frameworks such as PEPFAR. It demonstrates how structured data modelling, validation logic, and analytical design can be combined to deliver reliable, scalable, and decision-ready information systems.

The platform is built around a dimensional (star schema) architecture and supports:
- Indicator-driven reporting (e.g. TX_CURR, TX_NEW, HTS_TST)
- Multi-level disaggregation (geography, age, gender)
- Time-series analysis (calendar and programme-specific fiscal periods)
- Embedded data quality validation rules
- Integration with reporting tools such as Power BI and national systems

The full database implementation is available in:
- `sql/ddl/create_mis_star_schema.sql`



## System Architecture

The MIS platform follows a layered architecture that separates data ingestion, transformation, storage, and reporting.

### 1. Source Layer (Operational Systems)

Data originates from transactional or collection systems, which may include:
- Facility-level reporting tools
- National systems (e.g. DHIS2 or equivalent)
- Programme-specific data collection platforms

These systems typically store data in normalized (OLTP) structures optimized for data entry.

---

### 2. Transformation Layer (dbt / SQL)

A transformation layer is used to:
- Clean and standardize source data
- Enforce consistent naming and structures
- Apply business rules and indicator logic
- Prepare data for analytical use

This layer is implemented using SQL and can be extended using tools such as dbt for modular, version-controlled transformations.

---

### 3. Data Warehouse Layer (Star Schema)

The core of the platform is a dimensional data model designed for analytical workloads.

The model consists of:
- A central fact table capturing indicator values
- Multiple dimension tables providing context (time, geography, programme, demographics)

This structure enables:
- Fast aggregation
- Flexible slicing and dicing of data
- Consistent reporting across indicators

---

### 4. Semantic & Reporting Layer

The data model is designed to integrate with reporting tools such as Power BI.

At this layer:
- Measures are defined (e.g. totals, trends, growth rates)
- Time intelligence is applied
- Business logic is exposed in a user-friendly format

This enables:
- Dashboards for programme monitoring
- Automated reporting
- Decision support for stakeholders

---

### 5. Data Quality & Validation Layer

Data quality is treated as a core component of the system.

Validation rules are applied to:
- Ensure logical consistency between indicators (e.g. TX_NEW ≤ TX_CURR)
- Detect missing or incomplete disaggregation
- Identify anomalies at facility or aggregate levels

These checks can be embedded within transformation pipelines or exposed through reporting layers.

## Data Model

The platform is built on a dimensional (star schema) data model designed to support indicator-based reporting in M&E systems.

At the center of the model is a fact table that stores measured indicator values, surrounded by dimension tables that provide analytical context.

---

### Fact Table

**Data**

This table stores quantitative indicator values reported from facilities or aggregation points.

| Column        | Description |
|--------------|------------|
| ID           | Unique record identifier |
| FacilityID   | Reporting facility |
| DataElementID| Indicator identifier (e.g. TX_CURR, TX_NEW) |
| TimeKey      | Reporting period |
| GenderID     | Gender disaggregation |
| AgeGroupID   | Age group disaggregation |
| Value        | Reported numeric value |

This structure allows a single table to support multiple indicators across different programmes while maintaining consistency.

---

### Dimension Tables

#### Time Dimension

Supports both standard calendar reporting and programme-specific reporting periods.

- Calendar hierarchy (Year → Quarter → Month)
- PEPFAR financial year support
- Enables time-series analysis and trend reporting

---

#### Facility Dimension (Geographic Hierarchy)

Represents service delivery points and administrative hierarchy.

- Facility → District → Province
- Enables roll-up reporting from facility to national level
- Supports geographic performance analysis

---

#### Data Element (Indicator Dimension)

Defines the indicators being reported.

- Stores indicator codes (e.g. TX_CURR, TX_NEW, HTS_TST)
- Links to programme areas
- Allows flexible addition of new indicators without changing the fact table

---

#### Program Dimension

Represents programme areas such as:

- HIV Treatment
- Testing Services
- PMTCT

Supports grouping and filtering of indicators by programme.

---

#### Gender Dimension

Supports standard gender disaggregation required in M&E reporting.

---

#### Age Group Dimension

Supports age-based disaggregation, critical for:

- Target population analysis
- Programme performance tracking across age bands

---

### Design Principles

The data model is designed based on the following principles:

- **Indicator-driven design**: Supports multiple indicators within a unified structure
- **Disaggregation-first approach**: Age and gender included at the lowest level
- **Scalability**: New indicators and programmes can be added without schema changes
- **Consistency**: Standardized keys and relationships ensure reliable reporting
- **Performance**: Optimized for aggregation and analytical queries

---

### Analytical Capabilities Enabled

This model supports:

- Time-series trend analysis (e.g. TX_CURR over 12 months)
- Indicator comparison (e.g. TX_NEW vs TX_CURR)
- Disaggregated reporting (age, gender)
- Geographic rollups (facility → district → province)
- Data quality validation across indicators

## Data Quality Framework

Data quality is a core component of the MIS platform and is treated as an integral part of system design rather than a downstream reporting concern.

In M&E systems, data is often collected from multiple facilities under varying conditions. As such, systematic validation is required to ensure that reported values are logically consistent, complete, and fit for decision-making.

---

### Approach

The platform applies a layered data quality approach:

- **Structural validation**: Ensures required fields and relationships are present
- **Completeness checks**: Verifies expected disaggregation (e.g. age, gender)
- **Indicator logic validation**: Enforces consistency between related indicators
- **Anomaly detection**: Identifies unusual or unexpected values

These checks can be implemented within transformation pipelines (e.g. dbt), SQL validation scripts, or exposed directly in reporting tools.

---

### Example Validation Rules

The following rules reflect common M&E validation requirements (e.g. PEPFAR-style reporting):

#### 1. Indicator Consistency

- TX_NEW ≤ TX_CURR  
  (New patients on treatment cannot exceed total patients currently on treatment)

- HTS_POS ≤ HTS_TST  
  (Positive tests cannot exceed total tests conducted)

---

#### 2. Completeness of Disaggregation

- All reported values should include:
  - Age group
  - Gender

Missing disaggregation reduces the analytical value of the data and may indicate reporting gaps.

---

#### 3. Non-Negative Values

- Indicator values must be ≥ 0

---

#### 4. Time Consistency

- Reporting periods should be continuous
- Missing months should be flagged

---

### Example SQL Validation Queries

**Indicator Consistency Check (TX_NEW vs TX_CURR)**

```sql
SELECT
    t.YearName,
    t.MonthName,
    f.Facility,
    SUM(CASE WHEN de.DataElement = 'TX_NEW' THEN d.Value ELSE 0 END) AS TX_NEW,
    SUM(CASE WHEN de.DataElement = 'TX_CURR' THEN d.Value ELSE 0 END) AS TX_CURR
FROM Data d
JOIN DataElement de ON d.DataElementID = de.DataElementID
JOIN Facility f ON d.FacilityID = f.FacilityID
JOIN Time t ON d.TimeKey = t.TimeKey
GROUP BY t.YearName, t.MonthName, f.Facility
HAVING
    SUM(CASE WHEN de.DataElement = 'TX_NEW' THEN d.Value ELSE 0 END) >
    SUM(CASE WHEN de.DataElement = 'TX_CURR' THEN d.Value ELSE 0 END);


SELECT *
FROM Data
WHERE GenderID IS NULL
   OR AgeGroupID IS NULL;   

```
## Key Features

- **Indicator-Based Data Model**  
  Supports multiple programme indicators (e.g. TX_CURR, TX_NEW, HTS_TST) within a unified structure.

- **Multi-Dimensional Disaggregation**  
  Enables analysis by geography (facility, district, province), age group, and gender.

- **Time Intelligence Support**  
  Includes both calendar-based and programme-specific reporting periods (e.g. PEPFAR financial year).

- **Scalable Star Schema Design**  
  New indicators and programme areas can be added without structural changes.

- **Integrated Data Quality Framework**  
  Built-in validation rules ensure consistency and reliability of reported data.

- **Optimized for Analytics**  
  Designed for efficient aggregation and integration with BI tools such as Power BI.

## Example Use Cases

This platform is designed to support real-world M&E scenarios, including:

- **Routine Programme Monitoring**  
  Track key indicators such as TX_CURR and TX_NEW over time.

- **Disaggregated Reporting**  
  Analyse performance across age groups, gender, and geographic regions.

- **Trend Analysis**  
  Identify growth, decline, and seasonal patterns in programme indicators.

- **Data Quality Monitoring**  
  Detect inconsistencies such as TX_NEW exceeding TX_CURR.

- **Performance Analysis**  
  Compare facilities, districts, and provinces to identify high- and low-performing areas.

- **Policy and Planning Support**  
  Provide evidence for resource allocation and programme design decisions.

## Integration & Deployment

The platform is designed to integrate with common systems used in government and development environments.

### Reporting Tools

- Power BI (primary integration)
- Other BI tools (e.g. Tableau)

### National Systems

- DHIS2 or equivalent national health information systems
- Data exchange via APIs or scheduled extracts

### Deployment Options

- On-premise SQL Server environments
- Cloud-based data platforms (Azure SQL, Synapse, etc.)

### Pipeline Integration

- SQL-based ETL/ELT workflows
- dbt for modular transformation pipelines


## Repository Structure
monitoring-evaluation-mis-data-platform/
│
├── sql/ # Database scripts (DDL, views, validation)
├── docs/ # Architecture and design documentation
├── samples/ # Example analytical and validation queries
├── assets/ # Diagrams and supporting materials
│
├── README.md
└── .gitignore


This structure reflects a modular, production-oriented approach to MIS system design.

## Getting Started

1. Create a SQL Server database environment

2. Execute DDL scripts:

   - `sql/ddl/create_mis_star_schema.sql`

3. Load or simulate data into dimension and fact tables

4. Run sample queries:

   - `samples/analytical_queries.sql`
   - `samples/validation_queries.sql`

5. Connect a reporting tool (e.g. Power BI) to build dashboards

---

This repository is intended as a reference architecture and can be adapted to specific programme requirements.


## Author

Jonathan Mukundu  
Data Strategist | MIS Architect | M&E Systems Specialist

This repository reflects practical experience designing and implementing data systems for large-scale health and development programmes across multiple countries.

