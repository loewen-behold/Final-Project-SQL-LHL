# Starting With Data

## Question 1: Top 10 highest revenue products?

#### SQL Queries:

    SELECT 	DISTINCT(sbs.productsku), 
        p.name, 
        sbs.total_ordered, 
        p.productprice, 
        sbs.total_ordered * p.productprice AS total_revenue
    FROM sales_by_sku sbs
    JOIN all_sessions al ON sbs.productsku = al.productsku
    JOIN products p ON sbs.productsku = p.sku
    ORDER BY total_revenue DESC
    LIMIT 10;


#### Answer: 

Top 10 products and their revenue

1) Learning Thermostat 3rd Gen-USA - Stainless Steel: $23406.00
2) Cam Outdoor Security Camera - USA: $22288.00
3) Cam Indoor Security Camera - USA: $20298.00
4) 17oz Stainless Steel Sport Bottle: $6342.66
5) Satin Black Ballpoint Pen: $4198.95
6) Protect Smoke + CO White Wired Alarm-USA: $4158.00
7) Blackout Cap: $3589.11
8) Leatherette Journal: $3505.81
9) Android 17oz Stainless Steel Sport Bottle: $3171.33
10) Power Bank: $3135.44



## Question 2: Compute the percentage of visitors to the site that actually makes a purchase

NOTE: For this one, we had to notice that the analytics table only captures visits between 05-01-2017 and 08-01-2017 (3 months), but the all_sessions table has information from 08-01-2016 and 08-01-2017 (1 year).  This means when calculating purchase percentage we had to only count purchases within the same date range.

#### SQL Queries:

    SELECT 	(SELECT COUNT(visitid) FROM all_sessions WHERE totaltransactionrevenue IS NOT NULL AND date BETWEEN '2017-05-01' AND '2017-08-01') AS total_transactions,
        (SELECT COUNT(DISTINCT(visitid)) FROM analytics) AS total_visits,
        (ROUND(
          (SELECT COUNT(visitid)
          FROM all_sessions
          WHERE totaltransactionrevenue IS NOT NULL
            AND date BETWEEN '2017-05-01' AND '2017-08-01') * 100
          /
          (SELECT CAST(COUNT(DISTINCT(visitid)) AS NUMERIC(10,2))
          FROM analytics)
        , 3) || '%'
    )
    AS Percent Visits Resulting In purchase;


#### Answer:

0.015% - An extremely low percentage



## Question 3: Does the amount of time spent on the site have any correlation with a purchase?

#### SQL Queries:

    WITH timevspurchaseCTE AS(
      SELECT 	DISTINCT(an.visitid), 
          an.fullvisitorid, 
          an.timeonsite,
          al.totaltransactionrevenue
      FROM analytics an
      LEFT JOIN all_sessions al ON an.visitid = al.visitid
      WHERE an.timeonsite IS NOT NULL AND al.date BETWEEN '2017-05-01' AND '2017-08-01'
    )
    SELECT MIN(timeonsite), MAX(timeonsite), AVG(timeonsite)
    FROM timevspurchaseCTE
    WHERE totaltransactionrevenue IS NOT NULL
    UNION
    SELECT MIN(timeonsite), MAX(timeonsite), AVG(timeonsite)
    FROM timevspurchaseCTE
    WHERE totaltransactionrevenue IS NULL;


#### Answer:

I only calculated the range and average values of time spent on the site, but the visitors who made a purchase had a lower average time spent on the site (~4 min) than the visitors who did not make a purchase (~14 min).  However, it appears that the minimum time spent on the site is 1 second for someone who made a purchase, so there's obviously an error in the data.



## Question 4: What is the fullvisitorid of the top 10 customers who have viewed the most number of products on the site in the month of July, 2017?  Did any of these visitors make a purchase?

#### SQL Queries:

    WITH productviewsCTE AS (
      SELECT fullvisitorid, COUNT(unit_price) AS products_viewed
      FROM analytics
      WHERE date BETWEEN 20170701 AND 20170801
      GROUP BY fullvisitorid
      ORDER BY products_viewed DESC
      LIMIT 10
    ),
    julypurchasesCTE AS (
      SELECT fullvisitorid 
      FROM all_sessions 
      WHERE totaltransactionrevenue IS NOT NULL AND date BETWEEN '2017-07-01' AND '2017-08-01'
    )
    SELECT 	*,
        CASE 
          WHEN fullvisitorid IN (SELECT * FROM julypurchasesCTE) THEN 'Yes'
          ELSE 'No'
        END AS "Purchase Made?"
    FROM productviewsCTE;


#### Answer:

Top 10 visitor ids for number of products viewed in July:

Full Visitor Id
1) 9681060687378784629 - Viewed 1509 items
2) 4520185947201930710 - 1420
3) 8766368132729375069 - 1254
4) 8561959428190365620 - 1163
5) 5098754129844431163 - 1127
6) 1692871892756005610 - 1057
7) 6385346716669253089 - 1038
8) 8826538902252293768 - 1032
9) 0069486905584188344 - 1019
10) 5490104931040203110 - 969

Not one of these top 10 visitors for products viewed made a purchase.  All of them are extreme window-shoppers!



## Question 5: Top 5 visitors who have visited the site the most number of times.  Inlcude whether any of these customers have ever made a purchase on the site or not.

#### SQL Queries:

    SELECT 	fullvisitorid, 
        COUNT(DISTINCT(visitid)) AS num_visits_to_site,
        CASE
          WHEN fullvisitorid IN 	(SELECT DISTINCT(fullvisitorid) 
                      FROM all_sessions 
                      WHERE totaltransactionrevenue IS NOT NULL)
          THEN 'Yes'
          ELSE 'No'
        END AS "Purchase Made?"			
    FROM analytics
    GROUP BY fullvisitorid
    ORDER BY num_visits_to_site DESC
    LIMIT 10;


#### Answer:

Full Visitor ID and Number of Site Visits
1) 0232377434237234751 - 71 site visits
2) 3148617623907142276 - 70
3) 4731929753431036485 - 64
4) 1957458976293878100 - 49
5) 4215347458239853405 - 46
6) 4038076683036146727 - 43
7) 3937673380007666721 - 34
8) 066988820866909328 - 33
9) 7477638593794484792 - 31
10) 9941821419294775713 - 30

Again, none of these visitors made a purchase.  So number of site visits is not an indicator of a purchaser.  Maybe they need some more incentive?