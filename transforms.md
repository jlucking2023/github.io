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