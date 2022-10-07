# Iowa Liquor Sales Analysis
![chart of Iowa liquor sales](https://i.imgur.com/EuEc8g5.png)
![dashboard of various liquor statistics of Iowa](https://i.imgur.com/nvMwCqN.png)

## Overview

Data Source: https://data.iowa.gov/Sales-Distribution/Iowa-Liquor-Sales/m3tr-qhgy

This dataset was obtained from the Iowa Department of Commerce and includes, in their own words:

"This dataset contains the spirits purchase information of Iowa Class “E” liquor licenses by product and date of purchase from January 1, 2012 to current. The dataset can be used to analyze total spirits sales in Iowa of individual products at the store level."

The final product can be viewed here: https://docs.google.com/spreadsheets/d/1eNYj3kV3PIXwyAwS86uoIcxIOKGwXbXu/edit?usp=sharing&ouid=108376608193069191851&rtpof=true&sd=true

## Goals

The goal of this project is to demonstrate the complete process of analyzing data, from ingesting raw data, preparing the data for analysis, and visualizing the data in a dashboard in Microsoft Power BI and Tableau. 

Through this discovery process, the goal is to gather common business KPIs, including:
  - Weekly / Monthly / Quarterly / Yearly Sales Figures 
  - Total Sales by Liquor Type
  - Total Sales by Store
  - Top Products by County
  - Top Performing Stores
  - Top Vendors/Distributors 
  - Best Seller by Volume (ml)

## Preparation

The data provided by the Iowa Department of Commerce includes the following fields: Invoice Number, Data, Store Number, Store Name, Address, City, Zip Code, Country Number, County, Category, Vendor Number, Vendor Name, Item Number, Item Description, Pack, Bottle Volume (ml), State Bottle Cost, State Bottle Retail, Bottles Sold, Sale (USD), Volume Sold (Liters), Volume Sold (Gallons). 

Once a copy of the original data is safely stored and backed up, we can begin the process of sorting through the information we have and comparing it to the answers we seek, and removing the information that is not vital to the goal. To this extent, the following columns can be removed: Invoice Number, Pack, State Bottle Cost, State Bottle Retail, Bottles Sold, and Volume Sold (Gallons). Despite being an American myself, it is with great displeasure that I remove the gallons column but keep the liter column - though we mostly measure drinks by liters anyways, so a round of applause for our own consistency between unit measurements.

As this is just an example of the process of analyzing data, we'll assume the goals listed above were requested by business and aim to deliver those results and provided insights based on our findings. The information we seek will provided us with an overview of the sales in Iowa in terms of which stores are most profitable, which products sell the most, and sales trajectories throughout the year to deep dive and find any correlation in sales vs real world events. One such event would be looking at the data with an assumption of "sales will probably increase in February with the Super Bowl", and compare that to actual results. 

Now that we've determine which fields we want to keep, we need to clean up the data. Standardized formats are ideal for consistency and being able to quickly determine errors in the data. Since one of the goals will be the analyze sales via a heat map, I started by using Excel's =UPPER() function to define a standard format as later on in the visualization phase, Tableau and Power BI, to some extent, struggle with inconsistencies and will treat "Polk" and "POLK" as two different values. 

After applying a consistent format, I created a pivot table to get a count of the unique values. A quick Google search shows that Iowa has 99 counties - with that in mind, I can quickly skim my list of values and if there are more than 99, I can quickly determine that something is wrong. Skimming through this list, several errors appear:

1. Buena Vist - 18 Transactions
2. Buena Vista - 9824 Transactions
3.	Cerro Gord – 7 Transaction
4.	Cerro Gordo – 24208 Transactions
5.	Pottawatta – 111 Transactions
6.	Pottawattamie – 34,747 Transactions
[...]

Typos are a quick fix with a simple find and replace. Now that those replacements have been made, I can refresh the data in my pivot table, which leads me to a new issue: blank data. Of the 1,048,575 records found, 2,597 don’t have a County associated. Fortunately, these were largely big chunks that could easily be rectified, including 651 records in Belmond (Wright County), and 391 records in Guttenberg (Clayton County). For other stores, occasionally there is a location attached to the end of the store’s name, such as “Hy-Vee Food Store #1 / Ottumwa”. This store in particular appears in a lot of the incomplete column, so one way of automating this would be use a formula like the one below.

=IF(ISNUMBER(SEARCH("Ottumwa", cell#)), "WAPELLO", "")

Since we know that Ottumwa is part of Wapello County (thanks Google!), this formula will parse the Store Name field and if it finds a match of the string “Ottumwa”, then set the County field to “Wapello”, otherwise leave it blank. 

With this formula and strategy, while time consuming, it will provide us with the most accurate version of the data. Some of the remaining inputs have no discernible county, city, address, and will end up filtered from the results. Regardless, having several hundred rows of “dirty data” in a data set that has over a million rows will be well within the expected confidence intervals when making decisions. 

Store Location
This field has longitude and latitude coordinates in the address, but Tableau will be unable to process them automatically as they are. Fortunately, in a few quick steps, we can separate these values into their own columns.
1.	Create three new columns
2.	Data > Text to Columns > Delimited > Check ‘Space’ > Next > Finish
3.	Control + H to replace the ‘(‘ and ‘)’ characters.
4.	Rename column headers.

Date
When calculating the total sales amount for each year, the total sales amounts are all within a similar range from 2012-2015. 2018 only has data from the first quarter, while 2017 only has data from the 4th quarter.  For consistency and tracking trends, we’ll only look at information from the four-year period where sales data is available throughout the entire year, then use forecasting to see how our model compares to the real data available.  

Category Name
There are currently 105 different categories for products. After standardizing the data format and consolidating inconsistencies, for a higher level view, I created a new column that can categorize the products at a higher-level, that can be expanded upon if desired. These categories are whisky, vodka, rum, liqueurs, tequila, schnapps, brandy, gin, cocktails, scotch, and others. 

## Analysis

Now that the data has been cleaned, we can begin analyzing the data. 

When looking at the counties with the most sales data, Des Moines is, by far and away, the king (queen?) of number of transactions, and with Des Moines being the most populated county in Iowa, this comes as no surprise. 

As for the most popular bottle sold, the data shows that the most popular bottle sizes are:

1. 750ml - 3,822,469 transactions 
2. 1000ml - 2,373,791 transactions
3. 1750ml - 2,011,095 transactions 
4. 375ml - 1,004,819 transactions
5. 200ml - 630,235 transactions

The total sales corresponding to this discovery paints an even clearer picture:
Bottle Size   Total Sales
750ml         $53,002,264
1000ml        $34,216,179
1750ml        $33,216,847
375ml          $6,204,906
200ml          $2,393,216

## Visualizing 

There are two main visualizations created within Excel that highlight our findings.

The first is a visualization of the top selling categories by county, which can be further narrowed down to specific years - so for example, you could find out which category of product was the least popular in 2013 in Polk County (scotch), or the most popular in Black Hawk County in 2014 (whisky). 

![chart of Iowa liquor sales](https://i.imgur.com/EuEc8g5.png)

The next visualization is of a dashboard highlighting an overview of total sales per year (top-left), a heatmap of transaction numbers (top right), the top performing stores by county in terms of total sales (interactive, bottom left), and the total sales by product category (bottom right). Iowa loves whisky!

![dashboard of various liquor statistics of Iowa](https://i.imgur.com/nvMwCqN.png)

Key Takeaways:
- Iowa loves whisky, with vodka as a close second. 
- 750ml in the most popular seller in terms of bottle size.
- December is the most popular month for purchasing alcohol - probably because there is only one way through the holiday season. 

## Summary

I hope you found this valuable and that it demonstrated how 'dirty data' can be cleaned and transformed into clean, usable data that can affect business decisions. Otherwise, I hope you learned something new about Excel and learned something about drinking trends and traditions in Iowa. 
