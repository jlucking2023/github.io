
# Table of Contents

# Heading 1

## Heading 1.1

## Heading 1.2

# Heading 2


IL - InsuranceLake
3C’s - Collect, Cleanse, and Consume

IL Concepts
S3 Collect Bucket Database \ Table
The following eight AWS Services make up the 3C’s of InsuranceLake:

1. S3 - stores the incoming data files as well as the Apache Parquet Files
2. Lambda
3. Step Functions
4. Glue
5. DynamoDB
6. Athena
7. QuickSight
8. KMS

The following Key Concepts & Objects are important to keep in mind when working with IL:

* Batch File data sources - the most common types of data sources are large delimited text files, Excel files, and fixed length files. Therefore IL was initially built to process them by mapping source to target columns, transform each column, and apply data quality rules. 
* CDC / Streaming / Document data sources - IL cannot currently process these data sources, but can be enhanced relatively quickly to accept them.
* Collect to Cleanse Glue Job - 
* Cleanse to Consume Glue Job - 
* DIE Pattern: IL is based on Data files, Instruction files (CSV & JSON Metadata), and the generic Execution Engines (Glue PySpark Jobs).
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

A scalable ingestion framework is build on 3 pillars:
data (the subject);
metadata (the instructions);
code (the execution engine).

# QuickStart Install
Install the basics in 30 minutes
To set the region that InsuranceLake is installed in see the lib/configuration.py file.
https://gitlab.aws.dev/fsi-sat/aws-cdk-insurancelake-etl/-/blob/main/README.md
https://github.com/aws-samples/aws-insurancelake-etl/blob/main/README.md

# Full Install
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

