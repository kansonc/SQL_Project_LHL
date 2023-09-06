# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals

### This project takes 5 csv files, 
extracts them into a PostgreSQL graphic application called pgAdmin,
transforms into proper data types and applying proper cleaning, and
loads into a local database making proper ERD connections.

### With 5 business Questions (starting_with_questions.md) to answer and 
3 insights gained (sarting_with_data.md), this project gives no context
to the meaning of table, column, or record.  The goal of this project
is to create an understanding of the data despite no guidance and to
the best of my ability, answer the business questions.

### This project is to exclusively use sql from beginning to end and support
sql with PostgreSQL functions.


## Process
### 1. Examine the CSV files
### 2. Import the files into PGAdmin, where with errors given resulted in me needing to import all data as Varchar datatypes
### 3. Running a query to gain summary statistics regarding the values as strings (See png file of zdata_varchar_stats to see a sample of this table), which informs me of what cleaning processes are needed (cleaning_data.md)
### 4. understanding the data prior to transforming data types and cleaning
### 5. creating new tables of clean data
### 6. creating connections in an ERD format to show keys and cardinality
### 7. Analyzing the data to provide answers and insights

## Results
### See md files for starting with questions and starting with data
### Only 81 records have purchases from the all_sessions table.  Difficult to answer questions effectively


## Challenges 
### This data is significantly lacking detail when it comes to purchases and revenue.
### Tables are redundant, only really used all_sessions
### ERD contains 3 unconnected tables (sales_report, analytics, and my created table zdata_varchar_stats)
which are calculated tables not to be used within a relationship capacity.


## Future Goals
### Liase with the people who collected the data to better understand it
### recreate the database to effectively normalize tables
### Properly clean the analytics table in search of more insights



