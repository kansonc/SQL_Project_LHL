What are your risk areas? Identify and describe them.

## The products and sales_report are very similar.
## The sales_by_SKU can essentially be added to the products table
## The Analytics table was not useable, even data like unit_price did not align with product price in all_sessions
## Business insights based on 81 transactions is not reliable

QA Process:
Describe your QA process and include the SQL queries used to execute it.


##First I calculated the table of summary statistics
##Then I checked for trimming or change of lowercase/uppercase
##then I checked for specific characters, such as negative and decimals.


Below, provide the SQL queries you used to clean your data.

1. QUERYS TO PROVIDE SUMMARY STATISTICS AND ASSESSMENTS

/*	Query to assess the effect of spaces on data
to be used to determine whether trim is needed
to be used to determine whether the case of the text need be changed
Run 63 times manually by inserting the column and table name
*/

--CTE to reduce copy paste of manual input 
with cte_manualinput (col) as (
--manually input the character field
Select transactionID 
--manually input the table name
from zimport_products
)

--Main Query as described above
select 
	(Select Count(col) from cte_manualinput where col like '% %') as count_of_records_with_spaces,
	Count(Distinct Trim(both from col)) as count_distinct_with_trim_samecase,
	Count(Distinct col) as count_distinct_without_trim_samecase,
	Count(Distinct Trim(both from lower(col))) as count_distinct_with_trim_lowercase, 
	Count(Distinct lower(col)) as count_distinct_without_trim_lowercase
	from cte_manualinput
	
--Assess Cleaning necessesities with given results

--Found 4 records where trim needed to be applied to the name column of the products table and sales_report
--No need to change to uppercase or lowercase.

/* 	End of Query to assess the effect of spaces on data */


/**************************************/
/**************************************/
/**************************************/
/**************************************/
/**************************************/




/****START OF QUERRY CALCULATE "varchar_stats"******/

/*THIS QUERY IS USED TO COMPUTE LIMITED SUMMARY STATISTICS
FOR EACH COLUMN OF EACH TABLE IMPORTED TO BE USED FOR 
DETERMINING CLEANING STRATEGIES AND INPUT THESE RESULTS 
INTO A TABLE CALLED zdata_varchar_stats.

IT CONTAINS:
2 USER DEFINED TABLE FUNCTIONS
1 MAIN ACTION (DO STATEMENT)
*/

/************FUNCTION START**************************/

/* Function that takes one column from one table as varchar paramenters, 
then executes an SQL statement with the varchar values converted into 
identifiers and returns a single col (column) of values */
CREATE OR REPLACE FUNCTION varchar_col (tablename1 varchar, columnname1 varchar)
	
	RETURNS TABLE (col varchar) 
	
	AS 		
	$$ 		
	BEGIN 		
	RETURN QUERY

	execute format('SELECT %I as "value" FROM %I', columnname1, tablename1);

	end;     $$		LANGUAGE plpgsql;


/************FUNCTION BREAK**************************/


/* Function that returns a single record of summary statistics */
CREATE OR REPLACE FUNCTION calc_varchar_by_column (tablename2 varchar, columnname2 varchar)
	
	RETURNS TABLE 
		--This is the same format as the zdata table below
		(TableName varchar,
		ColumnName varchar,
		TableCount bigint,
		CountNotNull bigint,
		CountDistinct bigint,
		Nullcount bigint,
		MaxLength int,
		MinLength int,
		CountOnlyNum bigint,
		CountwDec bigint,
		CountwNeg bigint,
		CountOnlytext bigint) 
	
	AS 
	$$ 
	BEGIN 
	RETURN QUERY
	
		--CTE to obtain column values
		With settable as(
			Select varchar_col (tablename2, columnname2)
		),

		--CTE to append tablename and column name to the cte above
		cte(tablename3, columnname3, col) as (

			Select 	
				--Manually input table name as value
				tablename2,
				--Manually input column name as value
				columnname2,
				--Manually input column name as identifier to return values within
				settable.*
			from settable
		)

		--CTE TO COMPUTE LIMITED SUMMARY STATISTICS USED FOR CLEANING
		Select 	
					tablename3 as tablename,
					columnname3 as columnname,
					--Count the number of rows in the table
					Count(*) as TableCount,
					--Count the number of values within column that are not null 
					Count(col) as CountNotNull,
					--Count the number of distinct values within column
					Count(Distinct col) as CountDistinct,
					--Count the number of Nulls within column
					Sum(CASE WHEN col  is NULL Then 1 Else 0 End) as Nullcount,
					--Retrieve the longest value length
					Max(Length(col)) as MaxLength,
					--Retrivee the shortest value lenth
					Min(Length(col)) as MinLength,
					--Count records containing numbers only
					(Select Count (*) from cte Where trim(both from col) similar to '[0-9]+') as CountOnlyNum,
					--Count records containing a period or decimal character 
					(Select Count (*) from cte Where trim(both from col) similar to '%.%') as CountwDec,
					--Count records containing numbers only
					(Select Count (*) from cte Where trim(both from col) similar to '-%') as CountwNeg,
					--Count records contraining text only
					(Select Count (*) from cte Where trim(both from col) similar to '[^0-9]+') as CountOnlytext	
		From 	cte 
		Group by	tablename, columnname;
		
		--End of RETURN QUERY

	end;     
	$$
	LANGUAGE plpgsql;

/************FUNCTION END**************************/

/************MAIN ACTION START*********************/

/* This set of actions first creates a table to 
store the statistics calculated with the functions above,
then with a filtered table of columns and tables names to calculate
the varchar stats for, it iterates through each record (column name 
and table name) and calls the calculation function.  Lastly, it 
takes the single record query from the calculation function and 
inserts it into the temporary table.  

The create view at the end stores the results of 
the temporary table to allow the table, which takes more than 
10 minutes to successfully query and create, to be referenced 
in other sessions.*/

Do
$$
Declare
colname varchar := '';
tblname varchar := '';
rec record;

BEGIN 

CREATE TABLE zdata_varchar_stats
		(TableName varchar,
		ColumnName varchar,
		TableCount bigint,
		CountNotNull bigint,
		CountDistinct bigint,
		Nullcount bigint,
		MaxLength int,
		MinLength int,
		CountOnlyNum bigint,
		CountwDec bigint,
		CountwNeg bigint,
		CountOnlytext bigint);

/*Loop through each record, which contains a single table name
and a single column name from the system metadata table*/
for rec in 

	(SELECT	table_name, column_name
	FROM	information_schema.columns 
	WHERE	table_schema = 'public'
	AND table_name like 'zimport%'
	Order By table_name)
	
	LOOP

--Assign the column name and table name to the declared variables
colname = rec.column_name;
tblname = rec.table_name;


Insert into zdata_varchar_stats
--a single record of stats by table name and column name
Select * from calc_varchar_by_column (tblname, colname);

END LOOP;


/* By the end of the loop, the zdata_varchar_stats table 
includes all of the varchar stats (63 rows) that can be 
used to analyze the data before converting to new 
data types */
);

end;     
$$
LANGUAGE plpgsql;

/************MAIN ACTION END*********************/


/****END OF QUERRY CALCULATE "varchar_stats"******/

/* Results of this table guided the clean tables queries */


/**************************************/
/**************************************/
/**************************************/
/**************************************/
/**************************************/



QA DECIMALS

SELECT tablename, columnname, tablecount, countwdec 
FROM zdata_varchar_stats
WHERE countwdec > 0

--Result: no integer variables have decimals




/**************************************/
/**************************************/
/**************************************/
/**************************************/
/**************************************/



QA NEGATIVES

SELECT 		tablename, 
			columnname, 
			tablecount, 
			countnotnull, 
			countonlynum, 
		--How many non null records contain text
			countnotnull - countonlynum as diff_of
			
FROM 		zdata_varchar_stats

WHERE 	--Filter table to include which columnnames have more than 1 record with a number
			countonlynum > 0
		--Filter out any columnnames that would not be data type integer after data clean
	AND 	lower(columnname) not in ('sku', 
									  'productsku', 
									  'visitid',
									  'fullvisitorid','transactions')
		--Filter for columnnames that have records containing text
	AND 	countnotnull - countonlynum > 0

--Found 1 record to be deleted where there is a negative number of units_sold

