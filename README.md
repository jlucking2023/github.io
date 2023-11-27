# InsuranceLake Documentation
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
- [1 Quickly gain insights with your data using AWS InsuranceLake](https://youtu.be/V9vP92kJnDk)
- [2 Use Glue Data Quality with AWS InsuranceLake to define and enforce data rules](https://youtu.be/WDd8JunNpEQ)
- [3 Use data lineage tracking and observability in your data pipeline to build trust with AWS InsuranceLake](https://youtu.be/mbG3Opq7P3c)
- [4 Add custom data transformations using AWS InsuranceLake to quickly ingest 1st and 3rd party data](https://youtu.be/IDHmsFuZT2o)


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
5. DynamoDB - used for lookup & multilookup transforms as well as stores logs
6. Athena
7. QuickSight
8. KMS


# Installation
To set the region that InsuranceLake is installed in see the lib/configuration.py file.
- [30 minute QuickStart](https://github.com/aws-samples/aws-insurancelake-etl/blob/main/README.md#quickstart) 
- [60 minute QuickStart with CI/CD](https://github.com/aws-samples/aws-insurancelake-etl/blob/main/README.md#quickstart-with-cicd)

---------------------------
Make a change an deploy automatically with self-mutating CodePipeline

---------------------------
No VPC's needed
- InsuranceLake can be deployed with no VPC(s) simply by removing the subnet definition in configuration.py.
- The public subnet is completely optional as well. InsuranceLake does not require any VPC, so it also does not require public subnets. Creating a VPC with half public subnets and half private is the default behavior. You can modify this by passing the subnet_configuration parameter to the VPC creation in lib/vpc_stack.py.
- If the VPC is enabled in InsuranceLake, Glue is really the only service that will use it, and specifically, for Glue connections. If you try this out, you’ll see that the Glue connections specifically select the private subnet from the InsuranceLake-created VPC, through the vpc.subnets method.


# Collect Data
dev-insurancelake-<account ID>-us-east-2-collect > <database name> > <table name>

---------------------------
CSV Files
Add the following code block to your transform JSON file. This should be at the very top of the file.
```
```

---------------------------
Fixed Width Files
Add the following code block to your transform JSON file. This should be at the very top of the file.
```
{
  "input_spec": {     
    "fixed": {
    
    }
  },
  "transform_spec": {      
  }
}
```
Note on Sedgewick e02 fixed width data files: to handle zero-padded data fields the columnreplace transform was created to convert them into null values

Sedgwick e02 Claim Interface File Option 1: manually split files
Fixed Length files need a mapping file with these 3 columns: SourceName,DestName,Width.
Width is the length of the field.

How to handle fixed length files with multiple record types where each type has a different layout?
See this file e02-Claims.csv for an example of a mapping file for record type ''.
```
SourceName,DestName,Width
record_type,record_type,3
client_id,client_id,4
client_account,client_account,8
client_location,client_location,6
claim_number,claim_number,18
etc...
```

See this file e02-Claims.json for an example of a transform spec file.
note: date and implieddecimal fields removed for brevity.
```
{
	"input_spec": {

		"fixed": {}
	},

	"transform_spec": {

		"combinecolumns": [
			{
				"field": "datetime_of_loss",
				"format": "{} {}",
				"source_columns": [ "date_of_loss", "time_of_loss" ]
			}
		],
	
		"date": [
			{
				"field": "date_of_loss",
				"format": "yyyyMMdd"
			},
			{
				"field": "length_of_service",
				"format": "yyMMdd"
			}
		],

		"timestamp": [
			{
				"field": "datetime_of_loss",
				"format": "yyyyMMdd HHmm"
			}
		],

		"redact": {
			"claimant_last_name": "****",
			"claimant_ssn": "****",
			"claimant_age": "****",
			"date_of_birth": "****",
			"union_id": "****",
			"driver_name": "****",
			"driver_age": "****",
			"claimant_telephone_number": "****"
		},

		"implieddecimal": [
			{
				"field": "avg_weekly_wage",
				"format": "16,2"
			},
			{
				"field": "net_incurred",
				"format": "16,2"
			}
		]
	}
}
```

Here's how to perform a manual separation of record types:
```
grep ^<record type code> e02file > e02file-filtered
grep ^CLM e02file > e02file-claims
```

Sedgwick e02 Claim Interface File Option 2: use lambda split file function

---------------------------
Excel Files
Add the following code block to your transform JSON file. This should be at the very top of the file.
```
"input_spec": {
    "excel": {
        "sheet_names": [
            "Sheet1"
        ],
        "data_address": "A1",
        "header": true,
        "password": ""
    }
},
```


---------------------------
Parquet Files
Parquet file in the Collect bucket support (Parquet files should be uploaded by themselves, i.e. without any partition folder structure)


# Cleanse Data
dev-insurancelake-<account ID>-us-east-2-cleanse > <database name> > <table name>

---------------------------
## Mapping
Commas are not allowed in any column as they are used to indicate the next column.
- SourceName : required, must be the first column and match the column names in the 1 row of the data file, if there are no column names then col_1, col_2, col_(n+1) should be used.
- DestName : required, must be the 2nd column and contain all lower case characters.
- Description : optional column that can contain any character.
- Width : required for Fixed Width Files, must be the third column and contain only a positve integer.
- Threshold : ?
- Score : ?


---------------------------
## Transforms

---------------------------
## Data Quality Rules

quarantine
Build and test insurance data quality rules

* warn
* fail

Data Quality rule descriptions
Data Quality warning alerts

Note on Transforms & Data Quality
here might be situation where day and month is reversed and if for some dates the day is <=12 it will still transform them considering them as day while they are actually month. Similarly, as we have seen in the case of wrong formats where we expressed MM as mm it used its interpretation of facts.


# Consume Data
dev-insurancelake-<account ID>-us-east-2-consume > <database name> > <table name>

---------------------------
## Spark SQL
spark_<name>.sql - 1 statement in file is allowed







---------------------------
## Athena SQL
athena_<name>.sql - multiple statements in file is allowed

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
...handles currency fields with no decimal point and implied decimal values as the last two characters, which is what the Sedgwick file uses.
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

- columnreplace :
```
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
- Future: use / enhance existing DynamoDB load python code (located here?)
- Future: add a copy between Dev-Test-Prod in DevSecOps Process
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

- multivaluelookup : Add columns looked up from an external table using multiple conditions, returning any number of attributes
To setup a multilookup transform, begin by preparing the lookup data. The data should be saved as CSV and include all the match columns and return value columns. It is ok to have some columns that are not used, because the transform specification allows the user to select the specific return columns.
- Future: use / enhance existing DynamoDB load python code
```
./load_dynamodb_multilookup_table.py dev-insurancelake-etl-multi-lookup lookups.csv PolicyData-LOBCoverage originalprogram originalcoverage
```
- Future: add a copy between Dev-Test-Prod in DevSecOps Process

Use the included loading script in the resources directory to import the CSV data into etl-multi-lookup DynamoDB table:



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
---------------------------
Partitions



---------------------------
Purge and Reload Data
Replace a previously loaded dataset (same day and different day)
instructions on how to handle re-loading the same data file (perhaps Feb's Broker A data had an error and you want to reload it)


---------------------------
Schema Changes
IL allows for three types of methods for handling schema changes:
Permissive - <describe briefly here>
Strict - <describe briefly here>
Reorder - <describe briefly here>


---------------------------
Duplicate Column Names
Spark can handle duplicate column names by appending a number representing the index of the column.

# DevOps

---------------------------
dev-insurancelake-cleanse-to-consume-job

Python library path
s3://dev-insurancelake-<AWS Account Number>-us-east-2-etl-scripts/etl/lib/
01. custom_mapping.py,
02. datalineage.py,
03. dataquality_check.py,
04. datatransform_dataprotection.py,
05. datatransform_lookup.py,
06. datatransform_premium.py,
07. datatransform_premiumdemo.py,
08. datatransform_regex.py,
09. datatransform_typeconversion.py,
10. glue_catalog_helpers.py

Dependent JARs path
s3://dev-insurancelake-<AWS Account Number>-us-east-2-etl-scripts/etl/lib/
01. openlineage-spark-0.29.2.jar,
02. poi-ooxml-5.2.3.jar,
03. spark-excel_2.12-3.3.1_0.18.7.jar,
04. xmlbeans-5.1.1.jar


---------------------------
dev-insurancelake-collect-to-cleanse-job

Python library path
s3://dev-insurancelake-<AWS Account Number>-us-east-2-etl-scripts/etl/lib/
01. custom_mapping.py,
02. datalineage.py,
03. dataquality_check.py,
04. datatransform_dataprotection.py,
05. datatransform_lookup.py,
06. datatransform_premium.py,
07. datatransform_premiumdemo.py,
08. datatransform_regex.py,
09. datatransform_reshape.py,
10. datatransform_typeconversion.py,
11. glue_catalog_helpers.py

Dependent JARs path
s3://dev-insurancelake-<AWS Account Number>-us-east-2-etl-scripts/etl/lib/
01. openlineage-spark-0.29.2.jar,
02. poi-ooxml-5.2.3.jar,
03. spark-excel_2.12-3.3.1_0.18.7.jar,
04. xmlbeans-5.1.1.jar
