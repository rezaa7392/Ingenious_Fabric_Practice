# Data Engineering Assignment: Building a Modern Data Platform with Microsoft Fabric

## Assignment Overview

This assignment challenges you to build a complete data platform using Microsoft Fabric, the Ingenious framework, and DBT (Data Build Tool). You'll create a medallion architecture data pipeline that processes operational data through bronze, silver, and gold layers, culminating in business-ready analytics and reporting.

**Estimated Time**: 40-60 hours over 2-3 weeks  
**Difficulty Level**: Intermediate to Advanced  
**Technologies**: Microsoft Fabric, DBT, Python, Azure DevOps, PowerShell

## Learning Objectives

By completing this assignment, you will:
- ✅ Set up a complete Microsoft Fabric development environment
- ✅ Implement medallion architecture (bronze/silver/gold) data pipelines
- ✅ Master DBT for data transformation and modeling
- ✅ Learn to use Ingenious Framework
- ✅ Build CI/CD pipelines for automated deployment
- ✅ Create semantic models and business intelligence reports
- ✅ Implement data quality testing and monitoring
- ✅ Apply enterprise data engineering best practices

## Business Scenario

You are a data engineer at **NYC Traffic Safety Analytics**, a consulting company contracted by the **NYC Department of Transportation (DOT)** and **NYPD Traffic Division** to build a modern data platform supporting the **Vision Zero Initiative** - NYC's commitment to eliminating traffic deaths and serious injuries by 2024.

You'll be working with the **NYC Motor Vehicle Collisions** dataset - one of the most comprehensive traffic safety datasets available, containing detailed information about every reported collision in NYC with rich details about:

- **Incident Details**: Collision date/time, location, and severity
- **Casualty Information**: Persons injured/killed by type (motorist, pedestrian, cyclist)
- **Vehicle Analysis**: Vehicle types involved and damage assessment
- **Causality Factors**: Contributing factors and driver behaviors
- **Geographic Patterns**: Street-level location data and traffic enforcement zones

Your task is to build an end-to-end data platform that transforms raw collision data into actionable safety intelligence, demonstrating enterprise-grade data engineering capabilities while supporting a critical public safety mission with real-world impact.

## Prerequisites & Environment Setup

### 1. Create Development Tenant

Follow these steps to set up your isolated development environment:

#### Join the Microsoft 365 Developer Program
1. **Navigate to Developer Program**: Go to the [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program) dashboard
2. **Sign In**: Use a personal Microsoft Account (MSA) or create a new one
3. **Set up Subscription**: Click "Set up E5 subscription" on the dashboard
4. **Create Your Sandbox**:
   - Choose "Configurable sandbox"
   - Create an Admin username (e.g., `admin@yourname`)
   - Choose a Domain name (e.g., `yourname-dataeng` creates `yourname-dataeng.onmicrosoft.com`)
   - Complete identity verification (phone verification required)

**Result**: You now have a complete Microsoft 365 tenant with Azure AD, perfect for development and testing.

#### Activate Microsoft Fabric Trial
1. Go to [fabric.microsoft.com](https://fabric.microsoft.com)
2. Sign in with your new developer tenant admin account
3. Start your Fabric trial (60-day free trial with full features)
4. Create two new workspaces:
   - `DP_[YourName]` (Data Platform workspace)
   - `ER_[YourName]` (Enterprise Reporting workspace)

### 2. Development Environment Setup

#### Required Software Installation
- **Visual Studio Code** with extensions:
  - Python
  - Azure Account
  - Azure DevOps
  - SQL Tools
- **Python 3.12** from python.org
- **ODBC Driver for SQL Server 18** from Microsoft
- **Azure CLI** from Microsoft
- **Git** for version control

#### Framework Installation
```powershell
# Create virtual environment
python -m venv .env
.\.env\Scripts\Activate.ps1

# Install package manager
pip install uv

# Install Ingenious framework
uv pip install git+https://github.com/Insight-Services-APAC/Insight_Ingenious_For_Fabric.git@v1.0

# Install DBT wrapper
uv pip install git+https://github.com/Insight-Services-APAC/APAC-Capability-DAI-DbtFabricSparkNB.git
```

## Assignment Tasks

### Phase 1: Foundation Setup (Week 1)

#### Task 1.1: Project Structure Creation
Create a repository structure following enterprise patterns:

```
NYC-Traffic-Safety-Platform/
├── DP/                                    # Data Platform workspace
│   ├── deployment/
│   │   ├── DP_Full.yml                   # Azure DevOps pipeline
│   │   └── DP_Initial.yml                # Initial deployment pipeline
│   ├── dbt_project/
│   │   ├── models/
│   │   │   ├── bronze/                   # Raw collision and reference data sources
│   │   │   ├── silver/                   # Cleansed and validated safety data  
│   │   │   └── gold/                     # Traffic safety dimensional models
│   │   ├── macros/                       # Safety data transformation utilities
│   │   ├── tests/                        # Traffic safety data quality validations
│   │   └── dbt_project.yml               # DBT configuration
│   ├── fabric_workspace_items/
│   │   ├── lakehouses/                   # Bronze/Silver/Gold lakehouses
│   │   └── config/                       # DOT/NYPD API endpoints and refresh schedules
│   └── platform_manifest_development.yml # Deployment manifest
├── ER/                                   # Enterprise Reporting workspace
│   ├── deployment/
│   ├── fabric_workspace_items/
│   │   └── semantic_model/               # Vision Zero Analytics semantic models
│   └── platform_manifest_development.yml
└── docs/                                 # Documentation
    ├── README.md
    ├── architecture.md
    ├── deployment.md
    └── collision-data-dictionary.md    # NYC Motor Vehicle Collision schema and business rules
```

**Deliverable**: Git repository with proper folder structure and initial documentation

#### Task 1.2: NYC Motor Vehicle Collision Data Acquisition
Download and prepare multiple related NYC datasets for comprehensive traffic safety analysis:

**Primary Dataset**: 
- **Motor Vehicle Collisions - Crashes**: [NYC Motor Vehicle Collisions](https://data.cityofnewyork.us/d/h9gi-nx95)
  - **Size**: ~2 million collision records (2019-2024)
  - **Format**: CSV files (yearly files recommended)
  - **Key Fields**: crash date/time, location, injuries/fatalities, vehicle types, contributing factors

**Supplementary Datasets** (for dimensional modeling):
- **Motor Vehicle Collisions - Vehicles**: [Vehicle Details](https://data.cityofnewyork.us/d/bm4k-52h4) (Vehicle-level details)
- **Motor Vehicle Collisions - Persons**: [Person Details](https://data.cityofnewyork.us/d/f55k-p6yu) (Casualty details)
- **NYC Street Centerlines**: [Street Network](https://data.cityofnewyork.us/City-Government/Centerline/3mf9-qshr) (Street reference data)
- **Police Precincts**: [NYPD Precincts](https://data.cityofnewyork.us/City-Government/Police-Precincts/y76i-bdw7) (Geographic boundaries)

**Download Strategy**:
1. Start with **2022-2023 collision data** (~300K records) for initial development
2. Download vehicle and person datasets for the same period
3. Use the [NYC Open Data Portal](https://opendata.cityofnewyork.us/) 
4. Download in CSV format and store in `lh_bronze` lakehouse
5. Document data relationships and referential integrity rules
6. Expand to historical data once pipeline is working

**Why This Dataset Collection is Perfect for Star Schema**:
- ✅ **Natural Fact Tables**: Collisions (main), Vehicles (related), Persons (related)
- ✅ **Rich Dimensions**: Date/Time, Location, Vehicle Type, Injury Severity, Contributing Factors
- ✅ **Real Business Value**: Supports actual Vision Zero initiative with measurable KPIs
- ✅ **Complex Relationships**: Many-to-many relationships requiring bridge tables
- ✅ **Data Quality Challenges**: Missing geocoding, inconsistent vehicle types, data entry errors
- ✅ **Multiple Analysis Perspectives**: Temporal trends, geographic hotspots, causality analysis

**Deliverable**: Complete collision dataset collection with documented relationships and data lineage

#### Task 1.3: Fabric Workspace Setup
1. Create lakehouses: `lh_bronze`, `lh_silver`, `lh_gold`, `lh_logs`
2. Upload collision datasets in a Storage Account
3. Set up workspace permissions and security
4. Configure variable libraries for environment settings
5. Create data documentation with collision data schema and Vision Zero KPIs

**Deliverable**: Functional Fabric workspace with collision data properly ingested and cataloged

### Phase 2: Data Platform Development (Week 2)

#### Task 2.1: Bronze Layer Implementation
Create shortcuts or direct connections to your collision datasets:
- Configure bronze layer as raw data access point for all collision-related data using Fabric Shortcut
- Connect Fabric workspace to a temp repository and download the lakehouse folder
- Copy the lakehouse including shortcuts under fabric_workspace_items/lakehouses/

**DBT Sources Configuration**: You can use generateIngineous Fabric to download and generate schema
```yaml
# models/bronze/sources.yml
version: 2
sources:
  - name: bronze
    description: NYC Motor Vehicle Collisions - Raw datasets
    tables:
      - name: mv_collisions_crashes
        description: Motor vehicle collision incidents
        columns:
          - name: collision_id
            description: Unique collision identifier
            tests:
              - unique
              - not_null
          - name: crash_date
            description: Date of collision
            tests:
              - not_null
          - name: crash_time
            description: Time of collision
      - name: mv_collisions_vehicles
        description: Vehicle details for each collision
        columns:
          - name: collision_id
            description: Links to crashes table
            tests:
              - not_null
      - name: mv_collisions_persons
        description: Person/casualty details for each collision
      - name: police_precincts
        description: NYPD precinct boundaries
      - name: street_centerlines
        description: NYC street network reference
```

**Deliverable**: Bronze layer with proper data access and cataloging

#### Task 2.2: Silver Layer Transformations
Develop DBT models for data cleansing and standardization:

**Example Silver Model** (`models/silver/CollisionsClean.sql`):
```sql
{{ config(
    materialized='table',
    file_format='delta'
) }}

select 
    collision_id,
    cast(crash_date as date) as collision_date,
    cast(crash_time as time) as collision_time,
    borough,
    zip_code,
    latitude,
    longitude,
    on_street_name,
    cross_street_name,
    number_of_persons_injured,
    number_of_persons_killed,
    number_of_pedestrians_injured,
    number_of_pedestrians_killed,
    number_of_cyclist_injured,
    number_of_cyclist_killed,
    number_of_motorist_injured,
    number_of_motorist_killed,
    contributing_factor_vehicle_1,
    contributing_factor_vehicle_2,
    vehicle_type_code1,
    vehicle_type_code2,
    -- Data quality improvements
    case 
        when borough is null and zip_code is not null then 
            case 
                when zip_code between 10001 and 10282 then 'MANHATTAN'
                when zip_code between 10301 and 10314 then 'STATEN ISLAND'
                when zip_code between 10451 and 10475 then 'BRONX'
                when zip_code between 11201 and 11256 then 'BROOKLYN'
                when zip_code between 11001 and 11697 then 'QUEENS'
                else 'UNKNOWN'
            end
        else upper(borough)
    end as borough_clean,
    -- Severity classification
    case 
        when number_of_persons_killed > 0 then 'Fatal'
        when number_of_persons_injured > 0 then 'Injury'
        else 'Property Damage Only'
    end as collision_severity,
    -- Time of day classification
    case 
        when cast(crash_time as time) between '06:00:00' and '09:59:59' then 'Morning Rush'
        when cast(crash_time as time) between '10:00:00' and '15:59:59' then 'Midday'
        when cast(crash_time as time) between '16:00:00' and '19:59:59' then 'Evening Rush'
        when cast(crash_time as time) between '20:00:00' and '23:59:59' then 'Evening'
        else 'Late Night/Early Morning'
    end as time_period
from {{ source('bronze', 'mv_collisions_crashes') }}
where collision_id is not null
  and crash_date is not null
  and crash_date <= current_date()
```

**Requirements**:
- Implement silver layer models
- Add data quality validations and cleansing rules
- Include business logic and standardization
- Add comprehensive testing

**Deliverable**: Silver layer with cleansed, validated data models

#### Task 2.3: Gold Layer Dimensional Modeling
Create star schema dimensional models:

**Dimension Tables**:
- `DimCalendar`: Business calendar with Vision Zero campaign periods and holidays
- `DimLocation`: NYC boroughs, precincts, zip codes, and street network hierarchy
- `DimVehicleType`: Vehicle classifications and categories (car, truck, bike, etc.)
- `DimContributingFactor`: Collision causes and driver behavior classifications
- `DimSeverity`: Injury severity levels and casualty type classifications
- `DimTime`: Hour-level time dimension with rush hour and traffic pattern analysis

**Fact Tables**:
- `FactCollisions`: Individual collision incidents with location and severity details
- `FactCasualties`: Person-level injury and fatality records linked to collisions
- `FactVehicleInvolvement`: Vehicle-level details for multi-vehicle collisions
- `FactSafetyMetrics`: Aggregated safety statistics by location and time period

**Example Gold Model** (`models/gold/FactCollisions.sql`):
```sql
{{ config(
    materialized='table',
    file_format='delta'
) }}

select 
    collision_id,
    collision_date,
    collision_time,
    borough_clean as borough,
    zip_code,
    latitude,
    longitude,
    collision_severity,
    time_period,
    -- Casualty metrics
    number_of_persons_injured,
    number_of_persons_killed,
    number_of_pedestrians_injured + number_of_cyclist_injured as vulnerable_road_users_injured,
    number_of_pedestrians_killed + number_of_cyclist_killed as vulnerable_road_users_killed,
    -- Vision Zero KPIs
    case when number_of_persons_killed > 0 then 1 else 0 end as fatal_collision_flag,
    case when vulnerable_road_users_injured + vulnerable_road_users_killed > 0 then 1 else 0 end as vulnerable_user_involved_flag,
    -- Contributing factors (simplified)
    case 
        when contributing_factor_vehicle_1 like '%Alcohol%' or contributing_factor_vehicle_2 like '%Alcohol%' then 'Impaired Driving'
        when contributing_factor_vehicle_1 like '%Speed%' or contributing_factor_vehicle_2 like '%Speed%' then 'Speeding'
        when contributing_factor_vehicle_1 like '%Distracted%' or contributing_factor_vehicle_2 like '%Distracted%' then 'Distracted Driving'
        when contributing_factor_vehicle_1 like '%Traffic Control%' or contributing_factor_vehicle_2 like '%Traffic Control%' then 'Traffic Control Violation'
        else 'Other/Unknown'
    end as primary_contributing_factor,
    -- Date/time dimensions for joining
    year(collision_date) as collision_year,
    month(collision_date) as collision_month,
    dayofweek(collision_date) as day_of_week,
    hour(collision_time) as collision_hour,
    case 
        when dayofweek(collision_date) in (1,7) then 'Weekend'
        else 'Weekday'
    end as day_type
from {{ ref('CollisionsClean') }}
where collision_severity is not null
```

**Deliverable**: Complete dimensional model with facts and dimensions

### Phase 3: Automation & Quality (Week 3)

#### Task 3.1: CI/CD Pipeline Implementation
Create Azure DevOps pipeline for automated deployment:

**Pipeline Features**:
- Multi-environment deployment (DEV → UAT → PROD)
- Automated testing and validation
- Environment-specific configuration
- Rollback capabilities

**Example Pipeline** (`DP/deployment/DP_Full.yml`):
```yaml
trigger:
  branches:
    include:
      - main
      - develop

stages:
- stage: 'DEV'
  displayName: 'Development Deployment'
  variables:
  - group: Dev-Variables
  jobs:
  - deployment: DEV_Deployment
    environment: 'DEV'
    strategy:
      runOnce:
        deploy:
          steps:
          - checkout: self
          - task: UsePythonVersion@0
            inputs:
              versionSpec: '3.12'
          - script: |
              pip install uv
              uv pip install git+https://github.com/Insight-Services-APAC/Insight_Ingenious_For_Fabric.git@v1.0
            displayName: 'Install Ingenious'
          - script: |
              ingen_fab deploy deploy
            displayName: 'Deploy Artifacts'
            env:
              FABRIC_ENVIRONMENT: $(FABRIC_ENVIRONMENT)
              FABRIC_WORKSPACE_REPO_DIR: $(FABRIC_WORKSPACE_REPO_DIR)
```

**Deliverable**: Working CI/CD pipeline with multi-environment support

#### Task 3.2: Business Intelligence Layer
Create semantic models and reports:

**Semantic Model Requirements**:
- Pre-defined relationships between facts and dimensions
- Calculated measures and KPIs
- Optimized for self-service analytics

**Dashboard Requirements**:
- **Vision Zero Executive Dashboard**: High-level safety KPIs for DOT and NYPD leadership
- **Traffic Safety Analytics Dashboard**: Collision trends, hotspot analysis, and contributing factors

**Deliverable**: Complete BI solution with reports and dashboards



## Resources & Support

### Documentation
- [Microsoft Fabric Documentation](https://docs.microsoft.com/en-us/fabric/)
- [DBT Wrapper Documentation](https://blog.insight-services-apac.dev/APAC-Capability-DAI-DbtFabricSparkNb/)
- [Ingineous Fabric Documentation](https://blog.insight-services-apac.dev/Insight_Ingenious_For_Fabric/)



### Submission Format
1. **Git Repository**: Share repository access with instructors
2. **Documentation**: README with setup and usage instructions
3. **Demo Environment**: Working Fabric workspace for testing
4. **Presentation**: PowerPoint or equivalent with architecture overview
5. **Reflection Report**: 2-page summary of challenges and learnings

### Evaluation Process
- **Peer Review**: Code review by other participants
- **Instructor Review**: Technical assessment and feedback
- **Demo Session**: Live demonstration and Q&A
- **Final Scoring**: Based on rubric and deliverable quality

---

**Note**: This assignment simulates real-world data engineering challenges. Focus on building production-ready solutions with proper documentation, testing, and deployment practices. Good luck!

## Getting Started

1. **Set up your development tenant** using the Microsoft 365 Developer Program
2. **Clone the starter repository** (will be provided)
3. **Join the assignment Slack channel** for questions and collaboration
4. **Review the sample data** and understand the business requirements
5. **Start with Phase 1 tasks** and work systematically through each phase

