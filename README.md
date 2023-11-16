`# Overview
InsuranceLake was built to process batch files by mapping source to target columns, transforming each column, and applying data quality rules. The most common type of batch file data sources are large delimited text files, Excel files, and fixed width files. InsuranceLake can be enhanced to accept change data capture, streaming, and document data sources. It is based on the Olympic Data Lake Pattern (bronze-silver-gold) which we call Collect, Cleanse, and Consume your data. Each incoming data source (e.g. a specific csv file with commercial auto policies from broker abc) is intended to have a mapping, transform, data quality, and if desired, an entity match instruction file to be paired with it. These instruction files are no mandatory and InsuranceLake will create default ones if none are provided. The incoming data files are placed in the Collect layer, a workflow is then triggered to run the mapping, transform, data quality, and entity match processes and the results are stored in the Cleanse layer. Any data quality rules marked as quarantine will kick bad data out to a quarantine layer. Finally a set of spark sql and athena sql files can be run to populate the Consume layer.

The fun begins when you start to use and analyze your data. For starters check this out:
- [Gaining insights into Combined Ratio and Claims](https://youtu.be/qEbxU9Q6uVc)
- [Measuring agents book of business](https://youtu.be/Qx-iKbyYNDI)
- [QuickSight DemoCentral try out the dashboards and click the icon on the left with the pencil to go into developer view](https://democentral.learnquicksight.online/#)
- [General Insurance Dashboard](https://democentral.learnquicksight.online/#Dashboard-DashboardDemo-General-Insurance)
- [Life Insurance Dashboard](https://democentral.learnquicksight.online/#Dashboard-DashboardDemo-Life-Insurance)

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
- IL: InsuranceLake
- 3C's: Collect, Cleanse, and Consume Data
- DIE: [D]ata files, [I]nstruction files, and generic [E]ngines that use the instruction files to process the data files

---------------------------
Architectural Principles
A scalable ingestion framework is build on 3 pillars: data (the subject); metadata (the instructions); and code (the engine).

FLIPS:
- Foundation: Automated AWS account provisioning and monitoring
- Less is More: keep it Simple, Code is BOTH an Asset and a Liability
- Iterative Improvements: Build in weeks not months with 2 Pizza Teams that use DevSecOps
- Patterns: 3C's and DIE
- Serverless + Automation: Avoid the undifferentiated heavy lifting, everything as code with the AWS CDK

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
## Mapping

---------------------------
## Transforms

---------------------------
## Data Quality Rules


# Consume Data
---------------------------
## Spark SQL

---------------------------
## Athena SQL


# Monitor Data Quality


# MonitorData Lineage


# Using Transforms
The order that you enter the transforms into the json file is very important . Each transform is executed on the incoming dataset starting from the beginning of the transform_spec section of the file.

---------------------------
## Formatting

- currency : Convert specified numeric field with currnecy formatting to Decimal (fixed precision)
```
  "currency": [
        {
          "field": "SmallDollars",
          "format": "6,2"
        },
        {
          "field": "EuroValue",
          "euro": true
        }
        ]
```

- date : Convert specified date fields to ISO format based on known input format
```
  "date": [
        {
          "field": "StartDate",
          "format": "M/d/yy"
        },
        {
          "field": "EndDate",
          "format": "yy-MM-dd"
        },
        {
          "field": "valuationdate",
          "format": "yyyyMMdd"
        }
        ]
```
- decimal : Convert specified numeric field (usually Float or Double) fields to Decimal (fixed precision) typ "decimal"
```
  "decimal": [
        {
          "field": "ExpiringPremiumAmount",
          "format": "10,2"
        },
        {
          "field": "WrittenPremiumAmount",
          "format": "10,2"
        },
        {
          "field": "EarnedPremium",
          "format": "10,2"
        }
        ]
```
- implieddecimal : Convert specified numeric field (usually Float or Double) fields to Decimal (fixed precision) type with implied decimal point support (i.e. last 2 digits are to the right of decimal)
```
  "implieddecimal": [
        {
          "field": "indemnity_paid_current_period",
          "format": "16,2"
        }
        ]
```
- timestamp	Convert specified date/time fields to ISO format based on known input format
```
"timestamp": [
        {
        "field": "GenerationDate",
        "format": "yyyy-MM-dd HH:mm:ss.SSS+0000"
        }
        ]
```
---------------------------
## Data Manipulation

- addcolumns : Add two or more columns together in a new column
```
  "addcolumns": [
        {
          "field": "TotalWrittenPremium",
          "source_columns": [ "WrittenPremiumAmount" ]
        }
        ]
```
- columnfromcolumn : Add column to DataFrame based on regexp pattern on another column
```
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
```
- combinecolumns : Add column to DataFrame using format string and source columns
```
  "combinecolumns": [
        {
          "field": "RowKey",
          "format": "{}-{}-{}",
          "source_columns": [ "LOBCode", "PolicyNumber", "StartDate" ]
        }
        ]
```
- filename : Add column to DataFrame based on regexp pattern on the filename argument to the Glue job
```
  "filename": [
        {
          "field": "valuationdate",
          "pattern": "\\S+-(\\d{8})\\.csv",
          "required": true
        },
        {
          "field": "program",
          "pattern": "([A-Za-z0-9]+)\\S+\\.csv",
          "required": true
        }
        ]
```
- filterrows : Filter out rows based on standard SQL WHERE statement
```
  "filterrows": [
        {
          "condition": "claim_number is not null or file_number is not null"
        },
        {
          "condition": "`startdate` >= cast('1970-01-01' as date)"
        }
        ]
```        
- flipsign : Flip the sign of a numeric column in a Spark DataFrame, optionally in a new column 
```
  "flipsign": [
        {
          "field": "Balance"
        },
        {
          "field": "NewAccountBalance",
          "source": "AccountBalance"
        }
        ]
```
- literal : Add column to DataFrame with static/literal value supplied in specification 
```
  "literal": {
          "source": "syntheticdata"
        }
```
- lookup : Replace specified column values with values looked up from an dynamodb table
```
  "lookup": [
        {
        "field": "smokingclass",
        "lookup": "smokingclass"
        },
        {
        "field": "issuestatename",
        "source": "issuestate",
        "lookup": "StateCd",
        "nomatch": "N/A"
        }
        ]
```
Example of a script that populates a Dynamodb table:
```
./load_dynamodb_multilookup_table.py dev-insurancelake-etl-multi-lookup lookups.csv PolicyData-LOBCoverage originalprogram originalcoverage
```

- multivaluelookup : Add columns looked up from an external table using multiple conditions, returning any number of attributes
To setup a multilookup transform, begin by preparing the lookup data. The data should be saved as CSV and include all the match columns and return value columns. It is ok to have some columns that are not used, because the transform specification allows the user to select the specific return columns.

Use the included loading script in the resources directory to import the CSV data into etl-multi-lookup DynamoDB table:

./load_dynamodb_multilookup_table.py dev-insurancelake-etl-multi-lookup lookups.csv PolicyData-LOBCoverage originalprogram originalcoverage

Usage: load_dynamodb_multilookup_table.py [-h] table_name data_file lookup_group lookup_columns [lookup_columns ...]
The following arguments are required: table_name, data_file, lookup_group, lookup_columns
- table_name indicates the name of the DynamoDB table deployed by the InsuranceLake CDK stack for multi-lookups, in the form <environment>-<resource prefix>-etl-multi-lookup. All multilookup lookup datasets are stored in the same table and grouped by lookup_group.
-  lookup_group can be any name that is meaninginful to the user and will be specified in the transform spec.
- lookup_columns are listed as parameters last, separated by spaces. At least one lookup column is required.

Use the AWS Console for the DynamoDB service to confirm that the data is loaded correctly. Note that the lookup columns will be concatenated with a hyphen (-) separator and stored as a single string in the sort key. All return columns will be stored as separate attributes. This is important to understand when editing the data in the future.

Now insert the multilookup specification into your dataset’s transformation spec file (in the transform_spec section). An example follows:


```
  "multivaluelookup": [
        {
        "lookup_group": "LOBCoverage",
        "match_columns": [
          "program",
          "coverage"
        ],
        "return_attributes": [
          "coveragenormalized",
          "lob"
        ],
        "nomatch": "N/A"
        }
        ]
```

- The lookup_group string should match the lookup_group string you used to load the data in DynamoDB.
- match_columns indicates the columns in your incoming data set that must have matching values in the lookup data. Note that the column values only refer to the incoming dataset. The column names in your lookup data (in DynamoDB) do not matter, because all the lookup column values are stored in a concatenated string in the lookup_item sort key.
- return_attributes specifies the attribute names in the DynamoDB lookup table to add to the incoming dataset.

Important Note: if a column already exists, a duplicate column will be created, which will raise an error when saving to Parquet format. Take care to map your incoming dataset correctly so that it has unique column names after performing the multilookup transform. For example. suppose your incoming data has a lineofbusiness column, but it is composed of bespoke values that you want to normalize. Best practice would be to map lineofbusiness to the name originallineofbusiness so the incoming data is preserved, and use the multilookup to return a new (normalized) lineofbusiness attribute value.

- merge : Merge columns using coalesce 
```
  "merge": [
        {
          "field": "insuredstatemerge",
          "source_list": [
            "insuredstatename", "insuredstatecode"
        ],
          "default": "Unknown"
        }
        ]
```
---------------------------
## Data Security

- redact : Redact specified column values using supplied redaction string
```
  "redact": {
        "CustomerNo": "****"
        }
```
- hash : Hash specified column values using SHA256 and Spark UDF
```
  "hash": [
          "InsuredContactCellPhone",
          "InsuredContactEmail"
        ]
```  
- tokenize : Replace specified column values with hash and store original value in separate table
```
  "tokenize": [
        "EIN"
        ]
```
---------------------------
## Earned Premium

- enddate : Add a number of months to a specified date to get an ending/expiration date
```
  "enddate": [
        {
        "field": "CalcExpirationDate",
        "start_date": "EffectiveDate",
        "num_months": "Term"
        }
        ]
```
- expandpolicymonths : Expand dataset to one row for each month the policy is active with a calculated earned premium
```
  "expandpolicymonths": {
        "uniqueid": "generated_policy_number",
        "policy_month_start_field": "StartDate",
        "policy_month_end_field": "EndDate",
        "policy_effective_date": "EffectiveDate",
        "policy_expiration_date": "ExpirationDate"
        }
```
- policymonths : Calculate number of months between policy start/end dates
```
  "policymonths": [
        {
          "field": "CalcNumMonths",
          "written_premium_list": [
            "WrittenPremiumAmount"
        ],
          "policy_effective_date": "EffectiveDate",
          "policy_expiration_date": "ExpirationDate",
          "normalized": true
        }
        ]
```
- earnedpremium : Calculate monthly earned premium
```
  "earnedpremium": [
        {
        "field": "CalcEarnedPremium",
        "written_premium_list": [
          "WrittenPremiumAmount"
        ],
        "policy_effective_date": "EffectiveDate",
        "policy_expiration_date": "ExpirationDate",
        "period_start_date": "StartDate",
        "period_end_date": "EndDate",
        "byday": true
        }
        ]
```
# Entity Match


# Operation
Partitions
Reload Data
Schema Changes

# DevOps

