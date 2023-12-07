# InsuranceLake Collect-to-Cleanse Transform Index

|Name	|Description	|
|---	|---	|
|hash	|Hash specified column values using SHA256 and Spark UDF	|
|redact	|Redact specified column values using supplied redaction string	|
|tokenize	|Replace specified column values with hash and store original value in separate table	|
|lookup	|Replace specified column values with values looked up from an external table	|
|merge	|Merge columns using coalesce	|
|enddate	|Add a number of months to a specified date to get an ending/expiration date	|
|policymonths	|Calculate number of months between policy start/end dates	|
|expandpolicymonths	|Expand dataset to one row for each month the policy is active with a calculated earned premium	|
|earnedpremium	|Calculate monthly earned premium	|
|addcolumns	|Add two or more columns together in a new or existing column	|
|multiplycolumns	|Multiply two or more columns together in a new or existing column	|
|flipsign	|Flip the sign of a numeric column in a Spark DataFrame, optionally in a new column	|
|filename	|Add column to DataFrame based on regexp pattern on the filename argument to the Glue job	|
|columnfromcolumn	|Add column to DataFrame based on regexp pattern on another column	|
|literal	|Add column to DataFrame with static/literal value supplied in specification	|
|combinecolumns	|Add column to DataFrame using format string and source columns	|
|date	|Convert specified date fields to ISO format based on known input format	|
|timestamp	|Convert specified date/time fields to ISO format based on known input format	|
|decimal	|Convert specified numeric field (usually Float or Double) fields to Decimal (fixed precision) type	|
|implieddecimal	|Convert specified numeric field (usually Float or Double) fields to Decimal (fixed precision) type with implied decimal point support (i.e. last 2 digits are to the right of decimal)	|
|currency	|Convert specified numeric field with currnecy formatting to Decimal (fixed precision)	|
|filterrows	|Filter out rows based on standard SQL WHERE statement	|
|multilookup	|Add columns looked up from an external table using multiple conditions, returning any number of attributes	|
|bigint	|Convert specified numeric column to bigint	|
|titlecase	|Convert specified string column in DataFrame to title case	|
|columnreplace	|Add or replace a column in DataFrame with regex substitution on an existing column	|

# Using Transforms
The order that you enter the transforms into the json file is very important . Each transform is executed on the incoming dataset starting from the beginning of the transform_spec section of the file.

---------------------------
## Formatting

- currency : Convert specified numeric field with currnecy formatting to Decimal (fixed precision)
```json
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
```json
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
```json
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
```json
  "implieddecimal": [
        {
          "field": "indemnity_paid_current_period",
          "format": "16,2"
        }
        ]
```

- timestamp	Convert specified date/time fields to ISO format based on known input format
```json
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
```json
  "addcolumns": [
        {
          "field": "TotalWrittenPremium",
          "source_columns": [ "WrittenPremiumAmount" ]
        }
        ]
```

- multiplycolumns : Multiply two or more columns together in a new column
```json
		"multiplycolumns": [
			{
				"field": "SplitPremium",
				"source_columns": [ "WrittenPremiumAmount", "SplitPercent1", "SplitPercent2" ],
				"empty_value": 0
			}
		]
```

- columnfromcolumn : Add column to DataFrame based on regexp pattern on another column
```json
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
```json
		"columnreplace": [
			{
				"field": "clean_date_field",
				"source": "bad_date_field",
				"pattern": "0000-00-00",
				"replacement": ""
			}
		]
```

- combinecolumns : Add column to DataFrame using format string and source columns
```json
  "combinecolumns": [
        {
          "field": "RowKey",
          "format": "{}-{}-{}",
          "source_columns": [ "LOBCode", "PolicyNumber", "StartDate" ]
        }
        ]
```

- filename : Add column to DataFrame based on regexp pattern on the filename argument to the Glue job
```json
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
```json
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
```json
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
```json
  "literal": {
          "source": "syntheticdata"
        }
```

- lookup : Replace specified column values with values looked up from an dynamodb table
- Future: use / enhance existing DynamoDB load python code (located here?)
- Future: add a copy between Dev-Test-Prod in DevSecOps Process
```json
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
```bash
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


```json
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
```json
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
```json
  "redact": {
        "CustomerNo": "****"
        }
```

- hash : Hash specified column values using SHA256 and Spark UDF
```json
  "hash": [
          "InsuredContactCellPhone",
          "InsuredContactEmail"
        ]
```  

- tokenize : Replace specified column values with hash and store original value in separate table
```json
  "tokenize": [
        "EIN"
        ]
```
---------------------------
## Earned Premium

- enddate : Add a number of months to a specified date to get an ending/expiration date
```json
  "enddate": [
        {
        "field": "CalcExpirationDate",
        "start_date": "EffectiveDate",
        "num_months": "Term"
        }
  ]
```

- expandpolicymonths : Expand dataset to one row for each month the policy is active with a calculated earned premium
```json
  "expandpolicymonths": {
        "uniqueid": "generated_policy_number",
        "policy_month_start_field": "StartDate",
        "policy_month_end_field": "EndDate",
        "policy_effective_date": "EffectiveDate",
        "policy_expiration_date": "ExpirationDate"
  }
```

- policymonths : Calculate number of months between policy start/end dates
```json
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
```json
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
