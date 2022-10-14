# Iowa Liquor Sales Analysis
![Sales Analysis Dashboard](https://i.imgur.com/SkfQRev.png)
![Sales Dashboard Tableau](https://i.imgur.com/1TikTFH.png)

## Overview

Data Source: https://data.iowa.gov/Sales-Distribution/Iowa-Liquor-Sales/m3tr-qhgy

This dataset was obtained from the Iowa Department of Commerce and includes, in their own words:

"This dataset contains the spirits purchase information of Iowa Class “E” liquor licenses by product and date of purchase from January 1, 2012 to current. The dataset can be used to analyze total spirits sales in Iowa of individual products at the store level."

## Goals

The goal of this project is to demonstrate the complete process of data analysis, from ingesting raw data, preparing the data for analysis, visualizing the data in a dashboard, and answering some assumed business metrics that may be requested when presented with sales transaction data. This is not intended to be the end-all-be-all method for how to use these tools as there are a variety of ways to get to the goal. This is just one of the many methods.

Due to the private nature of data I have worked on in my professional career, I chose this dataset to give as close of a representation as possible with my experience working with:

  - Microsoft Excel
  - SQL
  - Power BI
  - Tableau

Through this discovery process, the goal is to gather common business KPIs, including:
  - Weekly / Monthly / Quarterly / Yearly Sales Figures 
  - Total Sales by Liquor Type
  - Total Sales by Store
  - Top Products by County
  - Best Seller by Volume (ml)

## Preparation

The data provided by the Iowa Department of Commerce includes the following fields: Invoice Number, Data, Store Number, Store Name, Address, City, Zip Code, Country Number, County, Category, Vendor Number, Vendor Name, Item Number, Item Description, Pack, Bottle Volume (ml), State Bottle Cost, State Bottle Retail, Bottles Sold, Sale (USD), Volume Sold (Liters), Volume Sold (Gallons). 

Once a copy of the original data is safely stored and backed up, we can begin the process of sorting through the information we have and comparing it to the answers we seek, and removing the information that is not vital to the goal. To this extent, the following columns can be removed: Invoice Number, Pack, State Bottle Cost, State Bottle Retail, Bottles Sold, and Volume Sold (Gallons). Despite being an American myself, it is with great displeasure that I remove the gallons column but keep the liter column - though we mostly measure drinks by liters anyways, so a round of applause for our own consistency between unit measurements.

As this is just an example of the process of analyzing data, I'll assume the goals listed above were requested by business and aim to deliver those results and provided insights based on my findings. The information I seek will provide us with an overview of the sales in Iowa in terms of which stores are most profitable, which products sell the most, and sales trajectories throughout the year to deep dive and find any correlation in sales vs real world events. One such event would be looking at the data with an assumption of "sales will probably increase in February with the Super Bowl", and compare that to actual results. 

Now that I've determined which fields I want to keep and the questions I am looking to answer, I need to clean up the data. In the next sections, I will explain my thought process for standardizing each column of data.

### Date
- I changed the data type to "short date" from "general" for consistency and data type accuracy.

### Store Number
- I changed the data type to "number" from "general" for consistency and data type accuracy.

### Store Name, Address
- I changed the data type to "text" from "general" for consistency and data type accuracy.
- I used Excel's =UPPER() function to standardize the formatting. 
- This field has longitude and latitude coordinates in the address, but Tableau will be unable to process them automatically as they are. Fortunately, in a few quick steps, we can separate these values into their own columns.
    1.	Create three new columns
    2.	Data > Text to Columns > Delimited > Check ‘Space’ > Next > Finish
    3.	Control + H to replace the ‘(‘ and ‘)’ characters.
    4.	Rename column headers to Store Name, Zip Code, Longitude, and Latitude. 

### City
- I changed the data type to "text" from "general" for consistency and data type accuracy.
- I used Excel's =UPPER() function to standardize the formatting. 

### Zip Code
- I changed the data type to "special (zip code)" from "general" for consistency and data type accuracy.

### Longitude, Latitude
- I changed the data type to "special (zip code)" from "general" for consistency and data type accuracy.

### County Number, Category, Vendor Number, Item Number, Bottle Volume (ml)
- I changed the data type to "number" from "general" for consistency and data type accuracy.

### Category
- There are currently 105 different categories for products. After standardizing the data format and consolidating inconsistencies, for a higher-level view, I created a new column called "Category Type", that can categorize the products at a higher-level, that can be expanded upon if desired. These categories are whisky, vodka, rum, liqueurs, tequila, schnapps, brandy, gin, cocktails, scotch, and others. I used the following code to parse the Category Name field for these 11 keywords and filled in my new column with the appropriate category. 

```
=IF(ISNUMBER(SEARCH("Vodka", cell#)), "VODKA", "")
```

## Inconsistent Data

Standardized formats are ideal for consistency and being able to quickly determine errors in the data. Tableau and Power BI, to some extent, struggle with inconsistencies and will treat "Polk" and "POLK" as two different values. 

After applying a consistent format where possible, I created a pivot table to get a count of the unique values. A quick Google search shows that Iowa has 99 counties - with that in mind, I can quickly skim my list of values and if there are more than 99, I can quickly determine that something is wrong. Skimming through this list, several errors appear:

```
1. Buena Vist - 18 Transactions
2. Buena Vista - 9824 Transactions
3. Cerro Gord – 7 Transaction
4. Cerro Gordo – 24208 Transactions
5. Pottawatta – 111 Transactions
6. Pottawattamie – 34,747 Transactions
[...]
```

Typos are a quick fix with a simple find and replace. Now that those replacements have been made, I can refresh the data in my pivot table, which leads me to a new issue: blank data. Of the 1,048,575 records found, 2,597 don’t have a County associated. Fortunately, these were largely big chunks that could easily be rectified, including 651 records in Belmond (Wright County), and 391 records in Guttenberg (Clayton County). For other stores, occasionally there is a location attached to the end of the store’s name, such as “Hy-Vee Food Store #1 / Ottumwa”. This store in particular appears in a lot of the incomplete column, so one way of automating this would be use a formula like the one below. Since we know that Ottumwa is part of Wapello County (thanks Google!), this formula will parse the Store Name field and if it finds a match of the string “Ottumwa”, then set the County field to “Wapello”, otherwise leave it blank. 

```
=IF(ISNUMBER(SEARCH("Ottumwa", cell#)), "WAPELLO", "")
```

With this formula and strategy, while time consuming, it will provide us with the most accurate version of the data. Some of the remaining inputs have no discernible county, city, address, and will end up filtered from the results. Regardless, having several hundred rows of “dirty data” in a data set that has over a million rows will be well within the expected confidence intervals when making decisions. 

### Date
When calculating the total sales amount for each year, the total sales amounts are all within a similar range from 2012-2015. 2018 only has data from the first quarter, while 2017 only has data from the 4th quarter. For consistency and tracking trends, we’ll only look at information from the four-year period where sales data is available throughout the entire year (2012-2015), then use forecasting to see how our model compares to the real data available. 

## Analysis

Now that the data has been cleaned, I can begin analyzing the data. I'll create a series of pivot tables that will answer these questions.

### Weekly / Monthly / Quarterly / Yearly Sales Figures 
-WIP-

### Total Sales by Liquor Type

Over the course of the four-year period from 2012-2015, whisky accounted for just over 33% of all sales, with vodka in second accounting for nearly 24% of sales. In every single county, whisky is clearly the preferred drink, while the runner up switches between vodka or rum, depending on the county. 

![Total Liquor Sales By Product Category](https://i.imgur.com/SBLS6o9.png)

### Total Sales by Store

As there are several thousand stores, I'll break this down into the top 10 stores in terms of total sales. 

![Top10StoresByTotalSales](https://i.imgur.com/wdGxuvi.png)

### Top Products by County
In this dashboard, I decided to make use of the slicer tool to be able to check for the top selling products by county, but also one step further to see how these numbers have changed over time. You could take a look at the sales trends and if the data showed, for example, that whisky was on a downward trend year on year but tequila numbers were skyrocketing, you might think consider having more tequila products in strategic spots if you're a store owner, or a new line of products to sell. 

![TopProductsByCountySlicerDashboard](https://i.imgur.com/dLGSvbO.png)

### Best Seller by Volume (ml)

![TopSellingBottlesByVolume](https://i.imgur.com/9KzPiQS.png)

## Visualizing 

There are two main visualizations created within Excel that highlight our findings.

The first is a visualization of the top selling categories by county, which can be further narrowed down to specific years - so for example, you could find out which category of product was the least popular in 2013 in Polk County (scotch), or the most popular in Black Hawk County in 2014 (whisky). 

![chart of Iowa liquor sales](https://i.imgur.com/EuEc8g5.png)

The next visualization is of a dashboard highlighting an overview of total sales per year (top-left), a heatmap of transaction numbers (top right), the top performing stores by county in terms of total sales (interactive, bottom left), and the total sales by product category (bottom right). Iowa loves whisky!

![dashboard of various liquor statistics of Iowa](https://i.imgur.com/nvMwCqN.png)

## SQL

I am working in Microsoft SQL Server Management Studio (SSMS) for the following section and will use these queries to form the visualizations in Power BI.

![What are the top 10 most popular bottle sizes in terms of total sales?](https://i.imgur.com/VsDUl5Q.png)

![Min/Max/AVG Sales daily](https://i.imgur.com/lLQJvZu.png)

![Top Performing Stores](https://i.imgur.com/x6oDmjt.png)

![SalesIn2014](https://i.imgur.com/zHJY9c0.png)

## Tableau

I used Tableau public for this deep dive and the link to the dashboard can be found [here](https://public.tableau.com/views/IowaLiquorSales_16620129866610/IowaLiquorSales?:language=en-US&:display_count=n&:origin=viz_share_link).

![Sales Dashboard](https://i.imgur.com/1TikTFH.png) 


## Power BI
-WIP-

# Summary

I hope you found this valuable and that it demonstrated how 'dirty data' can be cleaned and transformed into clean, usable data that can affect business decisions. Otherwise, I hope you learned something new about Excel and about drinking trends and traditions in Iowa. 
