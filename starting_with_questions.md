# Starting With Questions
    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

SQL Queries:

  WITH total_rev_CTE AS (
    SELECT  country, 
        CASE
          WHEN city = 'not available in demo dataset' THEN 'Not Provided'
          ELSE city
        END AS city,
        SUM(totaltransactionrevenue) AS total_revenue_per_city,
        (SELECT SUM(totaltransactionrevenue) FROM all_sessions as2 WHERE as1.country = as2.country GROUP BY country) AS total_revenue_per_country
    FROM all_sessions as1
    GROUP BY country, city
    HAVING SUM(totaltransactionrevenue) IS NOT NULL
    ORDER BY country, city
  )
  SELECT 	*,
      RANK () OVER (ORDER BY total_revenue_per_city DESC) AS city_rank,
      DENSE_RANK () OVER (ORDER BY total_revenue_per_country DESC) AS country_rank
  FROM total_rev_CTE
  WHERE city != 'Not Provided'
  ORDER BY city_rank, country_rank;


Answer:

  Top 5 Cities:           
  1) San Francisco        
  2) Sunnyvale            
  3) Atlanta              
  4) Palo Alto            
  5) Tel Aviv-Yafo        

  Top 5 Countries:
  1) United States
  2) Isreal
  3) Australia
  4) Canada - Woo!
  5) Switzerland



**Question 2: What is the average number of products ordered from visitors in each city and country?**

SQL Queries:

    WITH avg_prod_CTE AS (
      SELECT 	country, 
          CASE
            WHEN city = 'not available in demo dataset' THEN 'Not Provided'
            ELSE city
          END AS city, 
          ROUND(AVG(productquantity)) AS avg_num_prod_per_visitor_city,
          ROUND((SELECT AVG(productquantity) FROM all_sessions as2 WHERE as1.country = as2.country AND totaltransactionrevenue IS NOT NULL GROUP BY country)) AS avg_num_prod_per_visitor_country
      FROM all_sessions as1
      WHERE totaltransactionrevenue IS NOT NULL
      GROUP BY country, city
    )
    SELECT 	*,
        DENSE_RANK () OVER (ORDER BY avg_num_prod_per_visitor_city DESC) AS city_rank,
        DENSE_RANK () OVER (ORDER BY avg_num_prod_per_visitor_country DESC) AS country_rank
    FROM avg_prod_CTE
    WHERE city != 'Not Provided'
    ORDER BY city_rank, country_rank;


Answer:

Average number of products by city and their ranking:
1) Sunnyvale -	43
2) Atlanta -	36
3) Chicago -	19
4) Seattle -	12
5) Tel Aviv-Yafo -	7
6) San Francisco -	6
7) Los Angeles -	5
7) Austin -	5
7) San Bruno -	5
8) Toronto -	4
8) Zurich -	4
9) Mountain View -	3
9) New York -	3
9) Sydney	- 3
10) San Jose	- 2
10) Palo Alto	- 2
11) Nashville	- 1
11) Houston	- 1
11) Columbus	- 1


  COUNTRY         CITY            AVG NUM PRODUCTS BY CITY      AVG NUM PRODUCTS BY COUNTRY   CITY RANKING    COUNTRY RANKING
  United States...Sunnyvale.......43............................15	                          1	              1
  United States	  Atlanta	        36	                          15	                          2	              1
  United States	  Chicago	        19	                          15	                          3	              1
  United States	  Seattle	        12	                          15	                          4	              1
  Israel	        Tel Aviv-Yafo	  7	                            7	                            5	              2
  United States	  San Francisco	  6	                            15	                          6	              1
  United States	  Los Angeles	    5	                            15	                          7	              1
  United States	  Austin	        5	                            15	                          7	              1
  United States	  San Bruno	      5	                            15	                          7	              1
  Canada	        Toronto	        4	                            4	                            8	              3
  Switzerland	    Zurich	        4	                            4	                            8 	            3
  United States	  Mountain View	  3	                            15	                          9 	            1
  United States	  New York	      3	                            15	                          9 	            1
  Australia	      Sydney	        3	                            3	                            9 	            4
  United States	  San Jose	      2	                            15	                          10	            1
  United States	  Palo Alto	      2	                            15	                          10	            1
  United States	  Nashville	      1	                            15	                          11	            1
  United States	  Houston	        1	                            15	                          11	            1
  United States	  Columbus	      1	                            15	                          11	            1



**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**

SQL Queries:

  SELECT 	DISTINCT(country),
      v2productcategory AS types_of_products
  FROM all_sessions
  WHERE totaltransactionrevenue IS NOT NULL
  ORDER BY country, types_of_products;

  SELECT 	DISTINCT(city),
      v2productcategory AS types_of_products
  FROM all_sessions
  WHERE totaltransactionrevenue IS NOT NULL
  ORDER BY city, types_of_products;


Answer:

As far as purchase patterns for each country, it appears that Canada and Switzerland like their apparel, likely because it's cold and they need the extra layers.  Also, they must REALLY know their clothing sizes to risk purchasing their clothes online.  Australia and Isreal are more into purchasing Home Security - cameras to monitor for theives sniping their amazon packages from their front doors, and thermostats to control the blistering heat in their warmer climates.  Finally, the United States just buys it all!  Why head to the physical store when you can just buy it online? Muahaha!

When it comes to city purchases, we're all over the place.  One could make any number of conclusions, but I'll offer one:
The West Coast cities (San Fransisco, Seattle, Sunnyvale, LA, San Jose, Palo Alto, Mountain View) are all interested in buying Home Security.  Especially in California, where most of these cities are located, the crime rates are higher and therefore more home security items are purchased.  



**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**

SQL Queries:

--Top-selling product per country:

  WITH country_product_quantities_CTE AS (
    SELECT 	country,
        productname,
        SUM(productquantity) AS quantity_sold,
        RANK() OVER (PARTITION BY country ORDER BY SUM(productquantity) DESC) AS rank
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP by country, productname
  )
  SELECT 	country,
      productname AS top_product,
      quantity_sold
  FROM country_product_quantities_CTE
  WHERE rank = 1;


--Top-selling product per city:

  WITH city_product_quantities_CTE AS (
    SELECT 	city,
        productname,
        SUM(productquantity) AS quantity_sold,
        RANK() OVER (PARTITION BY city ORDER BY SUM(productquantity) DESC) AS rank
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL
    GROUP by city, productname
  )
  SELECT 	city,
      productname AS top_product,
      quantity_sold
  FROM city_product_quantities_CTE
  WHERE rank = 1
  ORDER BY quantity_sold DESC;


Answers:

--Top products per country--
The United States seems to buy alot of reusable shopping bags.  Maybe Americans are like me and forget my reusable bags in the car EVERY time I go shopping, which leads me to buy more reusable bags.  That might be my personal top-selling product as well ;).  As far as information we can extract from this beyond what was observed in #3, I can't really tell.

--Top products per city--
This query yields more interesting results than the top-products per country did.  For example, in Q#3 when we determined which product categories were purchased in each city, many of the California cities had Home Security prodcuts in their category list.  However, now that we look at the top products for each city it's apparent that many of the California city's top products are weather related - specifically that it's warmer in Cali. We see the top products in these places are things like SPF lip balm, tee-shirts, and home temperature control.  This makes sense.



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

  WITH total_rev_CTE AS (
    SELECT  country, 
        CASE
          WHEN city = 'not available in demo dataset' THEN 'Not Provided'
          ELSE city
        END AS city,
        SUM(totaltransactionrevenue) AS total_revenue_per_city,
        (SELECT SUM(totaltransactionrevenue) FROM all_sessions as2 WHERE as1.country = as2.country AND totaltransactionrevenue IS NOT NULL GROUP BY country) AS total_revenue_per_country
    FROM all_sessions as1
    GROUP BY country, city
    HAVING SUM(totaltransactionrevenue) IS NOT NULL
    ORDER BY country, city
  )
  SELECT 	*,
      RANK () OVER (ORDER BY total_revenue_per_city DESC) AS city_rank,
      DENSE_RANK () OVER (ORDER BY total_revenue_per_country DESC) AS country_rank
  FROM total_rev_CTE
  WHERE city != 'Not Provided'
  ORDER BY city_rank, country_rank;


Answer:

This query is identical to the one I used in Q1.  While I'm not sure what "impact" means in this case, I can say that the more affluent cities/countries generate the most revenue.  In the very least, the larger/largest cities in each country seems to generate the most revenue.  This makes sense, since more businesses, more people, more demand for goods would exist in these places.







