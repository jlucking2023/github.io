proposal for a new transform function, the vcoalesce. aggregating by the first field, records identified by the second field may or may not always have the third field filled. vcoalesce filled in that third field based on other records that shared the same first field
<img width="422" alt="image" src="https://github.com/jlucking2023/github.io/assets/147445875/5c143d90-8e97-4405-89ec-a34910f6e759">
https://stackoverflow.com/questions/67065847/spark-dataframe-filldown

-----------------------------
Demonstrates an "unpivot" operation on the Construction Noncaptive Premium data using Spark's stack function in Spark SQL. The column naming in the select statement likely needs to be adjusted for your workflow. You'll probably notice in the result data that there are duplicate rows any time one of the 7 insurance companies has no data (either because the broker hasn't provided the split data, or the policy has less than 7 splits). These duplicate rows are a side effect that I'd like to eliminate
select
	site_name,
	project_name,
	aws_project_code,
	policy_year,
	status,
	amazon_risk_management_contact,
	division,
	project_type_1,
	project_type_2,
	scope_of_work,
	address1,
	city,
	`state`,
	country,
	zip,
	latitude,
	longitude,
	hard_cost,
	equipment,
	delay_in_completion,
	tiv,
	date_of_first_notification,
	commencement_date,
	effective_date,
	expiration_date,
	days_to_expiration,
	project_premium,
	local_taxes,
	composite_annual_rate_per_100_pd_value,
	qtr_crt,
	notes_1,
	construction_manager_email,
	technical_pm_email,
	general_contractor_1,
	general_contractor_2,
​
	stack(7,
		insurance_co_1, insurance_co_1_participation, insurance_co_1_premium, insurance_co_1_participation * project_premium, 
		insurance_co_2, insurance_co_2_participation, insurance_co_2_premium, insurance_co_2_participation * project_premium, 
		insurance_co_3, insurance_co_3_participation, insurance_co_3_premium, insurance_co_3_participation * project_premium, 
		insurance_co_4, insurance_co_4_participation, insurance_co_4_premium, insurance_co_4_participation * project_premium, 
		insurance_co_5, insurance_co_5_participation, insurance_co_5_premium, insurance_co_5_participation * project_premium, 
		insurance_co_6, insurance_co_6_participation, insurance_co_6_premium, insurance_co_6_participation * project_premium, 
		insurance_co_7, insurance_co_7_participation, insurance_co_7_premium, insurance_co_7_participation * project_premium
​
	) as (insurance_co, insurance_co_participation, insurance_co_split_premium, insurance_co_calc_premium),
​
	execution_id,
	year,
	month,
	day
​
from test.premium

-----------------------------
