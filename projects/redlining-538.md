---
layout: page
title: Examining Redlining in the U.S. with fivethirtyeight Data
permalink:  /projects/redlining-538/
---

# Examining Redlining in the United States
## Using Data from fivethirtyeight

This is an expansion on a school project I did in the summer of 2022 about redlining in the United States, a practice used in the early- to mid-20th century that prevented investment in areas that were deemed hazardous or otherwise not suited for financial support. The effects of redlining persist today. All data comes from the 2020 United States census.  

## Introduction and Preprocessing

To access and prepare the data for use, follow these steps.

Step 1: The dataset on redlining in the United States can be accessed via this permalink: https://github.com/fivethirtyeight/data/blob/b22a21b264162ad0b5d8954b02e0bca5ab782113/redlining/metro-grades.csv

Step 2: Because of inconsistency with the csv file format, it is important to select the “copy” option from the page and paste the data in a new Excel file, rather than attempting to read the csv from the website directly into RStudio. The main reason for this is that the csv file on the website compresses all the data into a single column, which is easy to deal with in Excel.

Step 3: The data will be copied as a single column rather separated into a properly formatted table. If this occurs, the steps below can organize the data into tabular form.

3a. In Excel, first do a “Find and Replace” for the character string “, “ (note the space) and replace with “-“. This is necessary because the first column contains a city followed by a state, separated by a comma, and we do not want to separate the city and state into distinct columns.

Step 3b: Highlight column A and select “Text to Columns” in the Data tab at the top of the page.

Step 3c: Follow the prompts, choosing to delimit the data by comma.

Step 4: Save the csv file locally and copy the file path for importing into RStudio. The code for importing into RStudio is as follows, with the column classes explicitly laid out. Note that the file path given is local to my computer, and it will need to be updated when saved in a different location.

