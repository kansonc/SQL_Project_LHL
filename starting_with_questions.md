Answer the following questions and provide the SQL queries used to find the answer.

    
/**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**/


/*SQL Queries: */

/* Query for Question 1
Highest level of transactions can be defined by 
Either the sum of total_transaction_revenue
Or by the number of transactions that occurred

OUTPUT QUERY joins all 4 possible interpretations

Outliers:
No outliers will be removed from the quantitative column
Because query is not based on measure of central tendency

Deduping:
No duplicates as all_sessions table has already been cleaned

Cleaning:
Must cast datatype for the quantitative column
Must filter all_session table by transaction is true
*/

--Store query as view
Create or replace View q1 as 
	(

	-- 	Define the working table and variables
	With temp_q1 as (
		Select 	country,
				city,
				transactions,
				Cast(total_transaction_revenue as numeric)
		FROM	all_sessions
		Where 	transactions is true
		),

	--Rank ordered city by sum of revenue
	cte_q1a AS (
		SELECT 		country,
					city,
					SUM(total_transaction_revenue) as sum_total_transaction_revenue
		FROM 		temp_q1

		--Exclude cities that are not defined
		Where 		city not like '(not set)'
		group by 	country, city
		Order By 	sum_total_transaction_revenue DESC
		),

	--Rank ordered city by count of transactions
	cte_q1b as (
		SELECT 		country,
					city,
					COUNT(total_transaction_revenue) as count_total_transaction_revenue
		FROM 		temp_q1

		--Exclude cities that are not defined
		Where 		city not like '(not set)'
		group by 	country, city
		Order By 	count_total_transaction_revenue DESC
		),

	--Rank ordered country by sum of revenue
	cte_q1c AS (
		SELECT 		country,
					SUM(total_transaction_revenue) as sum_total_transaction_revenue
		FROM 		temp_q1
		group by 	country
		Order By 	sum_total_transaction_revenue DESC
		),

	--Rank ordered country by count of transactions
	cte_q1d as (
		SELECT 		country,
					COUNT(total_transaction_revenue) as count_total_transaction_revenue
		FROM 		temp_q1
		group by 	country
		Order By 	count_total_transaction_revenue DESC
		)
	
	Select 	'Highest Sum of Transactions by City in Dollars' as "Category",
			row_number() Over() as ranking,
			cte_q1a.country as country,
			cte_q1a.city as city,
			cte_q1a.sum_total_transaction_revenue as "Sum_Dollars/Num_Counts"
	FROM	cte_q1a

	UNION ALL

	Select 	'Highest Number of Transactions by City in Counts',
			row_number() Over() as ranking,
			cte_q1b.country as country,
			cte_q1b.city as city,
			cte_q1b.count_total_transaction_revenue as "sum/count"
	FROM	cte_q1b

	UNION ALL

	Select 	'Highest Sum of Transactions by Country in Dollars',
			row_number() Over() as ranking,
			cte_q1c.country as country,
			null as city,
			cte_q1c.sum_total_transaction_revenue as "sum/count"
	FROM	cte_q1c

	UNION ALL

	Select 	'Highest Number of Transactions by Country in Counts',
			row_number() Over() as ranking,
			cte_q1d.country as country,
			null as city,
			cte_q1d.count_total_transaction_revenue as "sum/count"
	FROM	cte_q1d
);
	
Select * from q1;


/* Answer:

Category						ranking	country		city		Sum_Dollars/Num_Counts
Highest Sum of Transactions by City in Dollars		1	United States	San Francisco	1564.32
Highest Sum of Transactions by City in Dollars		2	United States	Sunnyvale	992.23
Highest Sum of Transactions by City in Dollars		3	United States	Atlanta		854.44
Highest Sum of Transactions by City in Dollars		4	United States	Palo Alto	608
Highest Sum of Transactions by City in Dollars		5	Israel		Tel Aviv-Yafo	602
Highest Sum of Transactions by City in Dollars		6	United States	New York	530.36
Highest Sum of Transactions by City in Dollars		7	United States	Mountain View	483.36
Highest Sum of Transactions by City in Dollars		8	United States	Los Angeles	479.48
Highest Sum of Transactions by City in Dollars		9	United States	Chicago		449.52
Highest Sum of Transactions by City in Dollars		10	United States	Seattle		358
Highest Sum of Transactions by City in Dollars		11	Australia	Sydney		358
Highest Sum of Transactions by City in Dollars		12	United States	San Jose	262.38
Highest Sum of Transactions by City in Dollars		13	United States	Austin		157.78
Highest Sum of Transactions by City in Dollars		14	United States	Nashville	157
Highest Sum of Transactions by City in Dollars		15	United States	San Bruno	103.77
Highest Sum of Transactions by City in Dollars		16	Canada		Toronto		82.16
Highest Sum of Transactions by City in Dollars		17	Canada		New York	67.99
Highest Sum of Transactions by City in Dollars		18	United States	Houston		38.98
Highest Sum of Transactions by City in Dollars		19	United States	Columbus	21.99
Highest Sum of Transactions by City in Dollars		20	Switzerland	Zurich		16.99
Highest Number of Transactions by City in Counts	1	United States	San Francisco	12
Highest Number of Transactions by City in Counts	2	United States	New York	8
Highest Number of Transactions by City in Counts	3	United States	Mountain View	8
Highest Number of Transactions by City in Counts	4	United States	Sunnyvale	4
Highest Number of Transactions by City in Counts	5	United States	Palo Alto	3
Highest Number of Transactions by City in Counts	6	United States	Chicago		3
Highest Number of Transactions by City in Counts	7	United States	Atlanta		2
Highest Number of Transactions by City in Counts	8	United States	Austin		2
Highest Number of Transactions by City in Counts	9	United States	San Jose	2
Highest Number of Transactions by City in Counts	10	United States	Los Angeles	2
Highest Number of Transactions by City in Counts	11	Switzerland	Zurich		1
Highest Number of Transactions by City in Counts	12	Canada		New York	1
Highest Number of Transactions by City in Counts	13	Canada		Toronto		1
Highest Number of Transactions by City in Counts	14	United States	Columbus	1
Highest Number of Transactions by City in Counts	15	Israel		Tel Aviv-Yafo	1
Highest Number of Transactions by City in Counts	16	United States	Seattle		1
Highest Number of Transactions by City in Counts	17	United States	Houston		1
Highest Number of Transactions by City in Counts	18	Australia	Sydney		1
Highest Number of Transactions by City in Counts	19	United States	Nashville	1
Highest Number of Transactions by City in Counts	20	United States	San Bruno	1
Highest Sum of Transactions by Country in Dollars	1	United States	NULL		13154.17
Highest Sum of Transactions by Country in Dollars	2	Israel		NULL		602
Highest Sum of Transactions by Country in Dollars	3	Australia	NULL		358
Highest Sum of Transactions by Country in Dollars	4	Canada		NULL		150.15
Highest Sum of Transactions by Country in Dollars	5	Switzerland	NULL		16.99
Highest Number of Transactions by Country in Counts	1	United States	NULL		76
Highest Number of Transactions by Country in Counts	2	Canada		NULL		2
Highest Number of Transactions by Country in Counts	3	Israel		NULL		1
Highest Number of Transactions by Country in Counts	4	Switzerland	NULL		1
Highest Number of Transactions by Country in Counts	5	Australia	NULL		1

*/


/**Question 2: What is the average number of products ordered from visitors in each city and country?**/


/*SQL Queries:*/

/* Query for Question 3
Returning the Avg number of products, 
which is the number of distinct products, 
By City and Country
*/

--Store query as view

drop view q2;

Create or replace View q2 as 
	(
	--Return Avg Number of Products purchased by visitors in each country
	with cte_country as (
	Select	Country,
			AVG(count_products_purchased) as avg_num_products_purchased
	FROM (
		Select 
				Country,
				full_visitor_id,
				Count(distinct product_sku) as count_products_purchased
		FROM 	all_sessions
		Where 	Country not like '(not_set)'
		Group by 1,2
		) as temp
	GROUP BY	country
	HAVING 		
				--Avg to be calculated when there's more than one visitor from that country
				Count(distinct full_visitor_id) > 1
				--Avg to be calculated when there's at least 1 visitor that bought multiple items (Avg is 1.00~ otherwise) 
		AND		MAX(count_products_purchased) > 1
	Order By	avg_num_products_purchased DESC),

	cte_city as(
	--Return Avg Number of Products purchased by visitors in each city
	Select	city,
			AVG(count_products_purchased) as avg_num_products_purchased
	FROM (
		Select 
				city,
				full_visitor_id,
				Count(distinct product_sku) as count_products_purchased
		FROM 	all_sessions
		Where 	city not like '(not_set)'
		Group by 1,2
		) as temp
	GROUP BY	city
	HAVING 		
				--Avg to be calculated when there's more than one visitor from that country
				Count(distinct full_visitor_id) > 1
				--Avg to be calculated when there's at least 1 visitor that bought multiple items (Avg is 1.00~ otherwise) 
		AND		MAX(count_products_purchased) > 1
	Order By	avg_num_products_purchased DESC)

	Select 'Country' as "city/country",
			country as "place",
			Round(avg_num_products_purchased, 2) as avg_num_products_purchased
	FROM	cte_country

	UNION ALL

	Select 'City',
			city as "place",
			Round(avg_num_products_purchased, 2) as avg_num_products_purchased
	FROM	cte_city
);

Select * from q2;
		

	







Answer:

City/country	place	avg_num_products_purchased
City	Salem		1.59
City	Santa Fe	1.5
City	Culiacan	1.5
City	Pozuelo de Alarcon	1.33
City	Norfolk	1.33
City	Dubai	1.29
City	Buenos Aires	1.29
City	Vienna	1.25
City	Rome	1.25
City	Orlando	1.2
City	Quebec City	1.17
City	Bangkok	1.16
City	La Victoria	1.14
City	Portland	1.14
City	Boston	1.13
City	Kolkata	1.13
City	Fremont	1.13
City	Singapore	1.12
City	Chicago	1.11
City	Kiev	1.11
City	Stockholm	1.11
City	Hong Kong	1.1
City	Austin	1.1
City	Dallas	1.1
City	Sao Paulo	1.09
City	Santa Clara	1.09
City	Toronto	1.09
City	Philadelphia	1.08
City	Paris	1.08
City	Mountain View	1.08
City	New Delhi	1.08
City	San Jose	1.08
City	Los Angeles	1.08
City	Milan	1.08
City	New York	1.07
City	San Francisco	1.07
City	Eau Claire	1.07
City	Munich	1.07
City	Sunnyvale	1.07
City	Atlanta	1.06
City	Kuala Lumpur	1.06
City	Mumbai	1.06
City	London	1.06
City	Moscow	1.06
City	Melbourne	1.06
City	Irvine	1.05
City	Bogota	1.05
City	Madrid	1.05
City	Kirkland	1.05
City	Pune	1.05
City	San Bruno	1.04
City	Tel Aviv-Yafo	1.04
City	Zurich	1.04
City	Ann Arbor	1.04
City	Jakarta	1.04
City	San Diego	1.04
City	Pittsburgh	1.04
City	Dublin	1.04
City	Hyderabad	1.04
City	Seattle	1.03
City	Cambridge	1.03
City	Bengaluru	1.03
City	Sydney	1.03
City	Minato	1.03
City	Montreal	1.03
City	Washington	1.03
City	Palo Alto	1.02
City	Chennai	1.01
Country	Nepal	1.33
Country	Bolivia	1.33
Country	Bulgaria	1.25
Country	Cambodia	1.2
Country	Kuwait	1.2
Country	Ecuador	1.17
Country	Nigeria	1.14
Country	Argentina	1.13
Country	Hong Kong	1.12
Country	Croatia	1.12
Country	United Arab Emirates	1.12
Country	Malaysia	1.1
Country	Thailand	1.09
Country	Estonia	1.09
Country	Taiwan	1.09
Country	Ukraine	1.09
Country	Mexico	1.08
Country	Uruguay	1.08
Country	Egypt	1.08
Country	Singapore	1.08
Country	Austria	1.08
Country	Peru	1.08
Country	Switzerland	1.08
Country	United States	1.07
Country	Serbia	1.07
Country	Panama	1.07
Country	Israel	1.07
Country	Bangladesh	1.07
Country	Czechia	1.07
Country	Lithuania	1.06
Country	France	1.06
Country	Canada	1.06
Country	Sweden	1.06
Country	Brazil	1.05
Country	Ireland	1.05
Country	Germany	1.05
Country	Italy	1.05
Country	United Kingdom	1.04
Country	Philippines	1.04
Country	New Zealand	1.04
Country	South Africa	1.04
Country	China	1.04
Country	Romania	1.04
Country	Chile	1.04
Country	India	1.04
Country	Spain	1.03
Country	Japan	1.03
Country	Venezuela	1.03
Country	Denmark	1.03
Country	Russia	1.03
Country	Belgium	1.03
Country	Norway	1.03
Country	Australia	1.03
Country	Slovakia	1.03
Country	Netherlands	1.03
Country	Poland	1.02
Country	Indonesia	1.02
Country	Pakistan	1.02
Country	Greece	1.02
Country	South Korea	1.02
Country	Colombia	1.02




**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:

/* Question 3a
Question 3: Is there any pattern in the types (product categories) 
of products ordered from visitors in each city and country?

Because of the limited transactional data of only 82 purchases,
there is not enough to say there is a pattern by country, 
even by month there is no pattern
*/
Select		Country,
			v2_product_category,
			count (v2_product_category) as count_by_category
FROM		all_sessions
Where		country not like '(not set)'
	AND		v2_product_category not like '(not set)'
	AND 	transactions = True
GROUP BY	1,2
--filter out countries with only 1 purchase per category
HAVING		count (v2_product_category) > 1
ORDER BY	3 DESC, 1, 2

/* Question 3b
Question 3: Is there any pattern in the types (product categories) 
of products ordered from visitors in each city and country?

Because of the limited transactional data of only 82 purchases,
there is not enough to say there is a pattern by city
even by month there is no pattern
*/
Select		City,
			v2_product_category,
			count (v2_product_category) as count_by_category
FROM		all_sessions
Where		city not like '(not set)'
	AND		v2_product_category not like '(not set)'
	AND 	transactions = True
GROUP BY	1,2
--filter out cities with only 1 purchase per category
HAVING		count (v2_product_category) > 1
ORDER BY	3 DESC, 1, 2

Answer: 
/* Q3a No patterns could be found at the country level,
though the categories "Nest" and "Apparel" categories
have the highest amount of purchases.

country	v2_product_category	count_by_category
United States	Home/Nest/Nest-USA/	17
United States	Nest-USA	8
United States	Apparel	6
United States	Home/Apparel/Men's/Men's-T-Shirts/	5
United States	Home/Apparel/Kid's/Kid's-Infant/	3
United States	Home/Apparel/Women's/Women's-T-Shirts/	3
United States	Bags	2
United States	Electronics	2
United States	Home/Accessories/Fun/	2
United States	Home/Bags/	2
United States	Home/Drinkware/	2
United States	Home/Shop by Brand/Google/	2
United States	Lifestyle	2
United States	Waze	2
*/

/* Q3a No patterns could be found at the city level 

city	v2_product_category	count_by_category
San Francisco	Home/Nest/Nest-USA/	3
Mountain View	Apparel	2
Palo Alto	Home/Nest/Nest-USA/	2
*/



**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

/* Q4a

Question 4: What is the top-selling product from each city/country? 
Can we find any pattern worthy of noting in the products sold?

Q4a query will evalute for country by number of distinct products sold (not quantity)

*/

Select		country,
			v2_product_name,
			count(v2_product_name)

FROM 		all_sessions
WHERE		transactions = True
GROUP BY 	1,2
ORDER BY 	3 DESC

/* Q4b

Question 4: What is the top-selling product from each city/country? 
Can we find any pattern worthy of noting in the products sold?

Q4b query will evalute for city by number of distinct products sold (not quantity)

*/

Select		city,
			v2_product_name,
			count(v2_product_name)

FROM 		all_sessions
WHERE		transactions = True
	AND		city not like '(not set)'
GROUP BY 	1,2
ORDER BY 	3 DESC


/* Q4c

Question 4: What is the top-selling product from each city/country? 
Can we find any pattern worthy of noting in the products sold?

Q4c query will evalute for country by total revenue by products sold

*/
Select 	country, 
		v2_product_name,
		count_of_transactions,
		sum_of_transactions
from (

Select		country,
			v2_product_name,
			count(v2_product_name) as count_of_transactions,
			SUM(total_transaction_revenue) as sum_of_transactions,
			row_number() Over(partition by country order by SUM(total_transaction_revenue) DESC) as product_rank
FROM 		all_sessions
WHERE		transactions = True
	AND		country not like '(not set)'
GROUP BY 	1,2
ORDER BY 	4 DESC ) as temp
			   
Where product_rank = 1



/* Q4d

Question 4: What is the top-selling product from each city/country? 
Can we find any pattern worthy of noting in the products sold?

Q4d query will evalute for city by total revenue by products sold

*/
Select 	city, 
		v2_product_name,
		count_of_transactions,
		sum_of_transactions
from (

Select		city,
			v2_product_name,
			count(v2_product_name) as count_of_transactions,
			SUM(total_transaction_revenue) as sum_of_transactions,
			row_number() Over(partition by city order by SUM(total_transaction_revenue) DESC) as product_rank
FROM 		all_sessions
WHERE		transactions = True
	AND		city not like '(not set)'
GROUP BY 	1,2
ORDER BY 	4 DESC ) as temp
			   
Where product_rank = 1





Answer:

/* Q4a most transactions of a product by country (not quantity) include many of the home products by Nest */

country	v2_product_name	count
United States	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	7
United States	NestÂ® Cam Outdoor Security Camera - USA	5
United States	NestÂ® Cam Indoor Security Camera - USA	3
United States	NestÂ® Learning Thermostat 3rd Gen-USA	3
United States	Google Sunglasses	2
United States	Google Men's 100% Cotton Short Sleeve Hero Tee Navy	2
United States	NestÂ® Protect Smoke + CO White Wired Alarm-USA	2
United States	NestÂ® Protect Smoke + CO White Battery Alarm-USA	2
United States	Reusable Shopping Bag	2
United States	Google Men's 100% Cotton Short Sleeve Hero Tee Black	2
United States	NestÂ® Learning Thermostat 3rd Gen-USA - White	2
United States	Android Men's Vintage Henley	1
United States	Android Rise 14 oz Mug	1
United States	Android Women's Fleece Hoodie	1
United States	Android Women's Short Sleeve Tri-blend Badge Tee Light Grey	1
United States	Compact Bluetooth Speaker	1
United States	Crunch Noise Dog Toy	1
United States	Google Bongo Cupholder Bluetooth Speaker	1
United States	Google Infant Zip Hood Pink	1
United States	Google Laptop and Cell Phone Stickers	1
United States	Google Men's  Zip Hoodie	1
United States	Google Men's 100% Cotton Short Sleeve Hero Tee White	1
United States	Google Men's Bike Short Sleeve Tee Charcoal	1
United States	Google Men's Short Sleeve Badge Tee Charcoal	1
United States	Google Men's Short Sleeve Hero Tee Heather	1
United States	Google Men's Short Sleeve Performance Badge Tee Pewter	1
United States	Google Men's Vintage Badge Tee Sage	1
United States	Google Men's Vintage Badge Tee White	1
United States	Google Men's Vintage Tank	1
United States	Google Onesie Red/Graphite	1
United States	Google Tri-blend Hoodie Grey	1
United States	Google Women's 3/4 Sleeve Baseball Raglan Heather/Black	1
United States	Google Women's Scoop Neck Tee White	1
United States	Google Women's Short Sleeve Hero Tee Heather	1
United States	Google Women's Short Sleeve Hero Tee White	1
United States	Google Zipper-front Sports Bag	1
United States	Grip Kit Cable Organizer	1
United States	Leatherette Journal	1
United States	NestÂ® Protect Smoke + CO Black Battery Alarm-USA	1
United States	NestÂ® Protect Smoke + CO Black Wired Alarm-USA	1
United States	Rubber Grip Ballpoint Pen 4 Pack	1
United States	SPF-15 Slim & Slender Lip Balm	1
United States	Waterproof Backpack	1
United States	Waze Dress Socks	1
United States	Waze Mobile Phone Vent Mount	1
United States	Waze Mood Original Window Decal	1
United States	Windup Android	1
United States	YouTube Luggage Tag	1
Australia	NestÂ® Cam Indoor Security Camera - USA	1
United States	YouTube Men's Short Sleeve Hero Tee Charcoal	1
Canada	Google Men's  Zip Hoodie	1
Canada	Google Men's 3/4 Sleeve Raglan Henley Grey	1
Israel	NestÂ® Protect Smoke + CO Black Wired Alarm-USA	1
Switzerland	YouTube Men's 3/4 Sleeve Henley	1
United States	20 oz Stainless Steel Insulated Tumbler	1
United States	22 oz YouTube Bottle Infuser	1
United States	26 oz Double Wall Insulated Bottle	1
United States	Android 17oz Stainless Steel Sport Bottle	1
United States	Android Infant Short Sleeve Tee Pewter	1
United States	Android Men's Long Sleeve Badge Crew Tee Heather	1



/* Q4b - most transactions of a product by city(not quantity) show there is no pattern */
city	v2_product_name	count
Atlanta	Google Men's Short Sleeve Hero Tee Heather	1
Atlanta	Reusable Shopping Bag	1
Austin	Google Men's 100% Cotton Short Sleeve Hero Tee Black	1
Austin	Google Men's 100% Cotton Short Sleeve Hero Tee Navy	1
Chicago	Google Sunglasses	1
Chicago	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	1
Chicago	Rubber Grip Ballpoint Pen 4 Pack	1
Columbus	Google Men's Short Sleeve Badge Tee Charcoal	1
Houston	26 oz Double Wall Insulated Bottle	1
Los Angeles	Google Women's Short Sleeve Hero Tee White	1
Los Angeles	NestÂ® Cam Outdoor Security Camera - USA	1
Mountain View	Android Infant Short Sleeve Tee Pewter	1
Mountain View	Android Women's Fleece Hoodie	1
Mountain View	Google Men's Bike Short Sleeve Tee Charcoal	1
Mountain View	Google Men's Vintage Badge Tee Sage	1
Mountain View	Grip Kit Cable Organizer	1
Mountain View	NestÂ® Learning Thermostat 3rd Gen-USA	1
Mountain View	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	1
Mountain View	Waze Mobile Phone Vent Mount	1
Nashville	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	1
New York	Google Laptop and Cell Phone Stickers	1
New York	Google Men's  Zip Hoodie	1
New York	Google Men's 100% Cotton Short Sleeve Hero Tee White	1
New York	Google Men's Short Sleeve Performance Badge Tee Pewter	1
New York	Google Men's Vintage Tank	1
New York	Google Onesie Red/Graphite	1
New York	NestÂ® Cam Outdoor Security Camera - USA	1
New York	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	1
New York	YouTube Luggage Tag	1
Palo Alto	NestÂ® Cam Outdoor Security Camera - USA	1
Palo Alto	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	1
Palo Alto	NestÂ® Learning Thermostat 3rd Gen-USA - White	1
San Bruno	Google Men's Vintage Badge Tee White	1
San Francisco	20 oz Stainless Steel Insulated Tumbler	1
San Francisco	Android 17oz Stainless Steel Sport Bottle	1
San Francisco	Android Rise 14 oz Mug	1
San Francisco	Google Tri-blend Hoodie Grey	1
San Francisco	Google Women's 3/4 Sleeve Baseball Raglan Heather/Black	1
San Francisco	Google Women's Scoop Neck Tee White	1
San Francisco	NestÂ® Cam Outdoor Security Camera - USA	1
San Francisco	NestÂ® Learning Thermostat 3rd Gen-USA	1
San Francisco	NestÂ® Protect Smoke + CO Black Battery Alarm-USA	1
San Francisco	NestÂ® Protect Smoke + CO White Battery Alarm-USA	1
San Francisco	Waterproof Backpack	1
San Francisco	Windup Android	1
San Jose	Google Men's  Zip Hoodie	1
San Jose	NestÂ® Cam Outdoor Security Camera - USA	1
Seattle	NestÂ® Cam Indoor Security Camera - USA	1
Sunnyvale	Android Men's Vintage Henley	1
Sunnyvale	NestÂ® Cam Indoor Security Camera - USA	1
Sunnyvale	NestÂ® Protect Smoke + CO Black Wired Alarm-USA	1
Sunnyvale	SPF-15 Slim & Slender Lip Balm	1
Sydney	NestÂ® Cam Indoor Security Camera - USA	1
Tel Aviv-Yafo	NestÂ® Protect Smoke + CO Black Wired Alarm-USA	1
Toronto	Google Men's 3/4 Sleeve Raglan Henley Grey	1
Zurich	YouTube Men's 3/4 Sleeve Henley	1


/* Q4c - Product generating the highest revenue by country */
country	v2_product_name	count_of_transactions	sum_of_transactions
United States	Reusable Shopping Bag	2	$1,757.96 
Israel	NestÂ® Protect Smoke + CO Black Wired Alarm-USA	1	$602.00 
Australia	NestÂ® Cam Indoor Security Camera - USA	1	$358.00 
Canada	Google Men's 3/4 Sleeve Raglan Henley Grey	1	$82.16 
Switzerland	YouTube Men's 3/4 Sleeve Henley	1	$16.99 


/* Q4d - Product generating the highest revenue by city */
city	v2_product_name	count_of_transactions	sum_of_transactions
Atlanta	Reusable Shopping Bag	1	$742.48 
Sunnyvale	SPF-15 Slim & Slender Lip Balm	1	$649.24 
Tel Aviv-Yafo	NestÂ® Protect Smoke + CO Black Wired Alarm-USA	1	$602.00 
Los Angeles	NestÂ® Cam Outdoor Security Camera - USA	1	$363.00 
Sydney	NestÂ® Cam Indoor Security Camera - USA	1	$358.00 
Seattle	NestÂ® Cam Indoor Security Camera - USA	1	$358.00 
Chicago	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	1	$306.00 
Palo Alto	NestÂ® Cam Outdoor Security Camera - USA	1	$305.00 
San Francisco	NestÂ® Protect Smoke + CO Black Battery Alarm-USA	1	$301.00 
Nashville	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	1	$157.00 
Mountain View	NestÂ® Learning Thermostat 3rd Gen-USA	1	$156.00 
San Jose	NestÂ® Cam Outdoor Security Camera - USA	1	$154.00 
New York	NestÂ® Learning Thermostat 3rd Gen-USA - Stainless Steel	1	$152.00 
Austin	Google Men's 100% Cotton Short Sleeve Hero Tee Navy	1	$122.00 
San Bruno	Google Men's Vintage Badge Tee White	1	$103.77 
Toronto	Google Men's 3/4 Sleeve Raglan Henley Grey	1	$82.16 
Houston	26 oz Double Wall Insulated Bottle	1	$38.98 
Columbus	Google Men's Short Sleeve Badge Tee Charcoal	1	$21.99 
Zurich	YouTube Men's 3/4 Sleeve Henley	1	$16.99 






**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

/*
Question 5: Can we summarize the impact of revenue generated from each city/country?

Answer: by showing the percentage of revenue a city generates of the country revenue
and by showing the percentage of revenue a country generates of the total revenue
*/

SELECT *,
		sum_city / sum_country as "%_of_city_revenue",
		sum_country / (Select SUM(total_transaction_revenue) 
							  from all_sessions 
							  Where Not (country like 'Canada' and city like 'New York')) as "%_of_country_revenue"
from (
Select 	country,
		city,
		SUM_city,
		SUM(SUM_city) OVER(PARTITION BY country) as SUM_country
FROM 	(
	Select 	country,
			city,
			SUM(total_transaction_revenue) as SUM_city
	from 	all_sessions
	WHERE		transactions = True
		AND		country not like '(not set)'
	--found outlier where there is a mismatch record with new york and canada
		AND		Not (country like 'Canada' and city like 'New York')
	GROUP BY 	1,2) AS TEMP
GROUP BY	1,2,3
) as temp2
ORDER BY 5 DESC
			  



Answer: 

/* The impact by city on the total revenue of the country can be expressed as a percentage.
The impact by country on the total revenue can be expressed as a percentage */



country	city	sum_city	sum_country	%_of_city_revenue	%_of_country_revenue
Canada	Toronto	$82.16 	$82.16 	1	0.005780493
Israel	Tel Aviv-Yafo	$602.00 	$602.00 	1	0.042354636
Switzerland	Zurich	$16.99 	$16.99 	1	0.001195358
Australia	Sydney	$358.00 	$358.00 	1	0.025187641
United States	(not set)	$6,092.56 	$13,154.17 	0.463165673	0.925481872
United States	San Francisco	$1,564.32 	$13,154.17 	0.118921984	0.925481872
United States	Sunnyvale	$992.23 	$13,154.17 	0.075430833	0.925481872
United States	Atlanta	$854.44 	$13,154.17 	0.064955828	0.925481872
United States	Palo Alto	$608.00 	$13,154.17 	0.046221084	0.925481872
United States	New York	$530.36 	$13,154.17 	0.040318773	0.925481872
United States	Mountain View	$483.36 	$13,154.17 	0.036745762	0.925481872
United States	Los Angeles	$479.48 	$13,154.17 	0.036450798	0.925481872
United States	Chicago	$449.52 	$13,154.17 	0.034173194	0.925481872
United States	Seattle	$358.00 	$13,154.17 	0.027215704	0.925481872
United States	San Jose	$262.38 	$13,154.17 	0.019946526	0.925481872
United States	Austin	$157.78 	$13,154.17 	0.011994675	0.925481872
United States	Nashville	$157.00 	$13,154.17 	0.011935379	0.925481872
United States	San Bruno	$103.77 	$13,154.17 	0.007888753	0.925481872
United States	Houston	$38.98 	$13,154.17 	0.002963319	0.925481872
United States	Columbus	$21.99 	$13,154.17 	0.001671713	0.925481872








