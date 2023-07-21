# Cleaning The Data

The following includes some of the issues I decided to clean, along with the queries I used to achieve this after each issue.  I didn't decide to clean the entire database since we are lacking context as to what some of the data represents, but I did want to include a number of different techniques that deal with missing data, changing datatypes, dealing with null values, and cross-referencing from other tables.  My goal is to just show a broad range of skills, but no an exhaustive list.

For each table, I've included a list of the changes made to it, but also decided in some cases to include my thought process and queries in order to discover what I needed to change.


## All_Sessions table

**1.Datatype change and format update**
Changed monetary columns to NUMERIC instead of Integer and then moved the decimal place to represent a dollar value by dividing existing values by 1,000,000.

  Query:

    ALTER TABLE all_sessions
    ALTER COLUMN totaltransactionrevenue TYPE numeric(15,2),
    ALTER COLUMN productprice TYPE numeric(15,2),
    ALTER COLUMN productrevenue TYPE numeric(15,2),
    ALTER COLUMN itemrevenue TYPE numeric(15,2),
    ALTER COLUMN transactionrevenue TYPE numeric(15,2);

    UPDATE all_sessions
    SET totaltransactionrevenue = ROUND(totaltransactionrevenue/1000000, 2);
    UPDATE all_sessions
    SET productprice = ROUND(productprice/1000000, 2);
    UPDATE all_sessions
    SET productrevenue = ROUND(productrevenue/1000000, 2);
    UPDATE all_sessions
    SET itemrevenue = ROUND(itemrevenue/1000000, 2);
    UPDATE all_sessions
    SET transactionrevenue = ROUND(transactionrevenue/1000000, 2);

**2. Removing duplicate transactions**
When I was trying to determine the number of visitids that contain a value in the totaltransactionrevenue column, I received a different number of rows when I change  "visitid" to "distinct(visitid)" in my SELECT statement.  This told me that either a unique visitid may have more than one transaction or there might be a duplicate row for that visitid. 

  I isolated the visitid(s) that has more than one row using the following query:
    
    SELECT *
    FROM all_sessions
    WHERE visitid IN (
      SELECT visitid
      FROM all_sessions
      GROUP BY visitid
      HAVING COUNT(*) > 1
      )
      AND totaltransactionrevenue IS NOT NULL
    ORDER BY visitid;

After inspecting this visitid, I concluded that in this instance it was a duplicate row.  Even though the time, product info, and transactionids were different between entries it didn't make sense to have a different time for a single visit (same visitid, totaltransactionrevenue, transactions, timeonsite, pageviews, etc).  Thus, I decided to delete that entry so it didn't affect my analysis when summing transaction revenues for the countries.  

  Here's my query for deleting that specific entry in the table:
    
    DELETE FROM all_sessions
    WHERE fullvisitorid = '3764227345226401562' and time = 0 AND visitid = 1490046065;

Despite the fact that I deleted this particular entry for my own analysis query to be more accurate, it did lead me to wonder how many other duplicate rows there are for each visitid, with the same fullvisitorid. 

  Here's my query for checking for visitids that have more than one row associated with it:
    
    SELECT *
    FROM all_sessions
    WHERE visitid IN (
      SELECT visitid
      FROM all_sessions
      GROUP BY visitid
      HAVING COUNT(*) > 1
      )
    ORDER BY visitid;

Alas, it appeared that there are multiple entries that have the same visitid and fullvisitorid, but may have different starttimes, product info, or transactionids.  I wasn't sure what to do with this information at this point, but I'm aware that if I were to perform any analysis on the duplicated visitids, I'd have to address that so as not to get faulty values. 

**3. Correcting information**
I noticed that some cities seemed to be paired with the wrong country - namely New York listed as a Canadian City.

  I changed that specific entry's country from Canada to United States with the following query:
    
    UPDATE all_sessions
    SET country = 'United States' WHERE country = 'Canada' AND city = 'New York';

If I were to want to change all of the entries where the cities and countries were mismatched, I'd have to compare each entry in the all_sessions table with an accurate list of cities and their countries.  I did not have the time to perform this change on the entire table.

**4. Update the productquantity column to better reflect the number of products purchased.**  
For this, I cross-referenced the visitid from the analytics table and if it exists, I summed up the units_sold from that visitid and used that sum to fill in the productquantity.  If it doesn't exist in the analytics page, I would instead divide the totaltransactionrevenue by the product price to estimate how many of the single item listed would have been sold.  I recognize this is not necessarily accurate, because if you look up a visitid that is found in both the analytics and all_sessions pages, the all_sessions will only have listed one product, whereas on the analytics table, there will have been multiple different products purchased.  However, I still wanted to make an educated guess on what the value for productquantity should be.  The analytics table is accurate in the units_sold because if you sum the revenue up for that particular visitid, it always matches the totaltransactionrevenue column in the all_sessions table.  

  Here is my query making these changes to the productquantity column:

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


I also wanted to check to see if it worked using this query and it was a success!
    
    SELECT 	visitid,
        totaltransactionrevenue,
        productquantity,
        productprice
    FROM all_sessions al
    WHERE totaltransactionrevenue IS NOT NULL;


**5. Column name changes**
Changed the names of the columns v2productcategory and v2productname to productcategory and productname, respectively. This is more readable for the user.

Queries to alter names:

    ALTER TABLE all_sessions
    RENAME COLUMN v2productcategory TO productcategory;

    ALTER TABLE all_sessions
    RENAME COLUMN v2productname TO productname;


**6. Updated product categories column**
Changed product categories to something more legible - Once again, I only included the items affecting my queries in order to save some time, but the procedure would be very similar if I were to perform over the whole table.

Query to rename categories:

    UPDATE all_sessions al
    SET productcategory = 
    CASE WHEN al.totaltransactionrevenue IS NOT NULL
        THEN
        CASE
          WHEN productcategory LIKE '%Apparel%' THEN 'Apparel'
          WHEN productcategory LIKE '%Bags%' THEN 'Bags'
          WHEN productcategory LIKE '%Electronics%' OR productcategory LIKE '%Lifestyle%' THEN 'Electronics'
          WHEN productcategory LIKE '%Drinkware%' THEN 'Drinkware'
          WHEN productcategory LIKE '%Pet%' THEN 'Pet'
          WHEN productcategory LIKE '%Office%' THEN 'Office'
          WHEN productcategory LIKE '%Housewares%' THEN 'Housewares'
          WHEN productcategory LIKE '%Fun%' THEN 'Toys'
          WHEN productcategory LIKE '%Nest%' THEN 'Home Security'
          WHEN productcategory LIKE '%Waze%' THEN 'Car Accesories'
          WHEN productname LIKE '%Sleeve%' THEN 'Apparel'
          WHEN productname LIKE '% Tee %' THEN 'Apparel'
          WHEN productname LIKE '%Nest%' THEN 'Home Security'
          WHEN productname LIKE '%Sticker%' THEN 'Toys'
          WHEN productname LIKE '%Bottle%' THEN 'Drinkware'
          ELSE productcategory
        END
      ELSE productcategory
    END;


## Products table

**1. Added and populated a new column - product prices**
Added and populated a column with product prices by grabbing prices from the all_sessions table via productsku.  First I want to see whether all the products had a product price attached to them in the all_sessions table.

Query:
    
    SELECT DISTINCT(p.sku), p.name, al.productprice
    FROM products p
    LEFT OUTER JOIN all_sessions al ON p.sku = al.productsku;

It turned out that they do not all have prices.  But I will still wanted to perform this append to the products column since this is where I believe the product prices should be.  First, I added the column to the table.

Query: 
    
    ALTER TABLE products
    ADD COLUMN productprice NUMERIC(15,2);

Then, I tried to use this query to fill in the column from the all_sessions table, but it didn't work. It said that it returned too many rows, which means that in the all_sessions table there are different product prices associated with a single sku.

Query:

    UPDATE products p
    SET productprice = (SELECT productprice FROM all_sessions al WHERE p.sku = al.productsku);

This is the query I ended up using instead.  It looks for the price for a single sku that appears the most times, and uses that value for product price.  In the case of a "tie" (that is, two prices for a single sku were both used the most times), I'd opt for the greater of the two prices.  All in all, this was a monstrosity of a query, but it got the job done.

Query:

    UPDATE products p
    SET productprice = 
    CASE 
      WHEN p.sku = 'GGOEGAAX0755' THEN 25.99
      WHEN p.sku = 'GGOEGAFB035814' THEN 55.99
      WHEN p.sku = 'GGOEGOCD078399' THEN 44.99
      WHEN p.sku IN (SELECT productsku FROM all_sessions GROUP BY productsku HAVING COUNT(DISTINCT(productprice)) > 1)
        THEN (SELECT productprice 
            FROM(	SELECT DISTINCT(productsku), productprice, RANK() OVER (PARTITION BY productsku ORDER BY count DESC) AS rank
              FROM (	SELECT 	distinct(productsku), 
                      productprice, 
                      COUNT(*) OVER (PARTITION BY productsku, productprice) AS count
                  FROM all_sessions
                  WHERE productSKU IN (SELECT productsku
                            FROM all_sessions
                            GROUP BY productsku
                            HAVING COUNT(DISTINCT(productprice)) > 1)
                  ORDER BY productsku
              ) AS price_counts
              GROUP BY productsku, productprice, count
            ) AS subq
            WHERE p.sku = subq.productsku AND rank=1
        )
      ELSE (SELECT productprice 
          FROM (
              SELECT 	distinct(productsku), productprice
            FROM all_sessions
            WHERE productSKU IN (SELECT productsku
                      FROM all_sessions
                      GROUP BY productsku
                      HAVING COUNT(DISTINCT(productprice)) = 1)
            ORDER BY productsku
          ) subq
          WHERE p.sku = subq.productsku)
    END;


## Analytics table

**1. Dealing with timeonsite NULL values**
I wanted to fill in the NULL timeonsite values by using the average value for timeonesite from the other entries that had the same number of pageviews.  I tried to perform this change, but after 1 hour of this query processing, I wasn't sure how long it was going to take, so I cancelled the process. I still wanted to leave the code as an example of what I would have done - but there must be a more efficient way.  Or maybe not since this is a 4 million row table.

Query:
      
      UPDATE analytics an
      SET timeonsite =
      CASE 
        WHEN timeonsite IS NULL
        THEN (SELECT avg_time_per_views
            FROM (
              SELECT 	DISTINCT(pageviews),
                  ROUND(AVG(timeonsite)) AS avg_time_per_views
              FROM 
              (
                SELECT 	DISTINCT(visitid), 
                    fullvisitorid, 
                    pageviews, 
                    timeonsite
                FROM analytics
              ) AS subq
              WHERE timeonsite IS NOT NULL AND pageviews IS NOT NULL
              GROUP BY pageviews
              ORDER BY pageviews
            ) AS subq2
              WHERE an.pageviews = subq2.pageviews
        )
        ELSE timeonsite
      END;

