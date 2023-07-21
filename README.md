# Final-Project-Transforming-and-Analyzing-Data-with-SQL


## Project/Goals
1. Understanding the data:  
- Explore the data in order to determine what information the tables are telling me, as well as how the tables are connected to each other.  
- Find any data discrepancies or areas of concern so they can be "dealt with" or avoided.  
- Get to know what datatypes I'm working with, and maybe even determine why certain datatypes were used as opposed to others (ie. a varchar instead of an integer).

2. Clean the data:  
- Change any datatypes to be more consistent across tables 
- Remove any duplicate rows if needed 
- Rename columns to more user-friendly names
- Create primary and foreign keys for ease of querying if possible

3. Formulating/Answering questions: 
- Construct the queries that can answer these questions.
- Try and make queries that are re-useable, even if the data were to be updated and altered.

4. Construct a QA process: 
- Ensure that the data is consistent across tables
- Double-check complex query results in order to avoid errors


## Process
#### 1. Understanding the data

This data was difficult to understand in many ways. A few of the tables were a bit more straightforward in what they represented, namely the products, sales_by_sku, and the sales_report tables.  The tables that I found to be the most difficult to understand were the all_sessions and analytics tables.  They were not only difficult to understand individually, but also in how these two tables were connected. 

I've summarized some of my observations below

- Observation 1: There are many NULL entries, including entire columns.  I will have to determine whether these columns should stay or be eliminated based on the what the project/business goals are.  My first thought is simply to remove these columns, but maybe they have been created for future use.  In that case there is no harm in keeping them and filtering them out if necessary.

- Observation 2: Some of the tables seem to include some overlapping information with other tables, which may be of some benefit depending on what is wanting to be achieved, but I believe overall could be cleaned up.  Having this data duplicated across columns is inefficient because this introduces more room for error when the data is being updated, deleted, or altered.  More specifically, any changes would have to be done in multiple places instead of just one.

- Observation 3: In some cases, the date and time columns are integer values instead of date types or timestamps. Due to this, the times are not always easy to understand what they are to represent, and leads to questions that may not be answered easily without insight from someone who knows - Are they the seconds or milliseconds that have elapsed from some starting date/time?  Were they copied from another column by mistake?   Why is there more than one time listed for each visitid?, etc.

- Observation 4: Some of the column names are not common across the tables, and some are just not named in a friendly way.

- Observation 5: Some tables include columns that are absolutely unclear what they are intended to represent or achieve.  For example, the unit_price column in the analytics table.  Is that meant to represent all the products a user clicks on during a single site visit?  And if so, why not use the productsku or productname instead?  If it's just meant to be a count, then make it a column called "products_viewed" or something like that.  Without any other information to support the unit_price, it's not even clear what products were viewed.  But maybe that doesn't matter.  This will remain unclear.

- Observation 6: primary keys are NOT coming easy for the all_sessions and analytics tables. With so many near-duplicate rows in each of these tables, they would need a complex primary key involving 4 rows or more.  This is not ideal and indicates to me that these tables could be dismantled into multiple tables in order to have more obvious and easy primary keys created.

- Observation 7: There are so many unknowns about the data that it makes it a nearly impossible task to fully clean without having someone else to consult.  As a result, I decided to take a different approach and do my cleaning out of necessity when answering the questions.  If a particular query I used "touched" data that needed cleaning, then I would address it at that point.  What I did realize from this, however, is that it's entirely possible that working with such large tables in a real-life setting could present the same challenges, and likely "new" things are found needing to be cleaned only after someone has explored the data trying to answer questions.


#### 2. Cleaning the data

As mentioned, I ended up, for the most part, cleaning the data as I moved along through the questions.  I by no means cleaned the dataset completely, and in many cases, only cleaned the specific rows that a particular query was utilizing.  I did this to save some time, but also because if I showed I could do it for specific rows, then I could also do it across an entire table.  All changes, with their descriptions can be found in the **cleaning_data.md** file.


#### 3. Formulating/Answering questions

When asking and answering questions, I was able to generate some interesting insights that could be useful in a business and marketing setting.  I've summarized these insights in the results section below.  For specific queries and answers to the questions, refer to the **starting_with_questions.md** and **starting_with_data.md** files.


#### 4. Construct a QA process

For this process, I had two main focuses.  The first was to develop QA techniques in order to work with the raw data - dealing with null, duplicate, and inconsistent data.  The second focus was to implement QA on the results I received from particular queries, especially those that were more complex and contained aggregate and window functions.  For a more detailed breakdown of my QA process, please refer to the **QA.md** file.


## Results
After analyzing and exploring the data, there are a number of valuable insights that can be made.  Particularly regarding site visits/behaviours, geographical trends, revenue generation, and product popularity.

### Site visits/behaviours:

- Using the analytics and all_sessions tables we are able to observe the behaviours of the site visitors.  We can determine things such as the frequency that visitors use the site, how long and how many times they use it, the typical times of the day and month they are using it, and what sorts of products they are viewing.  From a business standpoint, this information could be helpful when generating ads specific to the visitor.  

- I was particularly interested in determining what viewing behaviours might be indicators of a visitor making a purchase or not.  A few of the behaviours I explored more in detail was the time spent on the site, number of product views, and total number of site visits.  Interestingly enough, all of these yielded results opposite to what I would have expected. It turns out that customers who spend more time on the site are less likely to make a purchase than those that spend less time on the site.  This inverse relationship with purchases was also observed with the number of product views and number of site visits. 

- I also wanted to know what percentage of visits actually generate purchases.  This could only be calculated for a 3 month period, since that was the overlap in dates between the analytics and all_sessions tables.  This number was also surprisingly low - only 0.015% of visits.  However, I do recognize that without a full understanding of the data I may have misinterpreted what actually represents a purchase or not.

- With all of this information, maybe after a certain amount of time has elapsed or number of pages have been visited, the site should have a coupon for 5% the visitor's order.  Maybe that would entice the users who typically don't make a purchase to actually make one.

### Geographical Trends:

- Another thing that can be determined is purchase trends based on geographical location, including city and country data.  What I observed was that the United States is the top purchasing country on this site by a large margin.  It's possible that advertisement and marketing is not reaching to other countries as well as hoped, or maybe the strategies being used are more tailored to the western culture.   I also determined that geographical location plays a part in the types of products being purchased as well as what revenue is being generated.  For example, much of the revenue was being generated from the more "affluent" cities and countries.

- What was most interesting to me was the difference in the "story" I was seeing unfold when determining the top categories being purchased versus the top products being purchased from different locations.  For example, when looking at the most popular categories per region it appeared that many of the West Coast USA cities had Home Security/Home Temp control products as some of their most popular categories.  However, "zooming in" and looking at the most popular products for these West Coast cities was not actually home devices, but products specific to warmer climates - which makes sense!  


### Revenue/Product Insights:

- Given the data, we are able to look for patterns that correlate to increased revenue and identify patterns for potential growth points.

- I was able to determine what the top products are for generating revenue - top 3 are security cameras and thermostats.  There were not only a good amount of them sold, but they are typically higher valued items as well.  What was interesting to me was that number 4 and 5 on the list was a stainless steel water bottle and a satin black ballpoint pen, respectively.  Sometimes even the smaller items will generate some of the most revenue!

- I've already talked about some of the findings for revenue and product popularity in regards to geographical location, but some things I didn't have the time to look into were comparing revenue to a visitors channel preference or social engagement.  I could have also investigated revenue with regards to the time - which time of the day or time of the month is revenue generation the highest, etc.  

### Summary 
In summary, this data-driven approach empowers businesses to refine marketing strategies, optimize revenue streams, and enhance product offerings. By constantly analyzing and leveraging these insights, a competitive edge can be maintained in the market and continuously improve overall business performance.


## Challenges 

1. Uploading the csv files into a table was a challenge - if just one datatype is wrong, then it wouldn't work.  

2. Having absolutely no context or subject matter experts to talk to in regards to the table made it difficult to understand what some of the data was to represent, whether it needed to be there or not, and when to consider it "faulty" or not.

3. Long query times on the analytics table - especially when performing either a subquery that referenced all of the 4M rows of the analytics table or trying to make an alteration to the 4M rows took too long.  There were a number of times when I had to simply abort the query because I was an hour deep and unsure of how much longer I could wait before I could continue working.  When waiting for the alterations to the analytics table to finish, I wasn't sure I had to wait to make queries on that table until it was finished or not.

4. Tables were not set up in a way where primary keys could easily be created.

5. Constantly finding inconsistencies and missing data within the database.


## Future Goals
If I had more time, I would have liked to attempt to clean the database more thoroughly.  I would have picked apart these tables and reconstructed new ones in order to have a more user-friendly database.  I also would have liked to spend a bit more time exploring the data for trends, but enough time has been spent on this project! 
