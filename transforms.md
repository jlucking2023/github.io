# InsuranceLake Collect-to-Cleanse Transform Index

<<<<<<< HEAD
|Name	|Description	|
|---	|---	|
|[currency](#currency)	|Convert specified numeric field with currency formatting to Decimal (fixed precision)	|
|[changetype](#changetype)	|Convert specified fields to decimal (fixed precision), int, bigint, string, etc.	|
|[date](#date)	|Convert specified date fields to ISO format based on known input format	|
|[implieddecimal](#implieddecimal)	|Convert specified numeric fields to Decimal (fixed precision) type with implied decimal point support (i.e. last 2 digits are to the right of decimal)	|
|[timestamp](#timestamp)	|Convert specified date/time fields to ISO format based on known input format	|
|[addcolumns](#addcolumns)	|Add two or more columns together in a new or existing column	|
|[columnfromcolumn](#columnfromcolumn)	|Add or replace column in DataFrame based on regexp group match pattern	|
|[columnreplace](#columnreplace)	|Add or replace a column in DataFrame with regex substitution on an existing column	|
|[combinecolumns](#combinecolumns)	|Add column to DataFrame using format string and source columns	|
|[filename](#filename)	|Add column in DataFrame based on regexp group match pattern on the filename argument to the Glue job	|
|[filldown](#filldown)  |Fill starting column value down the columns for all null values until the next non-null    |
|[rownumber](#rownumber)  |Adds row number column to rows based on a partition column list    |
|[filterrows](#filterrows)	|Filter out rows based on standard SQL WHERE statement	|
|[flipsign](#flipsign)	|Flip the sign of a numeric column in a Spark DataFrame, optionally in a new column	|
|[literal](#literal)	|Add column to DataFrame with static/literal value supplied in specification	|
|[lookup](#lookup)	|Replace specified column values with values looked up from an external table	|
|[merge](#merge)	|Merge columns using coalesce	|
|[multilookup](#multilookup)	|Add columns looked up from an external table using multiple conditions, returning any number of attributes	|
|[multiplycolumns](#multiplycolumns)	|Multiply two or more columns together in a new or existing column	|
|[titlecase](#titlecase)	|Convert specified string column in DataFrame to title or proper case	|
|[hash](#hash)	|Hash specified column values using SHA256	|
|[redact](#redact)	|Redact specified column values using supplied redaction string	|
|[tokenize](#tokenize)	|Replace specified column values with hash and store original value in DynamoDB table	|
|[earnedpremium](#earnedpremium)	|Calculate monthly earned premium	|
|[enddate](#enddate)	|Add a number of months to a specified date to get an ending/expiration date	|
|[expandpolicymonths](#expandpolicymonths)	|Expand dataset to one row for each month the policy is active with a calculated earned premium	|
|[policymonths](#policymonths)	|Calculate number of months between policy start/end dates	|

# Using Transforms

=======
|Formatting	|Description
|---	|---
|[currency](#currency)	|Convert specified numeric field with currency formatting to Decimal (fixed precision)
|[changetype](#changetype)	|Convert specified fields to decimal (fixed precision), int, bigint, string, etc.
|[date](#date)	|Convert specified date fields to ISO format based on known input format
|[implieddecimal](#implieddecimal)	|Convert specified numeric fields to Decimal (fixed precision) type with implied decimal point support (i.e. last 2 digits are to the right of decimal)
|[timestamp](#timestamp)	|Convert specified date/time fields to ISO format based on known input format
|[titlecase](#titlecase)	|Convert specified string column in DataFrame to title or proper case

|String Manipulation	|Description
|---	|---
|[columnfromcolumn](#columnfromcolumn)	|Add or replace column in DataFrame based on regexp group match pattern
|[columnreplace](#columnreplace)	|Add or replace a column in DataFrame with regex substitution on an existing column
|[combinecolumns](#combinecolumns)	|Add column to DataFrame using format string and source columns
|[literal](#literal)	|Add column to DataFrame with static/literal value supplied in specification
|[filename](#filename)	|Add column in DataFrame based on regexp group match pattern on the filename argument to the Glue job

|Data Security	|Description
|---	|---
|[hash](#hash)	|Hash specified column values using SHA256
|[redact](#redact)	|Redact specified column values using supplied redaction string
|[tokenize](#tokenize)	|Replace specified column values with hash and store original value in DynamoDB table

|Policy Data Operations	|Description
|---	|---
|[flipsign](#flipsign)	|Flip the sign of a numeric column in a Spark DataFrame, optionally in a new column
|[addcolumns](#addcolumns)	|Add two or more columns together in a new or existing column
|[multiplycolumns](#multiplycolumns)	|Multiply two or more columns together in a new or existing column
|[earnedpremium](#earnedpremium)	|Calculate monthly earned premium
|[enddate](#enddate)	|Add a number of months to a specified date to get an ending/expiration date
|[expandpolicymonths](#expandpolicymonths)	|Expand dataset to one row for each month the policy is active with a calculated earned premium
|[policymonths](#policymonths)	|Calculate number of months between policy start/end dates

|Structured Data	|Description
|---	|---
|[jsonexpandarray](#jsonexpandarray)    |Expand array type columns from JSON files into multiple rows
|[jsonexpandmap](#jsonexpandmap)    |Expand struct or map type columns from JSON files into multiple rows

|Miscellaneous Data Operations	|Description
|---	|---
|[lookup](#lookup)	|Replace specified column values with values looked up from an external table
|[multilookup](#multilookup)	|Add columns looked up from an external table using multiple conditions, returning any number of attributes
|[filldown](#filldown)  |Fill starting column value down the columns for all null values until the next non-null
|[filterrows](#filterrows)	|Filter out rows based on standard SQL WHERE statement
|[merge](#merge)	|Merge columns using coalesce
|[rownumber](#rownumber)  |Adds row number column to rows based on a partition column list

# Using Transforms

- Transform configuration is specified in the `transform-spec` section of the JSON configuration file. The filename follows the convention of `<database name>-<table name>.json` and is stored in the `/etl/transformation-spec` folder in the `etl-scripts` bucket. When using CDK for deployment, the contents of `/lib/glue_scripts/lib/transformation-spec` directory will be automatically deployed to this location.

- For an example of all transforms in one place, refer to the [all-transforms-example.json](../lib/glue_scripts/transformation-spec/all-transforms-example.json) in the `transformation-spec` directory of the repository.

>>>>>>> develop
- The order that you enter the transforms into the json file is important, and should be chosen deliberately. Each transform is executed in the order they are defined on the incoming dataset starting from the beginning of the transform_spec section of the file.

- Each transform type can only be used once in the transform specification. This limitation helps the transform implementation reduce the number of Spark withColumn statements (using withColumns or select), and [optimize the workflow performance](https://medium.com/@manuzhang/the-hidden-cost-of-spark-withcolumn-8ffea517c015).

- Except where noted, transforms will overwrite an existing field if specified as the result field. Where available use the `source` parameter to indicate that a different column should be used as the source data, and the column specified in the `field` parameter should be used for the result value. Specifying a source field to create a new column for transforms is useful for debugging issues with a transform, preserving original data, or having a backup datapoint available when incoming data formats are less clear.

---

## Formatting

### currency
<<<<<<< HEAD
Convert specified numeric field with currnecy formatting to Decimal (fixed precision)

- `format` parameter defaults to 16,2 if not specified
- While this transform will work on numeric fields, we recommend `changetype` to convert to decimal values because it is more efficient when combined with other data type changes.
=======
Convert specified numeric field with currency formatting to Decimal (fixed precision)

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the resulting decimal conversion, and source field if source not specified separately
|format    |optional    |Decimal precision and scale (separated by comma), defaults to 16,2
|source    |optional    |Name of source field, defaults to destination field
|euro    |optional    |If `true`, handle European (5.000.000,12) currency format, otherwise handle 5,000,000.12; defaults to `false`

- While this transform will work on numeric fields, we recommend `changetype` to convert to decimal values because it is more efficient when combined with other data type changes.
- This conversion essentially extracts any valid number from a string value; it removes any character that is not in `[0-9,-.]`
>>>>>>> develop

```json
"currency": [
    {
        "field": "SmallDollars",
        "format": "6,2"
    },
    {
        "field": "EuroValue",
        "source": "EuroValueFormatted",
        "euro": true
    }
]
```

### changetype
Convert specified fields to decimal (fixed precision), int, bigint, string, etc.

<<<<<<< HEAD
Field type syntax follows the [Spark simpleString](https://spark.apache.org/docs/3.3.0/api/python/_modules/pyspark/sql/types.html) definitions
=======
|Parameter    |Type    |Description
|---	|---	|---
|key    |required    |Name of the field to convert
|value  |required    |Destination data type expresseed using the [Spark simpleString](https://spark.apache.org/docs/3.3.0/api/python/_modules/pyspark/sql/types.html) syntax

- Transform spec is a single JSON object containing a list of string value pairs for each field to convert

- Transform can be used to rename a nested field in place by redefining the struct data type, with new field names using Spark's simpleString syntax for struct types, for example: `struct<name:type,name2:array<int>>`. See [all-transforms-example.json](../lib/glue_scripts/transformation-spec/all-transforms-example.json#L108) for a more complex example.
>>>>>>> develop

```json
"changetype": {
    "ExpiringPremiumAmount": "decimal(10,2)",
    "WrittenPremiumAmount": "decimal(10,2)",
    "EarnedPremium": "decimal(10,2)",
    "PrimaryKeyId": "bigint",
    "GrowingCount": "bigint",
<<<<<<< HEAD
    "PolicyKey": "string"
=======
    "PolicyKey": "string",
    "notes_struct": "json"
>>>>>>> develop
}
```

### date
Convert specified date fields to ISO format based on known input format

<<<<<<< HEAD
Date formats use [Spark datetime patterns](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html).
=======
|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the resulting date conversion, and source field if source not specified separately
|format    |required    |Date format specified using [Spark datetime patterns](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html)
|source    |optional    |Name of source field, defaults to destination field
>>>>>>> develop

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
        "source": "valuationdatestring",
        "format": "yyyyMMdd"
    }
]
```

### implieddecimal
<<<<<<< HEAD
Convert specified numeric fields to Decimal (fixed precision) type with implied decimal point support (i.e. last 2 digits are to the right of decimal)
=======
Convert specified numeric fields to Decimal (fixed precision) type with implied decimal point support (in other words, last 2 digits are to the right of decimal)

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the resulting decimal conversion, and source field if source not specified separately
|format    |required    |Decimal precision and scale (separated by comma)
|source    |optional    |Name of source field, defaults to destination field
|num_implied    |optional   |Number of implied decimal digits in the source field, defaults to `2`
>>>>>>> develop

- Use this transform to interpret decimal precision data stored in integer format, common in mainframe or flat file data formats.

```json
"implieddecimal": [
    {
        "field": "indemnity_paid_current_period",
        "num_implied": "4",
        "format": "16,4"
    },
    {
        "field": "claim_amount",
        "source": "claim_amount_string",
        "format": "16,2"
    }
]
```

### timestamp
Convert specified date/time fields to ISO format based on known input format

<<<<<<< HEAD
Timestamp formats use [Spark datetime patterns](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html).
=======
|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the resulting teimstamp conversion, and source field if source not specified separately
|format    |required    |Timestamp format specified using [Spark datetime patterns](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html)
|source    |optional    |Name of source field, defaults to destination field
>>>>>>> develop

```json
"timestamp": [
    {
        "field": "GenerationDate",
        "format": "yyyy-MM-dd HH:mm:ss.SSS+0000"
    },
    {
        "field": "DataLoadTimestamp",
        "source": "DataLoadString",
        "format": "yyyy-MM-dd HH:mm:ss.SSSZ"
    }
]
```

### titlecase
<<<<<<< HEAD
Convert specified string field to title or proper case
=======
Convert specified string field to title or proper case (for example, "my name" will become "My Name")

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of field to convert in place to title case

- Transform spec is a simple list of fields of type string to convert

>>>>>>> develop
```json
"titlecase": [
    "CompanyName",
    "AddressStreet"
]
```

---

<<<<<<< HEAD
## Data Manipulation

### addcolumns
Add two or more columns together in a new column
```json
"addcolumns": [
    {
        "field": "TotalWrittenPremium",
        "source_columns": [ "WrittenPremiumAmount" ]
    }
]
```

### columnfromcolumn
Add or replace column in DataFrame based on regexp group match pattern

- Only 1 (the first) match group will be used per specification block. For multiple groups, use multiple specification blocks and shift the parenthesis.

- Uses [Spark regexp_extract function](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.regexp_extract.html).

- Regular expressions follow the [Java Pattern syntax](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/regex/Pattern.html).

=======
## String Manipulation

### columnfromcolumn
Add or replace column based on regexp group match pattern

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the extracted pattern, and source field if source not specified separately
|pattern    |required    |Regular expression pattern with 1 match group following the [Java Pattern syntax](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/regex/Pattern.html)
|source    |optional    |Name of source field, defaults to destination field

- Only 1 (the first) match group will be used per specification block. For multiple groups, use multiple specification blocks and shift the parenthesis.
- Uses [Spark regexp_extract function](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.regexp_extract.html).

>>>>>>> develop
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

### columnreplace
<<<<<<< HEAD
Add or replace a column in DataFrame with regex substitution on an existing column

- Uses [Spark regexp_replace function](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.regexp_replace.html).

- Regular expressions follow the [Java Pattern syntax](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/regex/Pattern.html).

=======
Add or replace a column with regex substitution on an existing column

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the resulting substituted value, and source field if source not specified separately
|pattern    |required    |Regular expression pattern following the [Java Pattern syntax](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/regex/Pattern.html)
|replacement    |required   |String value to replace anything matched by the pattern
|source    |optional    |Name of source field, defaults to destination field

- Uses [Spark regexp_replace function](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.regexp_replace.html).

>>>>>>> develop
```json
"columnreplace": [
    {
        "field": "clean_date_field",
        "source": "bad_date_field",
        "pattern": "0000-00-00",
        "replacement": ""
    },
    {
        "field": "field_with_extra_data",
        "pattern": "[a-zA-z]{3,5}",
        "replacement": ""
    }
]
```

### combinecolumns
<<<<<<< HEAD
Add column to DataFrame using format string and source columns

- Uses [Python format string syntax](https://docs.python.org/3/library/string.html#format-string-syntax). Keyword arguments are not supported. Implicit references, positional references, and the [format specification mini-language](https://docs.python.org/3/library/string.html#format-specification-mini-language) are supported.
=======
Add column using a format string and list of source columns

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the resulting combined value, and source field if source not specified separately
|format    |required    |Format string using [Python format string syntax](https://docs.python.org/3/library/string.html#format-string-syntax). Implicit references, positional references, and the [format specification mini-language](https://docs.python.org/3/library/string.html#format-specification-mini-language) are supported. **Keyword arguments are not supported.**
|source_columns    |required    |List of source column names specified as a JSON array (at least 1 is required)
>>>>>>> develop

```json
"combinecolumns": [
    {
        "field": "RowKey",
        "format": "{}-{}-{}",
        "source_columns": [ "LOBCode", "PolicyNumber", "StartDate" ]
    }
]
```

<<<<<<< HEAD
### filename
Add column in DataFrame based on regexp group match pattern on the filename argument to the Glue job

- Regular expression group matching syntax follows the [Python regular expression syntax](https://docs.python.org/3/library/re.html)

- Only one (the first) match group will be used per specification block. For multiple groups, use multiple specification blocks and shift the parenthesis.

- Use the `required` parameter to optionally halt the workflow if the pattern is not matched. Without `required`, an unmatched group will be added as a null value string column.

=======
### literal
Add or replace column with supplied static/literal value

|Parameter    |Type    |Description
|---	|---	|---
|key    |required    |Name of the field to add or replace
|value  |required    |Literal value to store in the field (all [JSON data types](https://restfulapi.net/json-data-types/) supported, including objects, arrays, and null)

- Transform spec is a single JSON object containing a list of string value pairs for each field to create or replace

```json
"literal": {
    "source": "syntheticdata",
    "line_of_business": "synthetic"
}
```

### filename
Add or replace column based on regexp group match pattern on the incoming source data filename

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the extracted pattern
|pattern    |required    |Regular expression pattern with 1 match group following the [Python regular expression syntax](https://docs.python.org/3/library/re.html)
|required    |required    |true/false value indicating whether to halt the workflow if the pattern is not matched; if required is false and the pattern is not matched, a null value will be used

- Only one (the first) match group will be used per specification block. For multiple groups, use multiple specification blocks and shift the parenthesis.

>>>>>>> develop
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

<<<<<<< HEAD
### filldown
Fill starting column value down the columns for all null values until the next non-null

- This is a Spark implementation of [Pandas ffill](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.ffill.html) based on the article [How to utilize nested tables and window functions to fill down over null values by Robert O'Brien](https://towardsdatascience.com/tips-and-tricks-how-to-fill-null-values-in-sql-4fccb249df6f)

- This function is useful for replacing null values created by pivot tables in Excel that have category headers inline with only the first row of data. This will normalize the data ensuring that the headers are on all rows.

- Specify `sort` to order the data prior to filling down. Note that this will change the sort order of the data for subsequent transforms.

```json
"filldown": [
    {
        "field": "category"
    },
    {
        "field": "subcategory",
        "sort": [ "timestamp" ]
    }
]
```

### rownumber
Adds row number column to rows based on an optional partition column list, and optional sort column list. Use this transform to add row numbers, to index rows within categories, or to enumerate possible duplicate rows based on primary keys.

- When specifying the partition and sort options, always use a JSON list, even if there is a single column.

- Specifying a `sort` will change the sort order of the data for subsequent transforms.

```json
"rownumber": [
    {
        "field": "row_number"
    },
    {
        "field": "policy_month_index",
        "partition": [ "policynumber" ],
        "sort": [ "start_date" ]
    }
]
```

### filterrows
Filter out rows based on standard SQL WHERE statement

- Follows the [Spark WHERE clause syntax](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-where.html).

- Use only when certain rows can be systematically and confidently discarded. Examples of usage include removing blank rows, removing a totals rows, or removing subtotal rows. If review of filtered data is desired, consider using [data quality quarantine rules](./data_quality.md).

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

### flipsign
Flip the sign of a numeric column in a Spark DataFrame, optionally in a new column 
=======
---

## Data Security

### hash
Hash specified column values using SHA256

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of field to convert in place to SHA256 hash

- Transform spec is a simple list of fields of type string to convert
- If field does not exist, workflow will be halted to prevent unexpected schema changes from exposing sensitive data

```json
"hash": [
    "InsuredContactCellPhone",
    "InsuredContactEmail"
]
```

### redact
Redact/replace specified column values using supplied redaction string

|Parameter    |Type    |Description
|---	|---	|---
|key    |required    |Name of the field to replace
|value  |required    |Literal value to store in the field (all [JSON data types](https://restfulapi.net/json-data-types/) supported, including objects, arrays, and null)

- Transform spec is a single JSON object containing a list of string value pairs for each field to convert
- If field does not exist, workflow will be halted to prevent unexpected schema changes from exposing sensitive data

```json
"redact": {
    "CustomerNo": "****"
}
```

### tokenize
Replace specified column values with hash and store original value in DynamoDB table

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of field to convert in place to SHA256 hash; original value will be stored in a DynamoDB table

- Transform spec is a simple list of fields of type string to convert
- The `<environment>-insurancelake-etl-hash-values` DynamoDB table will be used for storage of all tokens for all fields and data sets. Since the hashing is deterministic, each value will only be stored once, regardless of how many columns contain the value.
- If field does not exist, workflow will be halted to prevent unexpected schema changes from exposing sensitive data

```json
"tokenize": [
    "EIN"
]
```

---

## Policy Data Operations

### flipsign
Flip the sign of a numeric column, optionally in a new column

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) numeric field to flip the sign (+/-), and source field if source not specified separately
|source    |optional    |Name of source field, defaults to destination field

>>>>>>> develop
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

<<<<<<< HEAD
### literal
Add column to DataFrame with static/literal value supplied in specification 

- Note that the transform is defined using a dictionary of field/value pairs, not a list.

```json
"literal": {
    "source": "syntheticdata",
    "line_of_business": "synthetic"
}
```

### lookup
Replace specified column values with values looked up from an dynamodb table
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
        "nomatch": "N/A",
        "source_system": "global"
=======
### addcolumns
Add two or more columns together in a new column

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the result of adding the source columns; can be the same field as one of the source columns, which will overwrite the original value with the sum
|source_columns    |required    |List of numeric source column names specified as a JSON array (at least 1 is required)

- Empty (null) source columns will be treated as 0 values

```json
"addcolumns": [
    {
        "field": "TotalWrittenPremium",
        "source_columns": [ "WrittenPremiumAmount" ]
>>>>>>> develop
    }
]
```

<<<<<<< HEAD
### merge
Merge column values using coalesce 

- Optionally specify a literal default value to use if all specified columns have a null value. Defaults to null.

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

### multilookup
Add columns looked up from an external table using multiple conditions, returning any number of attributes
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
"multilookup": [
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

### multiplycolumns
Multiply two or more columns together in a new or existing column

- Use for calculating premium splits
- `empty_value` parameter is optional and defaults to a value of 1.
=======
### multiplycolumns
Multiply two or more columns together in a new or existing column

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the result of multiplying the source columns; can be the same field as one of the source columns, which will overwrite the original value with the product
|source_columns    |required    |List of numeric source column names specified as a JSON array (at least 1 is required)
|empty_value    |optional   |Specifies the value to use for empty (null) fields, defaults to a value of `1`

- Use cases for this transform include calculating premium splits or allocating expenses
>>>>>>> develop

```json
"multiplycolumns": [
    {
        "field": "SplitPremium",
        "source_columns": [ "WrittenPremiumAmount", "SplitPercent1", "SplitPercent2" ],
        "empty_value": 0
    }
]
```

<<<<<<< HEAD
---

## Data Security

### hash
Hash specified column values using SHA256
```json
"hash": [
    "InsuredContactCellPhone",
    "InsuredContactEmail"
]
```

### redact
Redact specified column values using supplied redaction string

- Note that the transform is defined using a dictionary of field/value pairs, not a list.

```json
"redact": {
    "CustomerNo": "****"
}
```

### tokenize
Replace specified column values with hash and store original value in DynamoDB table

- The `<environment>-insurancelake-etl-hash-values` DynamoDB table will be used for storage of all tokens. Since the hashing is deterministic, each value will only be stored once, regardless of how many columns contain the value.

```json
"tokenize": [
    "EIN"
]
```

---

## Earned Premium

### earnedpremium
Calculate monthly earned premium

- Transform requires four dates fields and one or more written premium fields as inputs. If any of the date inputs have null values, the earned premium will be null. Null written premium values are treated as 0.

- The `period_start_date` and `period_end_date` parameters indicate the date columns for determining the earned premium calculation period for each row of data. These are usually the first and last day of the month, and are created by the [expandpolicymonths](#expandpolicymonths) transform.

- The `byday` parameter selects the calculation method. If true, earned premium will be proportional to the number of days in the reporting period. If false, earned premium will be divided evenly across all active policy months.

- For the `policy_effective_date` and `policy_expiration_date` parameters indicate the date columns for determining the lifetime of the policy. If `byday` is true, they will be used to pro-rate the earned premium for the first and last months of the policy. If `byday` is false, they will be used to identify the number of active policy months (always a whole number).
=======
### earnedpremium
Calculate monthly earned premium

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the calculated earned premium result; can be the same field as one of the `written_premium_list` columns, which will overwrite the original value with the result
|written_premium_list    |required    |List of numeric source column names containing written premium amounts specified as a JSON array (at least 1 is required)
|policy_effective_date   |required   |Indicates the existing date column to use for determining the start of the policy. When `byday` is `true`, this date will be used to pro-rate the earned premium for the first month of the policy; when `byday` is `false`, it will be used to identify the number of active policy months (always a whole number).
|policy_expiration_date  |required   |Indicates the existing date column to use for determining the end of the policy. When `byday` is `true`, this date will be used to pro-rate the earned premium for the first month of the policy; when `byday` is `false`, it will be used to identify the number of active policy months (always a whole number).
|period_start_date  |required   |Indicates the existing date column to use for determining the start of the earned premium calculation period for each row of data; usually this is the first day of the month and is created by the [expandpolicymonths](#expandpolicymonths) transform
|period_end_date    |required   |Indicates the existing date column to use for determining the end of the earned premium calculation period for each row of data; usually this is the last day of the month and is created by the [expandpolicymonths](#expandpolicymonths) transform
|byday    |optional   |Used to specify the calculation method: if `true`, earned premium will be proportional to the number of days in the reporting period; if `false`, earned premium will be divided evenly across all active policy months, defaults to `false`

- If you are overwriting an existing field and calculating the earned premium multiple times (for example, different methods), be aware that the operations will be processed in sequence and impact subsequent operations. In other words, the value that overwrites the field in the first operation will be used in the second operation and so on. If you need to calculate earned premium multiple times using the same inputs, you should use a new field for the result.

- If any of the date inputs have null values, the earned premium will be null. Empty or null written premium values are treated as 0.
>>>>>>> develop

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

### enddate
<<<<<<< HEAD
Add a number of months to a specified date to get an ending/expiration date
=======
Add a number of months to a specified date to calculate an ending/expiration date

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the calculated end date; can be an existing field, which will overwrite the original value with the result
|start_date |required   |Indicates the existing date column to use for determining the start of the policy
|num_months |required   |Indicates the existing numeric column to use for determining the number of policy months

>>>>>>> develop
```json
"enddate": [
    {
        "field": "CalcExpirationDate",
        "start_date": "EffectiveDate",
        "num_months": "Term"
    }
]
```

### expandpolicymonths
Expand dataset to one row for each month the policy is active with a calculated earned premium

<<<<<<< HEAD
- Use this transform to convert a list of insurance policies (one row per policy) to a list of active policy months (one row per month per policy). This transform will change the shape/size of the input data. However, it is possible to recover the original number of rows using a simple WHERE statement (e.g. `WHERE PolicyMonthIndex = 1`).

- Transform requires two date field parameters as inputs: `policy_effective_date` and `policy_expiration_date`. If either of the date field values are null, the policy row will not be expanded to any additional rows and will have null values for the policy month fields.

- Based on the `policy_month_start_field`, `policy_month_end_field`, and `policy_month_index` parameters, the transform will add colums to each row containing the first day of the month, the last day of the month, and the policy month number.

- Optionally use the `uniqueid` parameter to specify a field name to add with a generated GUID, unique to each policy.
=======
|Parameter    |Type    |Description
|---	|---	|---
|policy_effective_date   |required   |Indicates the existing date column to use for determining the start of the policy. When `byday` is `true`, this date will be used to pro-rate the earned premium for the first month of the policy; when `byday` is `false`, it will be used to identify the number of active policy months (always a whole number).
|policy_expiration_date  |required   |Indicates the existing date column to use for determining the end of the policy. When `byday` is `true`, this date will be used to pro-rate the earned premium for the first month of the policy; when `byday` is `false`, it will be used to identify the number of active policy months (always a whole number).
|policy_month_start_field  |required   |Indicates the name of the field to add to the dataset containing the first day of the month for the expanded row of data
|policy_month_end_field    |required   |Indicates the name of the field to add to the dataset containing the last day of the month for the expanded row of data
|policy_month_index |required   |Indicates the name of field to add to the dataset containing the expanded policy month index
|uniqueid   |optional   |Use to specify a field name to add with a generated GUID, unique to each policy

- Use this transform to convert a list of insurance policies (one row per policy) to a list of active policy months (one row per month per policy). This transform will change the shape/size of the input data; specifically, it will increase the number of rows to `number of policies x number of policy months`.
- This transform will add colums to each row containing the first day of the month, the last day of the month, and the policy month number/index.
- The index column is required so that it is always possible to recover the original number of rows using a simple WHERE statement (in other words, `WHERE PolicyMonthIndex = 1`).
- If either `policy_effective_date` or `policy_expiration_date` field values are null, the policy row will not be expanded to any additional rows and will have null values for the policy month and index fields.
- Index column values are 1-based, matching the array reference standard in Athena SQL.
>>>>>>> develop

```json
"expandpolicymonths": {
    "policy_effective_date": "EffectiveDate",
    "policy_expiration_date": "ExpirationDate",
    "uniqueid": "generated_policy_number",
    "policy_month_start_field": "StartDate",
    "policy_month_end_field": "EndDate",
    "policy_month_index": "PolicyMonthIndex"
}
```

### policymonths
<<<<<<< HEAD
Calculate number of months between policy start/end dates

- Optionally specify the `normalized` parameter as `true` to always return a whole number of months. If ommited `normalized` defaults to `false` and the number of months returned is a fractional number based on the exact number of days between the effective and expiration dates.
=======
Calculate number of months between policy start and end dates

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the calculated number of months; can be an existing field, which will overwrite the original value with the result
|policy_effective_date |required   |Indicates the existing date column to use for determining the start of the policy
|policy_expiration_date |required   |Indicates the existing date column to use for determining the end of the policy
|normalized |optional   |If `true` the calculated number of months will always be a whole number (uses Python's [rrule dateutil function](https://dateutil.readthedocs.io/en/stable/rrule.html) to perform a calendar walk); if `false` the calculated number of months will be a fractional number based on the exact number of days between the effective and expiration dates; defaults to `false`
>>>>>>> develop

```json
"policymonths": [
    {
        "field": "CalcNumMonths",
        "policy_effective_date": "EffectiveDate",
        "policy_expiration_date": "ExpirationDate",
        "normalized": true
    }
]
<<<<<<< HEAD
=======
```

---

## Structured Data

### jsonexpandarray
Converts an ArrayType column (typically created from loading JSON nested data) to 1 row per array element with index

|Parameter    |Type    |Description    |
|---	|---	|---	|
|field  |required   |Name of (destination) field to hold expanded array elements, and source ArrayType field if source is not specified separately    |
|source |optional   |Source ArrayType field; defaults to destination field
|index_field    |required   |Name of field to hold the expanded array element index

- This transform uses Spark's [posexplode_outer](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.posexplode_outer.html) function, so empty or null array values will remain in the dataset as a single row with null in the destination field
- Index column values are 1-based, matching the array reference standard in Athena SQL
- The index column is required so that there is always an easy way to get back to the data before being expanded (in other words, `where index == 1`)

```json
"jsonexpandarray": [
    {
        "field": "policyaddress",
        "source": "policyaddresses",
        "index_field": "policyaddress_index"
    }
]
```

### jsonexpandmap
Converts a MapType or StructType column (typically created from loading JSON nested data) to 1 row per map key, value pair with index column

|Parameter    |Type    |Description    |
|---	|---	|---	|
|field  |required   |Name of (destination) field to hold expanded map values, and source MapType or StructType field if source is not specified separately    |
|source |optional   |Source MapType or StructType field; defaults to destination field
|index_field    |required   |Name of field to hold the expanded map key, value pair index
|key_field  |required   |Name of field to hold the expanded map key name

- This transform uses Spark's [posexplode_outer](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.posexplode_outer.html) function, so empty or null map values will remain in the dataset as a single row with null in the destination field
- Index column values are 1-based, matching the array reference standard in Athena SQL
- The index column is required so that there is always an easy way to get back to the data before being expanded (in other words, `where index == 1`)

```json
"jsonexpandmap": [
    {
        "field": "activities",
        "index_field": "activity_index",
        "key_field": "activity_id"

    }
]
```

---

## Miscellaneous Data Operations

### lookup
Replace or add specified column values with values looked up from an DynamoDB table using a single value lookup key

|Parameter    |Type    |Description    |
|---	|---	|---	|
|field  |required   |Name of (destination) field to hold looked up values, and source field if source is not specified separately    |
|source |optional   |Source field with values matching the lookup data; defaults to destination field
|lookup    |required   |Name of lookup set of data which is used to match the `column_name` attribute in the DynamoDB table
|nomatch  |optional   |Value to use for lookups that have no match; defaults to null. **Must be the same data type as the looked up data.**
|source_system  |optional   |Value to use for the `source_system` attribute in the DynamoDB table; defaults to the database name or ([first level folder structure name in the Collect bucket](../README.md#bucket-layout)). Use this override parameter to share lookups across different databases.

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
        "nomatch": "N/A",
        "source_system": "global"
    }
]
```

- The provided `resources/load_dynamodb_lookup_table.py` script can be used to load prepared JSON data into the DynamoDB table:

    - Script parameters:
        |Parameter  |Type   |Description    |
        |---    |---    |---    |
        |source_system  |required   |String value that should match the source system name ([first level folder structure name in the Collect bucket](../README.md#bucket-layout)) for the workflow that will use the lookup
        |table_name |required   |The name of the DynamoDB table deployed by the InsuranceLake CDK stack for single value lookups, in the form `<environment>-<resource prefix>-etl-value-lookup`
        |data_file  |required   |Filename of the local JSON file containing lookup data to load into DynamoDB (format below)

    - Example usage:
        ```bash
        ./load_dynamodb_lookup_table.py SyntheticGeneralData dev-insurancelake-etl-value-lookup syntheticgeneral_lookup_data.json
        ```

    - JSON format of the lookup data file:
        ```json
        {
                "column_name1": { "lookup_value1": "lookedup_value1", "lookup_value2": "lookedup_value2", ... },
                "column_name2": { ... }
        }
        ```

### multilookup
Add columns looked up from an external table using multiple conditions, returning any number of attributes

|Parameter    |Type    |Description    |
|---	|---	|---	|
|lookup_group    |required   |Name of lookup set of data which is used to match the `lookup_group` attribute in the DynamoDB table; use to uniquely identify the set of lookup data
|match_columns    |required   |List of one or more columns specified as a JSON array to use for matching the lookup data; **the order of columns specified must match the order of the columns specified during the data load**
|return_attributes  |required   |Specifies the attribute names in the DynamoDB lookup table to add to the incoming dataset; defined as a JSON array and must contain at least one attribute
|nomatch  |optional   |Value to use for lookups that have no match, defaults to null. Used as the value for all `return_attributes` columns. **Must be the same data type as the looked up data.**

```json
"multilookup": [
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

- The `match_columns` names only refer to the incoming dataset. The column names in your lookup data (in DynamoDB) do not matter, because all the lookup column values are stored in a concatenated string in the lookup_item sort key.

- **Important Note:** If a column already exists, a duplicate column will be created, which will raise an error when saving to Parquet format. Take care to map your incoming dataset correctly so that it has unique column names after performing the multilookup transform. For example, suppose your incoming data has a `lineofbusiness` column, but it is composed of bespoke values that you want to normalize. Best practice would be to use the schema map to rename `lineofbusiness` to `originallineofbusiness` so the incoming data is preserved, and use the multilookup to return a new (normalized) `lineofbusiness` attribute value.

- The provided `resources/load_dynamodb_multilookup_table.py` script can be used to load prepared CSV data into the DynamoDB table:

    - Script parameters:
        |Parameter  |Type   |Description    |
        |---    |---    |---    |
        |table_name |required   |The name of the DynamoDB table deployed by the InsuranceLake CDK stack for multi-lookups, in the form `<environment>-<resource prefix>-etl-multi-lookup`. All multilookup lookup datasets are stored in the same table and grouped by lookup_group.
        |data_file  |required   |Filename of the local CSV file containing lookup data to load into DynamoDB
        |lookup_group  |required   |Any meaningful name to uniquely identify the lookup data in the DynamoDB table
        |lookup_columns |required   |One ore more columns in the CSV file to use as lookup values, listed **last**, separated by spaces. Note that field values from each specified column will be concatenated with a hyphen (`-`) separator to form a lookup key that matches the `lookup_item` attribute in the DynamoDB table. This is important to understand when editing the data in the future.

    - Example usage:
        ```bash
        ./load_dynamodb_multilookup_table.py dev-insurancelake-etl-multi-lookup lookups.csv PolicyData-LOBCoverage originalprogram originalcoverage
        ```

    - The lookup data file should be saved as CSV and include all the match columns and return value columns. It is ok to have some columns that are not used, because the transform specification allows the user to select the specific return columns they want in each transform.

### filldown
Fill starting column value down the columns for all null values until the next non-null

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of column on which to perform the filldown operation
|sort    |optional    |List of columns to use for sorting of the data before filling down, specified as a JSON array. This will change the order of the data for subsequent transforms. Defaults to no sort (data is left in the state from which it was loaded or from the last transform).

- This function is useful for replacing null values created by pivot tables in Excel that have category headers inline with only the first row of data. This will normalize the data ensuring that the headers are on all rows.

- This function works by partitioning the data over non-null values in the columns, so it is important that your rows of data are organized such that the non-null values indicate the values you want to fill in the subsequent rows of data. If your data is not already organized in this way, use the sort optional parameter.

- This is a Spark implementation of [Pandas ffill](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.ffill.html) based on the article [How to utilize nested tables and window functions to fill down over null values by Robert O'Brien](https://towardsdatascience.com/tips-and-tricks-how-to-fill-null-values-in-sql-4fccb249df6f)

```json
"filldown": [
    {
        "field": "category"
    },
    {
        "field": "subcategory",
        "sort": [ "timestamp" ]
    }
]
```

### filterrows
Filter out rows based on standard SQL WHERE statement

|Parameter    |Type    |Description
|---	|---	|---
|condition    |required    |String filter condition using [Spark WHERE clause syntax](https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-where.html); **rows that match will remain in the data set**
|description    |optional    |This parameter will be ignored, but we recommend using it to document the purpose of each filter condition

- Use only when certain rows can be systematically and confidently discarded. Examples of usage include removing blank rows, removing a totals rows, or removing subtotal rows. If review of filtered data is desired, consider using [data quality quarantine rules](./data_quality.md).

```json
"filterrows": [
    {
        "description": "Claim number or file number is required",
        "condition": "claim_number is not null or file_number is not null"
    },
    {
        "condition": "`startdate` >= cast('1970-01-01' as date)"
    }
]
```

### merge
Merge column values using coalesce 

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the result of merging/coalescing the source columns; can be the same field as one of the source columns, which will overwrite the original value
|source_list    |required    |List of source column names specified as a JSON array (at least 1 is required)
|default    |optional   |Specifies the literal value to use when all source columns have empty (null) values, defaults to null

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

### rownumber
Adds row number column to rows based on an optional partition column list, and optional sort column list. Use this transform to add row numbers, to index rows within categories, or to enumerate possible duplicate rows based on primary keys.

|Parameter    |Type    |Description
|---	|---	|---
|field    |required    |Name of (destination) field to hold the rownumber result; can be an existing field, which will overwrite the original value
|partition  |optional   |List of columns to partition over (look for changing values) specified as a JSON array; if not specified, the function will number every row of data in the set sequentially
|sort    |optional    |List of columns to use for sorting of the data before numbering, specified as a JSON array. This will change the order of the data for subsequent transforms. Defaults to no sort (data is left in the state from which it was loaded or from the last transform).

```json
"rownumber": [
    {
        "field": "row_number"
    },
    {
        "field": "policy_month_index",
        "partition": [ "policynumber" ],
        "sort": [ "start_date" ]
    }
]
>>>>>>> develop
```