# Overview
InsuranceLake was built to process batch files by mapping source to target columns, transforming each column, and applying data quality rules. The most common type of batch file data sources are large delimited text files, Excel files, and fixed width files. InsuranceLake can be enhanced to accept change data capture, streaming, and document data sources. It is based on the Olympic Data Lake Pattern (bronze-silver-gold) which we call Collect, Cleanse, and Consume your data. Each incoming data source (e.g. a specific csv file with commercial auto policies from broker abc) is intended to have a mapping, transform, data quality, and if desired, an entity match instruction file to be paired with it. These instruction files are no mandatory and InsuranceLake will create default ones if none are provided. The incoming data files are placed in the Collect layer, a workflow is then triggered to run the mapping, transform, data quality, and entity match processes and the results are stored in the Cleanse layer. Any data quality rules marked as quarantine will kick bad data out to a quarantine layer. Finally a set of spark sql and athena sql files can be run to populate the Consume layer.

The fun begins when you start to use and analyze your data. For starters check this out:
[General Insurance Dashboard]()
[Life Insurance Dashboard]()
[QuickSight DemoCentral try out the dashboards and click the icon on the left with the pencil to go into developer view](https://democentral.learnquicksight.online/#)

---------------------------
YouTube Videos
- [8 Minute Overview](https://youtu.be/UEVrSGrH3JA)
- [Part 1 - CodePipeline, Verisk Data, Build a Transform & Unit Test with Code Whisperer](https://youtu.be/2c4hopaboVs)
- [Part 2 - Run Collect to Cleanse](https://youtu.be/l2r2lBW2zXw)
- [Part 3 - Run Cleanse to Consume & View Verisk Data](https://youtu.be/MayAPuUMvcg)
- [Data Quality](https://youtu.be/dUhzjX0eabM)
- [Data Lineage](https://youtu.be/E8pjUZK62BI)

---------------------------
Acronyms
IL: InsuranceLake
3C's: Collect, Cleanse, and Consume Data
DIE: [D]ata files, [I]nstruction files, and generic [E]ngines that use the instruction files to process the data files

---------------------------
Architectural Principles
A scalable ingestion framework is build on 3 pillars: data (the subject); metadata (the instructions); and code (the engine).


AWS Services:
1. S3: stores the incoming data files as well as the Apache Parquet Files
2. Lambda
3. Step Functions: executes the Collect to Cleanse, the Cleanse to Consume, and the Entity Match Glue Jobs
4. Glue
5. DynamoDB
6. Athena
7. QuickSight
8. KMS



# Collect Data

---------------------------
CSV Files


---------------------------
Fixed Width Files


---------------------------
Excel Files


# Cleanse Data
---------------------------
**Mapping
---------------------------
**Transforms
---------------------------
**Data Quality Rules

# Consume Data
---------------------------
**Spark SQL
---------------------------
**Athena SQL

# Monitor Data Quality
# MonitorData Lineage

# Using Transforms
discuss how order and reuse in json file is important

---------------------------
Formatting

- currency : Convert specified numeric field with currnecy formatting to Decimal (fixed precision)


- date : Convert specified date fields to ISO format based on known input format

- decimal : Convert specified numeric field (usually Float or Double) fields to Decimal (fixed precision) type
  "decimal": [{"field": "ExpiringPremiumAmount","format": "10,2"},{"field": "WrittenPremiumAmount","format": "10,2"},{"field": "EarnedPremium","format": "10,2"}]

- implieddecimal : Convert specified numeric field (usually Float or Double) fields to Decimal (fixed precision) type with implied decimal point support (i.e. last 2 digits are to the right of decimal)
  "implieddecimal": [{"field": "indemnity_paid_current_period","format": "16,2"}]

- timestamp	Convert specified date/time fields to ISO format based on known input format
  "timestamp":[{"field": "GenerationDate","format": "yyyy-MM-dd HH:mm:ss.SSS+0000"}]

---------------------------
Data Manipulation

- addcolumns : Add two or more columns together in a new column
  "addcolumns": [
        {
        "field": "TotalWrittenPremium","source_columns": ["WrittenPremiumAmount"]}]

- columnfromcolumn : Add column to DataFrame based on regexp pattern on another column
  "columnfromcolumn": [
        {
        "field": "username",
        "source": "emailaddress",
        "pattern": "(\\S+)@\\S+"
        },
        {
        "field": "policyyear",
        "source": "policyeffectivedate",
        "pattern": "(\\d\\d\\d\\d)/\\d\\d/\\d\\d"
        }
        ]

- combinecolumns : Add column to DataFrame using format string and source columns


- filename : Add column to DataFrame based on regexp pattern on the filename argument to the Glue job


- filterrows : Filter out rows based on standard SQL WHERE statement


- flipsign : Flip the sign of a numeric column in a Spark DataFrame, optionally in a new column 
  "flipsign": [{"field": "Balance"},{"field": "NewAccountBalance","source": "AccountBalance"}]

- literal : Add column to DataFrame with static/literal value supplied in specification 
  "literal": {"source": "syntheticdata"}

- lookup : Replace specified column values with values looked up from an external table
  

- multilookup : Add columns looked up from an external table using multiple conditions, returning any number of attributes


- merge : Merge columns using coalesce 
  "merge": [{"field": "insuredstatemerge","source_list": ["insuredstatename", "insuredstatecode"],"default": "Unknown"}]

---------------------------
Data Security

- redact : Redact specified column values using supplied redaction string
  "redact": {"CustomerNo": "****"}

- hash : Hash specified column values using SHA256 and Spark UDF
  "hash": ["InsuredContactCellPhone","InsuredContactEmail"]

- tokenize : Replace specified column values with hash and store original value in separate table
  "tokenize": ["EIN"]

---------------------------
Earned Premium

- enddate : Add a number of months to a specified date to get an ending/expiration date


- expandpolicymonths : Expand dataset to one row for each month the policy is active with a calculated earned premium
  "expandpolicymonths": {"uniqueid": "generated_policy_number","policy_month_start_field": "StartDate","policy_month_end_field": "EndDate","policy_effective_date": "EffectiveDate","policy_expiration_date": "ExpirationDate"}

- policymonths : Calculate number of months between policy start/end dates
  "policymonths": [{"field": "CalcNumMonths","written_premium_list": ["WrittenPremiumAmount"],"policy_effective_date": "EffectiveDate","policy_expiration_date":"ExpirationDate","normalized": true}]

- earnedpremium : Calculate monthly earned premium
  "earnedpremium": [{"field": "CalcEarnedPremium","written_premium_list": ["WrittenPremiumAmount"],"policy_effective_date": "EffectiveDate","policy_expiration_date": "ExpirationDate","period_start_date": "StartDate","period_end_date": "EndDate","byday": true}]

# Entity Match


# Operation
Partitions
Reload Data
Schema Changes

# DevOps



* code (the execution engine).Metadata contained in csv and json files holds the information that describes how to process each incoming data file;
    Figure 1 shows an example blueprint of an ingestion framework. In this example validated and, if required, transformed data is ingested in either SQLDB, NoSQL DB (Graph), both or none. The sample framework is able to apply three types of validations;
    snapshot validation (ensure the date of the received file is the latest date);
    rowcount anomaly (sanity check on the amount of records in the file);
    datatype validation (ensure the datatype of a column is the expected datatype).
    In reality you could have many more validation or quality checks that you want to apply. However, it’s not a given that all checks apply to all sources. Some sources might be eligible for more (or less) validation checks, depending on file-specific features like sensitivity, governance etc.
* Mapping CSV file -
* Transform JSON file -
* Data Quality JSON file -
* Athena SQL Files - 
* Spark SQL Files - 



# Installation
Install the basics in 30 minutes
To set the region that InsuranceLake is installed in see the lib/configuration.py file.
https://gitlab.aws.dev/fsi-sat/aws-cdk-insurancelake-etl/-/blob/main/README.md
https://github.com/aws-samples/aws-insurancelake-etl/blob/main/README.md

can this be done both stand alone AND after the QuickStart is done?
install the basics + DevOps tools
Setup a new repository and deploy using CI/CD

Pull from an existing repository and deploy using CI/CD (will be the de-facto install once published in Github)

Install with 3 environments

Make a change and deploy automatically with self-mutating CodePipeline

InsuranceLake can be deployed with no VPC simply by removing the subnet definition in configuration.py. The VPC is only used if the customer needs it.

The public subnet is completely optional as well. InsuranceLake does not require any VPC, so it also does not require public subnets. Creating a VPC with half public subnets and half private is the default behavior. You can modify this by passing the subnet_configuration parameter to the VPC creation in lib/vpc_stack.py.

If the VPC is enabled in InsuranceLake, Glue is really the only service that will use it, and specifically, for Glue connections. If you try this out, you’ll see that the Glue connections specifically select the private subnet from the InsuranceLake-created VPC, through the vpc.subnets method.

1. Add Permission boundaries to all the roles that CDK creates example add Permission boundary name CloudCoreL3PermissionBoundary to all the roles 
2. Add the mandatory tags to all resources without which SCP will deny any resource creation.
