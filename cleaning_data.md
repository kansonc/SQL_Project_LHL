What issues will you address by cleaning the data?

1. Since PostgreSQL/PGAdmin had difficultly loading the CSV data, all columns were imported as VarChar, and cleaning including converting datatypes
2. Deleting Duplicate primary keys in tables
3. Determining whether to trimming spaces from the beginning and end of the data
4. Determining whether converting the case of words has any effect
5. Properly converting all currency based columns to a Money datatype and by dividing by 1,000,000 wfor consitency.
6. Checking non-negativity contraints after converting number types and removing records that are negative (does not apply to sentiment_score)
7. Checking whether the amounts ordered exceed the amounts in stock on the products and sales_report tables
8. Removing columns of irrelevant data (same repeated or entirely null values)
9. Determining whether null values should stay null
10. Determining whether values that are the absence of should be convert to Null or to another nomenclature


QUERYS FOR EACH TABLE, AS FOLLOWS:


/* 	QUERY TO CREATE all_sessions table
The description of all data cleaning for a particular column preceeds its select statement clause.
Duplicates have not been removed.
Columns containing null values only have been removed.
Datatypes have been assigned accordingly
*/

drop table if exists all_sessions;

CREATE TABLE all_sessions as
(
Select 
	/*FullVisitorID:
	column to snakecase
	Type: VarChar as some IDs start with 0 and is nominal
	No trimming needed
	no nulls (no conversion necessary
	Is not distinct*/
	fullVisitorId as full_visitor_id,

	/*Channelgrouping
	column to snakecase
	No trimming needed
	Type: VarChar and nominal
	Contains 7 categories 
	no nulls (no conversion necessary)
	*/
	channelGrouping  as channel_grouping,

	/* time
	Cast to INT as unit of time cannot be determined
	no nulls (no conversion necessary)
	No trimming needed
	*/
	cast(time as int)  as "time",

	/* country
	no nulls (no conversion necessary)
	(not set) is the value for missing values (24 instances)
	Maintain capitalized format as the first letter of each word
	No trimming needed 
	Type: VarChar Nominal
	*/
	country,

	/* city
	no nulls (no conversion necessary)
	(not set) is the value for missing values (24 instances)
	‘not available in the demo dataset’ must be changed to (not set)
	Maintain capitalized format as the first letter of each word
	No trimming needed 
	Type: VarChar Nominal
	*/
	CASE WHEN city like 'not available%' THEN '(not set)' ELSE city END as city,

	/* totaltransactionrevenue
	column to snakecase
	Maintain null values
	No trimming needed
	Type: Money (No decimal, Negative) Must be divided by 1,000,000
	*/
	CAST((CAST(totaltransactionrevenue AS NUMERIC) /1000000) AS MONEY) as total_transaction_revenue,

	/* transactions
	Type: BOOLEAN (No decimal, negative)
	Null values are 0 (False); must convert
	No trimming needed
	*/
	CAST(COALESCE(transactions, '0') AS BOOL) as transactions,

	/*timeOnSite
	column to snakecase
	Cast to INT as unit of time cannot be determined
	Maintain nulls as 0 means no time on site and 0 is not a value
	No trimming needed
	*/
	cast(timeOnSite as int) as time_on_site,

	/* pageViews
	column to snakecase
	no nulls (no conversion necessary)
	Type: INT (No decimals, negatives)
	No trimming needed
	*/
	cast(pageviews as int) as page_views,

	/* sessionsQualityDim
	Maintain nulls as value of zero has meaning
	Type: INT (no decimal, negative)
	No trimming needed
	*/
	CAST(sessionQualityDim AS INT) as session_quality_dim,

	/* date
	no nulls (no conversion necessary)
	No trimming needed
	Fixed length
	Type: Date
	*/
	to_date("date", 'YYYYMMDD') as date,

	/* visitorid
	no nulls (no conversion necessary)
	Fixed length
	Type: VarChar as Nominal Data
	Not Distinct
	column to snakecase
	*/
	visitId as visit_id,

	/*type
	no nulls (no conversion necessary)
	2 values, both ALL CAPS
	No trimming needed
	Type: VarChar Nominal
	*/
	type as "type",

	/* productRefundAmount
	Contains all Null Values and will not be included in final table
	*/

	/* productQuantity
	53 Not NULL
	Type: INT (no decimal, negative)
	Maintain Null Values as Zero means No Quantity
	No trimming needed
	column to snakecase
	*/
	CAST(productQuantity AS INT) as product_quantity,

	/*productprice
	Contains many 0s
	Type: Money (No decimal, Negative) Must be divided by 1,000,000
	No trimming needed
	column to snakecase
	*/
	CAST((CAST(productprice AS NUMERIC) /1000000) AS MONEY) as product_price,

	/* productRevenue
	Only 4 non null values 
	Only 4 non null values, corresponds to total revenue
	No Null conversion
	Type: Money (No decimal, Negative) Must be divided by 1,000,000
	No trimming needed
	column to snakecase
	*/
	CAST((CAST(productRevenue AS NUMERIC) /1000000) AS MONEY) as product_revenue,

	/*productSKU
	3 values contain a spaces
	Foreign Key from different table
	No Null Values
	No trimming needed
	Type: VarChar Nominal
	column to snakecase
	*/
	productSKU as product_sku,

	/* v2ProductName  
	No Nulls (no conversion)
	No trimming needed
	Type: VarChar Nominal
	column to snakecase
	*/
	v2productName as v2_product_name,

	/* v2ProductCategory
	No Nulls (no conversion)
	(not set) is the null value
	Value ‘${escCatTitle}’ must be converted to (not set)
	Type: VarChar Nominal
	column to snakecase
	*/
	CASE WHEN v2ProductCategory LIKE '${escCatTitle}' THEN '(not set)' ELSE v2ProductCategory END as v2_product_category,

	/* productVariant
	No Nulls (no conversion)
	(not set) is the null value
	No trimming necessary	
	Type: VarChar Nominal
	column to snakecase
	Most Descriptions have these values in them
	*/
	productVariant as product_variant,

	/* currencyCode
	Contains only 1 value ‘USD’
	Contains Null Vales – will not convert
	Type: VarChar Nominal 
	No trimming necessary	
	column to snakecase
	*/
	currencyCode as currency_code,

	/* itemQuantity
	Contains Null Values only
	Will not be added to the table
	*/

	/* itemRevenue
	Contains Null Values only
	Will not be added to the table
	*/

	/* transactionRevenue
	Type: Money (No decimal, Negative) Must be divided by 1,000,000
	column to snakecase
	Only 4 values, corresponds to product revenue
	*/
	CAST((CAST(transactionRevenue AS NUMERIC) /1000000) AS MONEY) as transaction_revenue,

	/* transactionId
	Contains only 9 distinct values
	4 of 9 values contain transaction revenue and product revenue
	Type: VarChar Nominal 
	No trimming necessary	
	column to snakecase
	*/
	transactionId as transaction_id,

	/* pageTitle
	Contains 1 null value (no conversion necessary)
	Type: VarChar Nominal 
	No trimming necessary	
	column to snakecase
	*/
	pageTitle as page_title,

	/* searchKeyword
	All Null Values
	Will not be added to the table
	*/

	/* pagePathLevel1
	11 distinct values
	No nulls
	Type: VarChar Nominal 
	No trimming necessary	
	column to snakecase
	*/
	pagePathLevel1 as page_path_level_1,

	/* eCommerceAction_type
	7 categories (range 0 – 6)
	Type: VarChar Nominal
	No nulls
	column to snakecase
	*/
	eCommerceAction_type as ecommerce_action_type,

	/* eCommerceAction_step
	3 categories (range 1 – 3)
	‘2’ corresponds to action ‘Payment’
	‘3’ corresponds to action 'Review’
	Type: VarChar Nominal
	No nulls
	column to snakecase
	*/
	eCommerceAction_step as ecommerce_action_step,

	/* eCommerceAction_option
	‘Payment’ corresponds to step ‘2’
	‘Review’ corresponds to step ‘3’
	‘Billing and Shipping' corresponds to type ‘5’
	Type: VarChar Nominal
	Null Values need not change
	column to snakecase
	*/
	eCommerceAction_option as ecommerce_action_option,

	/* 
	Create a primary key duplicate counter
	*/
	Row_Number () Over(Partition by fullVisitorId, visitId, productSKU) as row_num

from 	zimport_all_sessions);

/* End of QUERY TO CREATE all_sessions TABLE */


/**************************************/


/*	Query to Check non-negativity constraints of the all_sessions table	*/

Select 
	(Select Count(*) from products Where "time" < 0) as countneg_time,
	(Select Count(*) from products Where total_transaction_revenue < '0') as countneg_total_transaction_revenue,
	(Select Count(*) from products Where time_on_site < 0) as countneg_time_on_site,
	(Select Count(*) from products Where page_views < 0) as countneg_page_views,
	(Select Count(*) from products Where session_quality_dim < 0) as countneg_session_quality_dim,
	(Select Count(*) from products Where product_quantity < 0) as countneg_product_quantity,
	(Select Count(*) from products Where product_price < '0') as countneg_product_price,
	(Select Count(*) from products Where product_revenue < '0') as countneg_product_revenue,
	(Select Count(*) from products Where transaction_revenue < '0') as countneg_transaction_revenue
from all_sessions
LIMIT 1;
--RESULT: NO NEGATIVE VALUES COUNTED, DATA IS VALID

/*	End of Query to Check non-negativity constraints of the all_sessions table	*/


/**************************************/


/* Query to remove duplicates from all_sessions Table */

Delete from all_sessions
Where row_num > 1;

Alter table all_sessions
drop column row_num;

--this deleted 5 rows

/* End of Query to remove duplicates from all_sessions Table */


/**************************************/


/* Query to add constraints to all_sessions table */

Alter Table all_sessions
ADD Primary Key (full_visitor_id, visit_id, product_sku)

/* End Query to add constraints to all_sessions table */



/**************************************/
/**************************************/
/**************************************/
/**************************************/
/**************************************/



/* 	Query to create products table
The description of all data cleaning for a particular column preceeds its select statement clause.
There are NO Duplicates
Datatypes have been assigned accordingly
*/

CREATE TABLE products as
(Select
	/* SKU
	Primary Key
	No duplicates
	No trimming necessary (no spaces)
	Type: VarChar Nominal
	convert to snakecase
	*/
	sku as product_sku,

	/* name
	Is not unique
	No Nulls (no conversion nessary)
	MUST TRIM (spaces before and after)
	Type: VarChar Nominal
	convert to snakecase
	*/
	Trim(both from "name") as product_name,

	/* orderedquantity
	Type: INT (no decimal, negative)
	No nulls (no conversion necessary)
	No need to trim
	convert to snakecase
	*/
	Cast (orderedquantity as INT) as ordered_quantity,

	/*stocklevel
	Type: INT (no decimal, negative)
	No need to trim
	convert to snakecase
	No nulls (no conversion necessary)
	*/
	CAST (stocklevel AS INT) as stock_level,

	/*restockingleadtime
	Type: INT (no decimal, negative)
	No need to trim
	convert to snakecase
	No nulls (no conversion necessary)
	*/
	CAST(restockingLeadTime as INT) as restocking_lead_time,

	/* sentimentscore
	Type: NUMERIC (HAS BOTH DECIMAL AND NEGATIVE)
	No need to trim
	convert to snakecase
	HAS 1 NULL, SAME AS sentimentmagnitude
 	Range is from -0.2 to 1.0
	*/
	TO_NUMBER	(sentimentscore, 'S9D99') as sentiment_score,

	/* sentimentmagnitude
	Type: NUMERIC (HAS DECIMAL but is not negative)
	No need to trim
	convert to snakecase
	HAS 1 NULL SAME AS sentimentscore
 	Range is from 0 to 2.0
	*/
	TO_NUMBER	(sentimentmagnitude, '9D99') as sentiment_magnitude

from zimport_products);

/*	End of QUERY TO CREATE products TABLE	*/


/**************************************/


/*	Query to Check non-negativity constraints and consistency of the product table	*/

Select 
	(Select Count(*) from products Where ordered_quantity < 0) as countneg_ordered_quantity,
	(Select Count(*) from products Where stock_level < 0) as countneg_stock_level,
	(Select Count(*) from products Where restocking_lead_time < 0) as countneg_restocking_lead_time,
	(Select Count(*) from products Where sentiment_magnitude < 0) as countneg_sentiment_magnitude,
	(Select Count(*) from products Where ordered_quantity > stock_level) as countexcee_ratio_stock_ordered
from products
LIMIT 1;
--RESULT: NO NEGATIVE VALUES COUNTED, DATA IS VALID
--RESULT: 15 PRODUCTS HAVE MORE ORDERS THAN STOCK ITEMS

/*	End of Query to Check non-negativity constraints and consistency of the product table	*/


/**************************************/


/* 	Query to Add Primary Key to products table	*/

Alter table products
ADD PRIMARY KEY (product_sku);

/*	End of Query to Add Primary Key to products table	*/


/**************************************/
/**************************************/
/**************************************/
/**************************************/
/**************************************/


/* 	Query to create sales_report table
The description of all data cleaning for a particular column preceeds its select statement clause.
There are NO Duplicates
Datatypes have been assigned accordingly
*/

drop table if exists sales_report;

CREATE TABLE sales_report as
(Select
	/* productsku
	Primary Key
	No duplicates
	No trimming necessary (no spaces)
	Type: VarChar Nominal
	convert to snakecase
	*/
	productsku as product_sku,

	/* name
	Is not unique
	No Nulls (no conversion nessary)
	MUST TRIM (spaces before and after)
	Type: VarChar Nominal
	convert to snakecase
	*/
	Trim(both from "name") as product_name,

	/* total_ordered
	Type: INT (no decimal, negative)
	No nulls (no conversion necessary)
	No need to trim
	convert to snakecase
	*/
	Cast (total_ordered as INT) as total_ordered,

	/*stocklevel
	Type: INT (no decimal, negative)
	No need to trim
	convert to snakecase
	No nulls (no conversion necessary)
	*/
	CAST (stocklevel AS INT) as stock_level,

	/*restockingleadtime
	Type: INT (no decimal, negative)
	No need to trim
	convert to snakecase
	No nulls (no conversion necessary)
	*/
	CAST(restockingLeadTime as INT) as restocking_lead_time,

	/* sentimentscore
	Type: NUMERIC (HAS BOTH DECIMAL AND NEGATIVE)
	No need to trim
	convert to snakecase
	HAS 1 NULL, SAME AS sentimentmagnitude
 	Range is from -0.2 to 1.0
	*/
	TO_NUMBER	(sentimentscore, 'S9D99') as sentiment_score,

	/* sentimentmagnitude
	Type: NUMERIC (HAS DECIMAL but is not negative)
	No need to trim
	convert to snakecase
	HAS 1 NULL SAME AS sentimentscore
 	Range is from 0 to 2.0
	*/
	TO_NUMBER	(sentimentmagnitude, '9D99') as sentiment_magnitude,
 
 	/* ratio
 	Type: numeric (no negatives)
 	No need to trim
 	Null values mean no products ordered and none in stock
 	0 Values mean no products ordered but units in stock
 	*/
 	Cast (ratio as numeric) as ratio_of_stock_ordered
 	
from zimport_sales_report);

/*	End of QUERY TO CREATE sales_report TABLE	*/


/**************************************/


/*	Query to Check non-negativity and range constraints of the sales_report table	*/

Select 
	(Select Count(*) from sales_report Where total_ordered < 0) as countneg_total_ordered,
	(Select Count(*) from sales_report Where stock_level < 0) as countneg_stock_level,
	(Select Count(*) from sales_report Where restocking_lead_time < 0) as countneg_restocking_lead_time,
	(Select Count(*) from sales_report Where sentiment_magnitude < 0) as countneg_sentiment_magnitude,
	(Select Count(*) from sales_report Where ratio_of_stock_ordered < 0) as countneg_ratio_of_stock_ordered,
	(Select Count(*) from sales_report Where ratio_of_stock_ordered > 1) as countexceed_ratio_of_stock_ordered
from sales_report
LIMIT 1;
--RESULT: NO NEGATIVE VALUES COUNTED, DATA IS VALID
--RESULT: 2 values where there were more ordered than stock available

/*	End of Query to Check non-negativity and range constraints of the sales_report table	*/


/**************************************/


/* 	Query to Add Primary Key to sales_report table	*/

Alter table sales_report
ADD PRIMARY KEY (product_sku);

/*	End of Query to Add Primary Key to sales_report table	*/



/**************************************/
/**************************************/
/**************************************/
/**************************************/
/**************************************/


/* 	Query to create sales_by_sku table
The description of all data cleaning for a particular column preceeds its select statement clause.
There are NO Duplicates
Datatypes have been assigned accordingly
*/

drop table if exists sales_by_sku;

CREATE TABLE sales_by_sku as
(Select	
	/* productsku
	Primary Key
	No duplicates
	No trimming necessary (no spaces)
	Type: VarChar Nominal
	convert to snakecase
	*/
	productsku as product_sku,
 
 	/* total_ordered
	Type: INT (no decimal, negative)
	No nulls (no conversion necessary)
	No need to trim
	convert to snakecase
	*/
	Cast (total_ordered as INT) as total_ordered

from zimport_sales_by_sku);

/*	End of QUERY TO CREATE sales_report TABLE	*/


/**************************************/


/*	Query to Check non-negativity constraints of the sales_by_sku table	*/

Select 
	(Select Count(*) from sales_by_sku Where total_ordered < 0) as countneg_total_ordered
from sales_by_sku
LIMIT 1;
--RESULT: NO NEGATIVE VALUES COUNTED, DATA IS VALID

/*	End of Query to Check non-negativity constraints of the sales_by_sku table	*/


/**************************************/


/* 	Query to Add Primary Key to sales_by_sku table	*/

Alter table sales_by_sku
ADD PRIMARY KEY (product_sku);

/*	End of Query to Add Primary Key to sales_by_sku table	*/

/**************************************/
/**************************************/
/**************************************/
/**************************************/
/**************************************/

/* 	Query to create analytics table
The description of all data cleaning for a particular column preceeds its select statement clause.
There are duplicates but not addressed in this query
Datatypes have been assigned accordingly
*/

drop table if exists analytics;

CREATE TABLE analytics as
(Select	
 
 	/*visitnumber
 	column to snakecase
 	Type: Int (no decimal, negative)
  	no nulls (no conversion necessary)
	No trimming needed
 	*/
 	Cast(visitnumber AS INT) as visit_number,
 
 
 	/*visitid
 	column to snakecase
 	Type: VarChar Nominal
  	no nulls (no conversion necessary)
	No trimming needed
 	All Numbers
 	Not unique
 	*/
 	visitid as visit_id,
 
 
 	/*visitstarttime
 	column to snakecase
	Type: VarChar Nominal
  	no nulls (no conversion necessary)
	No trimming needed
 	*/
 	visitstarttime as visit_start_time,
 
  	/*date
	Type: VarChar Nominal
 	no nulls (no conversion necessary)
	No trimming needed
	Fixed length
	Type: Date
 	Date range is less than that of all_sessions
 	*/
 	to_date("date", 'YYYYMMDD') as date,
 
  	/*fullvisitorid
 	column to snakecase
	Type: VarChar Nominal
  	no nulls (no conversion necessary)
	No trimming needed
 	is not unique
 	*/
 	fullVisitorId as full_visitor_id,
 
   	/*userid
	empty column, will not include
 	*/
 	
	/*Channelgrouping
	column to snakecase
	No trimming needed
	Type: VarChar and nominal
	Contains 8 categories, same as all_sessions, but includes (other) and 'social'
	no nulls (no conversion necessary)
	*/
	channelGrouping  as channel_grouping,
 
 	/*socialengagementtype
 	Contains the same value: no social engagement
 	will drop from table
 	*/
 
  	/*units_sold
 	column to snakecase
	Type: Int (no decimal, negative)
  	Null means 0, but keep as null
	No trimming needed
 	*/
 	CAST(units_sold as INT) as units_sold,
 
 	/* pageViews
	column to snakecase
	72 nulls, 0 equals null but keep as null
	Type: INT (No decimals, negatives)
	No trimming needed
	*/
	cast(pageviews as int) as page_views,
 
 	/*timeOnSite
	column to snakecase
	Cast to INT as unit of time cannot be determined
	0 equals null but keep as null
	No trimming needed
	*/
	cast(timeOnSite as int) as time_on_site,
 
 	/*bounces
	Type: Boolean
  	no nulls (no conversion necessary)
	No trimming needed
	*/
	cast(bounces as Bool) as bounces,
 
  	/*revenue
	Cast to INT 
	Type: Money (No decimal, Negative) Must be divided by 1,000,000
  	Has nulls, missing data
	No trimming needed
	*/
	CAST((CAST(revenue AS NUMERIC) /1000000) AS MONEY) as revenue,
 
 	/*unit_price
	Cast to INT 
	Type: Money (No decimal, Negative) Must be divided by 1,000,000
  	Has values of zero for null
	No trimming needed
	*/
	CAST((CAST(unit_price AS NUMERIC) /1000000) AS MONEY) as unit_price

from zimport_analytics);

/*	End of QUERY TO CREATE analytics TABLE	*/


/**************************************/


/*	Query to Check non-negativity constraints of the analytics table	*/

Select 
	(Select Count(*) from analytics Where visit_number < 0) as countneg_visit_num,
	(Select Count(*) from analytics Where units_sold < 0) as countneg_units_sold,
	(Select Count(*) from analytics Where page_views < 0) as countneg_page_views,
	(Select Count(*) from analytics Where time_on_site < 0) as countneg_time_on_site,
	(Select Count(*) from analytics Where revenue < '0') as countneg_revenue,
	(Select Count(*) from analytics Where unit_price < '0') as countneg_unit_price
from analytics
LIMIT 1;
--RESULT: One NEGATIVE VALUE COUNTED in units_sold

Delete from analytics
Where units_sold < 0;
--RESULT: ONE RECORD DELETED, DATA IS VALID

/*	End of Query to Check non-negativity constraints of the analytics table	*/


/**************************************/


/* 	Query to Add Primary Key to analytics table	*/

-- Alter table analytics
-- ADD PRIMARY KEY (visit_number);

/*	End of Query to Add Primary Key to analytics table	*/

/**************************************/
/**************************************/
/**************************************/
/**************************************/
/**************************************/








