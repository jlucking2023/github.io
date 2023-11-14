Explain where they go in the Transform JSON file and the importance of the order that they are inserted into it.
22 transforms taken from here

addcolumns

"addcolumns": [
            {
                "field": "TotalWrittenPremium",
                "source_columns": [ "WrittenPremiumAmount" ]
            }
        ]



columnfromcolumn

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

combinecolumns

"combinecolumns": [
            {
                "field": "RowKey",
                "format": "{}-{}-{}",
                "source_columns": [ "LOBCode", "PolicyNumber", "StartDate" ]
            }
        ]



currency

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



date

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



decimal

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



earnedpremium

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



enddate

"enddate": [
            {
                "field": "CalcExpirationDate",
                "start_date": "EffectiveDate",
                "num_months": "Term"
            }
        ]



expandpolicymonths

"expandpolicymonths": {
            "uniqueid": "generated_policy_number",
            "policy_month_start_field": "StartDate",
            "policy_month_end_field": "EndDate",
            "policy_effective_date": "EffectiveDate",
            "policy_expiration_date": "ExpirationDate"
        }



filename

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



filterrows

"filterrows": [
            {
                "condition": "claim_number is not null or file_number is not null"
            },
            {
                "condition": "`startdate` >= cast('1970-01-01' as date)"
            }
        ]        



flipsign

"flipsign": [
            {
                "field": "Balance"
            },
            {
                "field": "NewAccountBalance",
                "source": "AccountBalance"
            }
        ]



hash

"hash": [
            "InsuredContactCellPhone",
            "InsuredContactEmail"
        ]



implieddecimal

"implieddecimal": [
            {
                "field": "indemnity_paid_current_period",
                "format": "16,2"
            }
        ]



literal

"literal": {
            "source": "syntheticdata"
        }



merge

"merge": [
            {
                "field": "insuredstatemerge",
                "source_list": [
                    "insuredstatename", "insuredstatecode"
                ],
                "default": "Unknown"
            }
        ]



policymonths

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



redact

"redact": {
            "CustomerNo": "****"
        }



timestamp

"timestamp": [
            {
                "field": "GenerationDate",
                "format": "yyyy-MM-dd HH:mm:ss.SSS+0000"
            }
        ]


