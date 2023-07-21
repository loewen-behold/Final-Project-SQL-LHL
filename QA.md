# QA Process

## Risk Areas Addressed
These are some of the general risk areas that I addressed, whether via cleaning, targeted filtering, or as a simple query to double-check analysis results:

1. Duplicate data: These especially became an issue with aggregate functions, offering faulty conclusions.
2. Null values: I would either ignore these, change them using an educated guess or grabbing data from other tables, or treat them as 0s or 1s, depending on the circumstance. 
3. Inconsistent Data: There were plenty of times where values and calculation results were not what I was expected.  There were also columns that didn't have very user-friendly names and different columns shared by tables that have different datatypes or names.
4. Calculation validity control: In some cases I wanted to double-check the validity of some aggregate calculations I created to ensure they were constructed and used properly.
5. BONUS SECTION - This includes some thought processes/rabbit-holes I went down while initially exploring the tables.  I felt that I should include this in the QA section because you can't really develop air-tight QA procedures unless you understand the data you're working with.  I mainly wanted to explore what was unique, what was duplicated, where the tables were connected, when the data provided didn't make sense, which data I felt I could use in analysis and which was useless, etc.


## QA Process (with Queries)
This is, by no means, an exhaustive list of ALL of the queries in dealing with specific risk areas.  Below I've simply provided a few examples and portions of my queries that I felt dealt with some of these risk areas.

1. Finding and dealing with duplicate data: 
- In order to deal with duplicate data, I had to be able to find out whether it was duplicated or not.  In this example, I checked to see whether the visitid field was unique in the all_sessions table.  This query specifically returns all the visitids that have more than one row associated with it. 

    SELECT *
    FROM all_sessions
    WHERE visitid IN (
                      SELECT visitid
                      FROM all_sessions
                      GROUP BY visitid
                      HAVING COUNT(*) > 1
                      )
    ORDER BY visitid;

- Once I know that I had duplicate data, I recognized that if these duplicates are included in my query, they would affect my aggregate functions.  More specifically calculations like SUM, AVG, COUNT, etc. are affected because some values would be included multiple times.  This can ultimately lead us to making false conclusions. I would deal with these by making good use of the DISTINCT feature, and then I would double check the calculations afterwards to ensure that the duplicate data was indeed excluded from the results.  Here's a basic query making use of DISTINCT:   
    SELECT DISTINCT(visitid), COUNT(*)
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL;

2. Finding and dealing with NULL values:   
- In most cases, I would simply exclude rows with NULL values when they were not needed for my specific query. In order to filter for these specific entries, I would use the IS NULL or IS NOT NULL statement in my query.  For example:

    SELECT *
    FROM all_sessions
    WHERE totaltransactionrevenue IS NOT NULL;

- In some cases, I would treat a NULL as a 0 or a 1 (depending of the scenario) if I were wanting to perform calculations with columns that had them.  
For example, when performing calculations involving the productquantity column in the all_sessions table , if the productquantity was NULL, the calculation would be incorrect.  In this case I would change productquantity to 1.  Here is the query I included in cleaning_data.md file to address these values.  Keep in mind there were multiple changes being made, but the "ELSE 1" in this case would deal with the NULL values for productquantity.

    UPDATE all_sessions al
      SET productquantity = 
      CASE WHEN al.totaltransactionrevenue IS NOT NULL
          THEN
          CASE	WHEN al.visitid IN (SELECT DISTINCT(visitid) FROM analytics) THEN (SELECT SUM(units_sold) 
                                                                                  FROM analytics a 
                                                                                  WHERE al.visitid = a.visitid)
              WHEN FLOOR(totaltransactionrevenue/productprice) = 0 THEN 1
              WHEN FLOOR(totaltransactionrevenue/productprice) != productquantity THEN FLOOR(totaltransactionrevenue/productprice)
              WHEN FLOOR(totaltransactionrevenue/productprice) != 1 THEN FLOOR(totaltransactionrevenue/productprice)
              ELSE 1
          END
        ELSE al.productquantity
      END;


3. Inconsistent Data:     
- When results or columns were not in the format I expected, I would either make changes to the table itself or just to a specific calculation with the CAST, TO_DATE, TO_CHAR, ROUND, or FLOOR functions, depending on what was necessary. For example, here's my query for changing the datatype of the totaltransactionrevenue column and divide it by 1 million in the all_sessions table.

    ALTER TABLE all_sessions
    ALTER COLUMN totaltransactionrevenue TYPE numeric(15,2);
    
    UPDATE all_sessions
    SET totaltransactionrevenue = ROUND(totaltransactionrevenue/1000000, 2);

- I also changed a couple column names that didn't have user-friendly names. Here's an example of changing the productcategory and productname columns to exclude the "v2":

    ALTER TABLE all_sessions
    RENAME COLUMN v2productcategory TO productcategory ;

    ALTER TABLE all_sessions
    RENAME COLUMN v2productname TO productname;


4. Calculation validity control:  
- Here are some specific QA Queries I created in order to double check that my results/calculations were accurate and unaffected by duplicate/null/inconsistent data:

- For the question "Which cities and countries have the highest level of transaction revenues on the site?", my query would SUM the total revenue of each city and each country within the same query.  In the past, my results haven't always "added up" if I've messed up by GROUP BY statement or my PARTITION BY statement.  So I created a query that compares the total revenue listed for all the cities to the sum of the totaltransactionrevenue in the all_sessions table.   If the original query were created properly, these values should be the same.

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
    SELECT	CASE
          WHEN SUM(total_revenue_per_city) = (SELECT SUM(totaltransactionrevenue) FROM all_sessions) THEN 'Good!'
          ELSE 'Bad!'
        END AS QA_Results
    FROM total_rev_CTE;


- For the question asking "What is the top-selling product from each city/country?" I needed to SUM the number of products sold according to each country so my QA query double checks that the original query is actually summing the products correctly.  I compared the total of the quantity_sold column with a sum of all productquantities on the all_sessions table.

    WITH t1 AS (
      SELECT 	country,
          productname,
          SUM(productquantity) AS quantity_sold,
          RANK() OVER (PARTITION BY country ORDER BY SUM(productquantity) DESC) AS rank
      FROM all_sessions
      WHERE totaltransactionrevenue IS NOT NULL
      GROUP BY country, productname
    )
    SELECT 	CASE
          WHEN SUM(quantity_sold) = (SELECT SUM(productquantity) FROM all_sessions WHERE totaltransactionrevenue IS NOT NULL)
          THEN 'Good!'
          ELSE 'Bad!'
        END AS QA_Result
    FROM t1;


5. BONUS QA SECTION:  
These are some of my thought processes/rabbit-holes/queries when trying to understand what the data on the analytics and all_sessions.  Some of this was me trying to see what was unique and what wasn't, how the tables were connected, and exploring some discrepencies in what I was seeing compared to what I thought I would see.

a. Exploring visitid: 
This seems to be a unique identifier of each website visit.  When the entire table is viewed, it appears that many of the rows are duplicated. At first I thought this was only due to the unit_price column, but after "eliminating" this column and counting how many distinct visitids still appeared more than once, I found there were still quite a number of them. 

    SELECT DISTINCT(visitid), count(*) AS counts
		FROM (
        SELECT DISTINCT(visitid), visitnumber, visitstarttime, date, fullvisitorid, userid, channelgrouping, socialengagementtype, units_sold, pageviews, timeonsite, bounces, revenue
				FROM analytics) AS subq1 
    GROUP BY visitid 
    HAVING count(*) > 1;

I then focused in on one of these visitids on the list, particularly visitid = 1493627652, and found there was variation not only in the unit_price column, but in the revenue and units_sold columns as well.
    SELECT *
    FROM analytics
    WHERE visitid = 1493627652;

Overall, I may have been led to believe that maybe the unit_prices column was designed to capture the different items a user would click on, but as to why there are varying units_sold and revenue values for one visitid is unclear.  When I looked up this visitid in the all_sessions table, the query returned nothing, leading me to believe that nothing was purchased in this visit.  But why have a value in the "units_sold" column if nothing was sold?  Unclear.


b. Exploring visitstarttime:
It was evident that this is the same value as the corresponding visitid.  Although there are a few that are close but don't quite match.  These were my queries to explore those that matched and those that did not:

--Counting how many distinct visitids match the visitstarttime (147894):

    SELECT COUNT(DISTINCT(visitid))
    FROM analytics
    WHERE visitid = visitstarttime;

--Counting how many distinct visitids do not match the visitstarttime (985):

    SELECT COUNT(DISTINCT(visitid))
    FROM analytics
    WHERE visitid != visitstarttime;

It was difficult to determine whether the visitstarttime was intended to be identical to the visitid or not, but maybe that is how the visitid is generated, or maybe it was done in error.  Nonetheless, the visitstarttime does not even appear to be a valid time.  I then tried to figure out what the starttime might represent.  I tried a number of different ways to determine this, playing with one particular visitid to see if I could figure out if it was in milliseconds or seconds from the beginning of the year or from the epoch year, etc.  These are just a couple of my attemps.

--Is it number of milliseconds from the time data was being collected? - timestamp incorrect based on other data

    SELECT 	visitid, 
            visitstarttime, 
            timestamp '2017-05-01 00:00:00' + interval '1493622167 milliseconds', 
            date, 
            timeonsite
    FROM analytics
    WHERE visitid = 1493622167
    LIMIT 1;

--How about from the beginning of the year? - timestamp generated is incorrect again

    SELECT 	visitid, 
            visitstarttime, 
            timestamp '2017-01-01 00:00:00' + interval '1493622167 milliseconds', 
            date,
            timeonsite
    FROM analytics
    WHERE visitid = 1493622167
    LIMIT 1;

I also tried using seconds, but was getting a return date that was in the year 2064.

What leads me to believe that this may represent some value for time is that two different fullvisitorids can end up having the same visitid.  Why this is suspicious is because if the visitid is being generated by a time of some sort, two different users could visit the site at exactly the same time, thus producing a visitid that is identical.  If that's the case, this is a poor method of generating unique visitids.
    
Here was my query to see which visitids had more than one fullvisitorid attached to it:

    SELECT visitid, COUNT(fullvisitorid)
    FROM (
	      SELECT DISTINCT(visitid), fullvisitorid
	      FROM analytics
	      ) AS subq
    GROUP BY visitid
    HAVING COUNT(fullvisitorid) > 1;

If I had more time, I would have liked to be able to figure out what this starttime is supposed to represent.

c. Exploring revenue:
These have varying values, but perhaps they represents the revenue generated during that visit?  I checked to see whether the revenue would be unique for every visitid, but found this is not the case.

    SELECT visitid, COUNT(revenue)
    FROM analytics
    WHERE revenue IS NOT NULL
    GROUP BY visitid
    HAVING COUNT(revenue) > 1;

So it could be the case that more than one purchase could be made per visit.  In which case, the sum of the revenues per visit should be equal to the totaltransactionrevenue column for that same visitid in the all_sessions table.  Indeed, I found that this was the case.  Here was my query for that:

    SELECT 	an.visitid, 
            SUM(an.revenue) AS total_revenue, 
            al.totaltransactionrevenue
    FROM analytics an
    JOIN all_sessions al ON an.visitid = al.visitid
    WHERE an.revenue IS NOT NULL AND al.totaltransactionrevenue IS NOT NULL
    GROUP BY an.visitid, al.totaltransactionrevenue;

NOTE: There are a couple of sums that don't quite add up, but they are off by 1/100000 dollars if these revenue and price columns were to be in dollars.  So I believe it's just a rounding discrepency.

I also wanted to check whether there were visits in the analytics table that have a value for revenue, but do not appear in the all_sessions table.

    SELECT DISTINCT(visitid) 
    FROM analytics
    WHERE visitid NOT IN (
      SELECT visitid
      FROM all_sessions
      WHERE totaltransactionrevenue IS NOT NULL
      )
      AND revenue IS NOT NULL;

This was the case, and so there are some visits in analytics that do have a value for revenue, but don't actually appear to end up as a completed transaction.  I'm not sure how this could happen, but it was good to know.

## Conclusions

When dealing with such inconsistent, missing, and duplicate data, especially when I have NO CLUE how this data was generated and what much of it is intended to represent, I can definitely see the need for a robust QA process.  It would be so easy to make the wrong conclusions and assumptions and sometimes the person analyzing the data would never even suspect it.  So it's essential to have checks and balances included in the process in order to mitigate that error.