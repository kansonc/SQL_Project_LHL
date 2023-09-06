Question 1: How often are categories ranked highest in view count by country?

SQL Queries:

/* Query for my Question 1
How often are categories ranked highest in view count by country?
*/

--Store query as view

drop view my_q1;

Create or replace View my_q1 as 
(
	--Return count of views by category and country
	--where count is greater than 1
	Select	Country,
			v2_product_category,
			Count(*) count_of_views
	FROM	all_sessions
	Where	country not like '(not set)'
		AND	v2_product_category not like '(not set)'
	group by 1,2
	HAVING 	Count(*) > 1
	
);

--Establish the median of all purchases by country
With cte_q1_1 as(
	Select 		country,
				percentile_cont(0.5) WITHIN GROUP (ORDER BY count_of_views) as median_views_country
	from 		my_q1
	GROUP BY	country
	)

--filter for categories above the with counts above the median

,cte_q1_2 as
(
	SELECT 		my_q1.country,
				my_q1.v2_product_category,
				my_q1.count_of_views,
				RANK () OVER (Partition By my_q1.country ORDER by my_q1.country, my_q1.count_of_views desc) as country_category_ranking,
				cte_q1_1.median_views_country
	FROM		my_q1
	Join		cte_q1_1
	ON			my_q1.country = cte_q1_1.country
	--filter count_of_purchases above the median of the country
	WHERE		my_q1.count_of_views >= cte_q1_1.median_views_country

	ORDER BY	country,count_of_views DESC
	)

Select 	v2_product_category,
		country_category_ranking,
		Count(*) AS count_at_ranking
FROM	cte_q1_2
--Filter by categories in the top 3 by country
Where	country_category_ranking <=3

GROUP BY	1,2
--Filter by categories and rankings with counts above 10
HAVING		Count(*) >10
ORDER BY	2, 3 DESC



Answer: 
the product category "Home/Shop by Brand/YouTube/" was the top ranked category in 66 countries.  


v2_product_category			country_category_ranking	count_at_ranking
Home/Shop by Brand/YouTube/		1				66
Home/Apparel/Men's/Men's-T-Shirts/	1				23
Home/Apparel/Men's/Men's-T-Shirts/	2				39
Home/Electronics/			3				11




Question 2: What percentage of records in the all_sessions table were transactions

SQL Queries:

Select Cast((select Count(*)from all_sessions where transactions = true) as numeric)
/ Cast((select count(*) from all_sessions) as numeric) from all_sessions
limit 1


Answer:

0.00535395597858417609%






Question 3:  What total count of products have the same name and have simliar names?

/* My question 3:
What count of products have the same name and have simliar names?
*/

CREATE EXTENSION IF NOT EXISTS pg_trgm;
--This extension evalutes two strings by a trigram


Select 	product_name, 
		common_product_name,
		prod_count + common_prod_count as total_similiar_products
from 
(

	Select	
		--reference
		product_name,
		--reference count
		prod_count,
		--this sets the next "lowername" value as a record 
		Lead (product_name) Over() as common_product_name,
		Lead (prod_count) Over() as common_prod_count,
		--from the trgm extension, this compares sets of 3 characters at a time and outputs a number where 1 is a complete match
		similarity  (product_name, Lead (product_name) Over()) as  sim,
		--from the trgm extension, this compares sets of 3 characters against and consectutive 3 characters in second stringand outputs a number where 1 is a complete match
		word_similarity  (product_name, Lead (product_name) Over()) as  word_sim,
		--from the trgm extension, this is like word_similarity but is delimited by space and outputs a number where 1 is a complete match
		strict_word_similarity  (product_name, Lead (product_name) Over()) as  strict_word_sim
	From(
		Select 	product_name,
				count(product_name) as prod_count
		from products
		group by 1
		order by 1
		) as t
	Order by sim DESC
) as t2
--focus on close matching values
Where sim >= 0.5





Answer:

-- The results of this question could lead to further insights in these
-- similar products, assuming the differences are essentially cosmetic (colour, size, style)


product_name	common_product_name	total_similiar_products
Android Stretch Fit Hat M/L	Android Stretch Fit Hat S/M	2
Women's Short Sleeve Hero Tee Heather	Women's Short Sleeve Hero Tee Red Heather	11
Protect Smoke + CO White Battery Alarm-USA	Protect Smoke + CO White Battery Alarm - CA	3
Protect Smoke + CO White Wired Alarm-USA	Protect Smoke + CO White Wired Alarm - CA	2
Cam Outdoor Security Camera - CA	Cam Outdoor Security Camera - USA	3
Gift Card - $25.00	Gift Card - $250.00	4
Cam Indoor Security Camera - CA	Cam Indoor Security Camera - USA	3
Men's 100% Cotton Short Sleeve Hero Tee Navy	Men's 100% Cotton Short Sleeve Hero Tee Red	13
Android Stretch Fit Hat	Android Stretch Fit Hat Black	6
Men's Short Sleeve Performance Badge Tee Navy	Men's Short Sleeve Performance Badge Tee Pewter	7
Men's 100% Cotton Short Sleeve Hero Tee Red	Men's 100% Cotton Short Sleeve Hero Tee White	13
Android Toddler Short Sleeve T-shirt Pewter	Android Toddler Short Sleeve T-shirt Pink	9
Women's Fleece Hoodie	Women's Fleece Hoodie Black	9
Men's 100% Cotton Short Sleeve Hero Tee Black	Men's 100% Cotton Short Sleeve Hero Tee Navy	14
Men's Vintage Badge Tee Black	Men's Vintage Badge Tee Sage	11
Android Toddler Short Sleeve T-shirt Aqua	Android Toddler Short Sleeve T-shirt Pewter	8
Android Infant Short Sleeve Tee Pewter	Android Infant Short Sleeve Tee Pink	8
Android Men's Short Sleeve Hero Tee Heather	Android Men's Short Sleeve Hero Tee White	8
Women's Short Sleeve Badge Tee Grey	Women's Short Sleeve Badge Tee Navy	10
Men's Vintage Badge Tee Sage	Men's Vintage Badge Tee White	13
Android Youth Short Sleeve T-shirt Aqua	Android Youth Short Sleeve T-shirt Pewter	11
Gift Card - $250.00	Gift Card - $50.00	4
Women's Short Sleeve Hero Tee Grey	Women's Short Sleeve Hero Tee Heather	10
Men's Short Sleeve Performance Badge Tee Charcoal	Men's Short Sleeve Performance Badge Tee Navy	9
Men's Short Sleeve Performance Badge Tee Black	Men's Short Sleeve Performance Badge Tee Charcoal	13
Leather Journal	Leather Journal-Black	4
Women's Short Sleeve Tri-blend Badge Tee Charcoal	Women's Short Sleeve Tri-blend Badge Tee Grey	12
Women's Quilted Insulated Vest Black	Women's Quilted Insulated Vest White	9
Men's Airflow 1/4 Zip Pullover Black	Men's Airflow 1/4 Zip Pullover Lapis	14
Women's 1/4 Zip Performance Pullover Black	Women's 1/4 Zip Performance Pullover Two-Tone Blue	8
Women's Short Sleeve Performance Tee Charcoal	Women's Short Sleeve Performance Tee Pewter	11
Android Stretch Fit Hat Black	Android Stretch Fit Hat M/L	2
Toddler Short Sleeve Tee Red	Toddler Short Sleeve Tee White	14
Infant Short Sleeve Tee Green	Infant Short Sleeve Tee Red	5
Youth Short Sleeve Tee Red	Youth Short Sleeve Tee White	16
Women's Scoop Neck Tee Black	Women's Scoop Neck Tee White	12
Gift Card- $100.00	Gift Card - $25.00	4
Men's Short Sleeve Hero Tee  White	Men's Short Sleeve Hero Tee Black	7
Women's Short Sleeve Hero Tee Sky Blue	Women's Short Sleeve Hero Tee White	12
Men's Vintage Tank	Men's Vintage Tee	10
Women's Short Sleeve Hero Tee Charcoal	Women's Short Sleeve Hero Tee Grey	10
Women's Short Sleeve Hero Tee Black	Women's Short Sleeve Hero Tee Charcoal	11
Men's Short Sleeve Hero Tee Charcoal	Men's Short Sleeve Hero Tee Heather	17
Infant Short Sleeve Tee Red	Infant Short Sleeve Tee Royal Blue	8
Android Men's Vintage Henley	Android Men's Vintage Tank	12
Yoga Mat	Yoga Mat Blue	4
Women's Vintage Hero Tee Platinum	Women's Vintage Hero Tee White	12
Women's Convertible Vest-Jacket Black	Women's Convertible Vest-Jacket Sea Foam Green	14
Men's Short Sleeve Hero Tee Black	Men's Short Sleeve Hero Tee Charcoal	19
Android Men's Short Sleeve Hero Tee White	Android Men's Short Sleeve Tri-blend Hero Tee Grey	5
Leather Journal-Black	Leather Journal-Brown	4
Learning Thermostat 3rd Gen-USA - Stainless Steel	Learning Thermostat 3rd Gen-USA - White	4
Women's Vintage Hero Tee Black	Women's Vintage Hero Tee Lavender	12
Women's Short Sleeve Hero Tee Red Heather	Women's Short Sleeve Hero Tee Sky Blue	13
Men's Short Sleeve Hero Tee Heather	Men's Short Sleeve Hero Tee Light Blue	11
Learning Thermostat 3rd Gen-USA - Copper	Learning Thermostat 3rd Gen-USA - Stainless Steel	4
Android Women's Short Sleeve Badge Tee Dark Heather	Android Women's Short Sleeve Hero Tee Black	11
Android BTTF Cosmos Graphic Tee	Android BTTF Moonshot Graphic Tee	11
Wool Heather Cap Heather/Black	Wool Heather Cap Heather/Navy	2
Men's Short Sleeve Hero Tee Light Blue	Men's Short Sleeve Hero Tee White	10
Protect Smoke + CO White Battery Alarm - CA	Protect Smoke + CO White Wired Alarm-USA	2
Mood Happy Window Decal	Mood Ninja Window Decal	3
Toddler Short Sleeve T-shirt Grey	Toddler Short Sleeve T-shirt Royal Blue	8
Cam Indoor Security Camera - USA	Cam Outdoor Security Camera - CA	3
Men's Quilted Insulated Vest Battleship Grey	Men's Quilted Insulated Vest Black	13
Women's Vintage Hero Tee Lavender	Women's Vintage Hero Tee Platinum	12
Men's Long & Lean Tee Charcoal	Men's Long & Lean Tee Grey	21
Infant Short Sleeve Tee Royal Blue	Infant Short Sleeve Tee White	8
Youth Short Sleeve T-shirt Green	Youth Short Sleeve T-shirt Royal Blue	5
Women's Short Sleeve Performance Tee Pewter	Women's Short Sleeve Tee	9
Youth Short Sleeve T-shirt Royal Blue	Youth Short Sleeve T-shirt Yellow	6
5-Panel Cap	5-Panel Snapback Cap	2
Mood Ninja Window Decal	Mood Original Window Decal	4
Ballpoint Pen Blue	Ballpoint Stylus Pen	3
Onesie Red	Onesie Red/Graphite	11
Women's Short Sleeve Hero Dark Grey	Women's Short Sleeve Hero Tee Black	12
Men's Vintage Henley	Men's Vintage Tank	16
Android Men's Pep Rally Short Sleeve Tee Navy	Android Men's Short Sleeve Hero Tee Heather	11
Youth Short Sleeve T-shirt Yellow	Youth Short Sleeve Tee Red	15
Android Men's Paradise Short Sleeve Tee Olive	Android Men's Pep Rally Short Sleeve Tee Navy	11
Learning Thermostat 3rd Gen-USA - White	Learning Thermostat 3rd Gen - CA - Stainless Steel	3
Ballpoint LED Light Pen	Ballpoint Pen Blue	2
Toddler Short Sleeve T-shirt Royal Blue	Toddler Short Sleeve Tee Red	15
Men's Performance 1/4 Zip Pullover Heather/Black	Men's Performance Full Zip Jacket Black	13
Waterproof Backpack	Waterproof Gear Bag	4
Infant Zip Hood Pink	Infant Zip Hood Royal Blue	10
Women's Short Sleeve Tee	Women's Short Sleeve Tri-blend Badge Tee Charcoal	9
Android Men's Long & Lean Badge Tee Charcoal	Android Men's Long Sleeve Badge Crew Tee Heather	14
