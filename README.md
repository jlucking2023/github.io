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

- [Detailed Collect-to-Cleanse transform reference](transforms.md)
- [Data Quality reference](data_quality.md)

# Installation
To set the region that InsuranceLake is installed in see the lib/configuration.py file.
- [30 minute QuickStart](https://github.com/aws-samples/aws-insurancelake-etl/blob/main/README.md#quickstart) 
- [60 minute QuickStart with CI/CD](https://github.com/aws-samples/aws-insurancelake-etl/blob/main/README.md#quickstart-with-cicd)
- [Full Deployment Guide (central CI/CD environment, and 3 deployment environments](./full_deployment_guide.md)

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
[Detailed Collect-to-Cleanse transform reference](transforms.md)

---------------------------
## Data Quality Rules
[Data Quality reference](data_quality.md)

quarantine
Build and test insurance data quality rules

* warn
* fail

Data Quality rule descriptions
Data Quality warning alerts

Note on Transforms & Data Quality
here might be situation where day and month is reversed and if for some dates the day is <=12 it will still transform them considering them as day while they are actually month. Similarly, as we have seen in the case of wrong formats where we expressed MM as mm it used its interpretation of facts.

---------------------------
Entity Match



# Consume Data
dev-insurancelake-<account ID>-us-east-2-consume > <database name> > <table name>

---------------------------
## Spark SQL
spark_<name>.sql - 1 statement in file is allowed

- columns to rows
spark stack function





---------------------------
## Athena SQL
athena_<name>.sql - multiple statements in file is allowed

# Monitor Data Quality


# MonitorData Lineage


# Using Transforms
The order that you enter the transforms into the json file is very important . Each transform is executed on the incoming dataset starting from the beginning of the transform_spec section of the file.

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
Permissive - Overwrite previous with incoming data
Strict - Don't do anything, preserve the original
Reorder - If schemas are all the same but the columns are out of order, put them in the original order
Evolve - Merge changes


---------------------------
Duplicate Column Names
Spark can handle duplicate column names by appending a number representing the index of the column.

---------------------------
Time Travel
Track changes to claims from TPA

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
