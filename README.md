# Iowa Liquor Sales Analysis
![alt-text][https://i.imgur.com/EuEc8g5.png]

## Overview

Data Source: https://data.iowa.gov/Sales-Distribution/Iowa-Liquor-Sales/m3tr-qhgy

This dataset was obtained from the Iowa Department of Commerce and includes, in their own words:

"This dataset contains the spirits purchase information of Iowa Class “E” liquor licensees by product and date of purchase from January 1, 2012 to current. The dataset can be used to analyze total spirits sales in Iowa of individual products at the store level."

## Goals

The goal of this project is to demonstrate the the complete process of analyzing data, from ingesting raw data, preparing the data for analysis, and visualizing the data in a dashboard in Microsoft Power BI and Tableau. 

Through this discovery process, the goal is to gather common business KPIs, including:
  - Weekly / Monthly / Quarterly / Yearly Sales Figures 
  - Total Sales by Liquor Type
  - Total Sales by Store
  
In addition to these questions, several dashboards will be created, including:
  - Heatmap of Sales
  - Store KPI Overview

## Preparation

Standardized formats are ideal for consistency and being able to quickly determine errors in the data. Since one of the goals will be the analyze sales via a heat map, I started by using Excel's =UPPER() function to define a standard format as later on in the visualization phase, Tableau and Power BI, to some extent, struggle with inconsistencies and will treat "Polk" and "POLK" as two different values. 

After applying a consistent format, I created a pivot table to get a count of the unique values. A quick Google search shows that Iowa has 99 counties - with that in mind, I can quickly skim my list of values and if there are more than 99, I can quickly determine that something is wrong. Skimming through this list, several errors appear:

1. Buena Vist - 18 Transactions
2. Buena Vista - 9824 Transactions
3.	Cerro Gord – 7 Transaction
4.	Cerro Gordo – 24208 Transactions
5.	Pottawatta – 111 Transactions
6.	Pottawattamie – 34,747 Transactions
[...]

Typos are a quick fix with a simple find and replace. Now that those replacements have been made, I can refresh the data in my pivot table, which leads me to a new issue: blank data. Of the 1,048,575 records found, 2,597 don’t have a County associated. Fortunately, these were largely big chunks that could easily be rectified, including 651 records in Belmond (Wright County), and 391 records in Guttenberg (Clayton County).

For other stores, occasionally there is a location attached to the end of the store’s name, such as “Hy-Vee Food Store #1 / Ottumwa”. This store in particular appears in a lot of the incomplete column, so one way of automating this would be use a formula like the one below.

=IF(ISNUMBER(SEARCH("Ottumwa", D595344)), "WAPELLO", "")

Since we know that Ottumwa is part of Wapello County (thanks Google!), this formula will parse the Store Name field and if it finds a match of the string “Ottumwa”, then set the County field to “Wapello”, otherwise leave it blank. 

With this formula and strategy, while time consuming, it will provide us with the most accurate version of the data. Some of the remaining inputs have no discernible county, city, address, and will end up filtered from the results. Regardless, having several hundred rows of “dirty data” in a data set that has over a million rows will be well within the expected confidence intervals when making decisions. 

Store Location
This field has longitude and latitude coordinates in the address, but Tableau will be unable to process them automatically as they are. Fortunately, in a few quick steps, we can separate these values into their own columns.
1.	Create three new columns
2.	Data > Text to Columns > Delimited > Check ‘Space’ > Next > Finish
3.	Control + H to replace the ‘(‘ and ‘)’ characters.
4.	Rename column headers.

Date
When calculating the total sales amount for each year, the total sales amounts are all within a similar range from 2012-2015. 2018 only has data from the first quarter, while 2017 only has data from the 4th quarter.  For consistency and tracking trends, we’ll only look at information from the four year period where sales data is available throughout the entire year, then use forecasting to see how our model compares to the real data available.  

Category Name
There are currently 105 different categories for products. After standardizing the data format and consolidating inconsistencies, for a higher level view, I created a new column that can categorize the products at a higher level, that can be expanded upon if desired. These categories are whisky, vodka, rum, liqueurs, tequila, schnapps, brandy, gin, cocktails, scotch, and others. 

## Analysis

## Modeling

## Managing 

## Visualizing 


