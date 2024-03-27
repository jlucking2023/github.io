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

## Contents

### Installation
* [30 minute QuickStart](README-ETL.md#quickstart) 
* [60 minute QuickStart with CI/CD](README-ETL.md#quickstart-with-cicd)
* [Full Deployment Guide (central CI/CD environment, and 3 deployment environments](full_deployment_guide.md)

### User Documentation
* [Detailed Collect-to-Cleanse transform reference](transforms.md)
* [Schema Mapping Documentation](https://github.com/aws-samples/aws-insurancelake-etl/blob/main/resources/schema_mapping.md)
* [File Formats and Input Specification Documentation](file_formats.md)
* [Data quality rules with Glue Data Quality reference](data_quality.md)
* [Using SQL for Cleanse-to-Consume](using_sql.md)

### Developer Documentation
* [Developer Guide](developer_guide.md)
* [AWS CDK Detailed Instructions](cdk_instructions.md)
* [Github / CodePipeline Integration Guide](github_guide.md)


## Configuration
Make a change an deploy automatically with self-mutating CodePipeline

No VPC's needed
- InsuranceLake can be deployed with no VPC(s) simply by removing the subnet definition in configuration.py.
- The public subnet is completely optional as well. InsuranceLake does not require any VPC, so it also does not require public subnets. Creating a VPC with half public subnets and half private is the default behavior. You can modify this by passing the subnet_configuration parameter to the VPC creation in lib/vpc_stack.py.
- If the VPC is enabled in InsuranceLake, Glue is really the only service that will use it, and specifically, for Glue connections. If you try this out, you’ll see that the Glue connections specifically select the private subnet from the InsuranceLake-created VPC, through the vpc.subnets method.

To set the region that InsuranceLake is installed in see the lib/configuration.py file.


## Collect Data
dev-insurancelake-<account ID>-us-east-2-collect > <database name> > <table name>

[Detailed File Format and Input Specification Documentation](file_formats.md)


## Cleanse Data
dev-insurancelake-<account ID>-us-east-2-cleanse > <database name> > <table name>

[Detailed Schema Mapping Documentation](schema_mapping.md)
[Detailed Collect-to-Cleanse transform reference](transforms.md)
[Data Quality reference](data_quality.md)


## Consume Data
dev-insurancelake-<account ID>-us-east-2-consume > <database name> > <table name>

[Using SQL for Cleanse-to-Consume](using_sql.md)


## Entity Match

Time Travel
Track changes to claims from TPA

Full documentation in development


## Operation

### Partitions

Purge and Reload Data
Replace a previously loaded dataset (same day and different day)
instructions on how to handle re-loading the same data file (perhaps Feb's Broker A data had an error and you want to reload it)

### Monitor Data Quality

### Monitor Data Lineage


## Notes

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

Zero filled date values can be handled with the columnreplace transform, which was created to convert them into null values

Sedgwick e02 Claim Interface File Option 1: manually split files into separate files for each record type (because each record type has a different schema)
```
grep ^<record type code> e02file > e02file-filtered
grep ^CLM e02file > e02file-claims
```

Sedgwick e02 Claim Interface File Option 2: use lambda split file function

## Roadmap
- Data Quality rule descriptions
- Integrated Data Quality warning alerts

