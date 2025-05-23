# Welcome to my Youtube Data Analytics Project!!!

We'll use MySQL to look at data from the top-1000-trending-youtube-videos.csv file (which you can find above!) and see what we can find out. We'll go through setup, cleaning, and analysis, using SQL features like:

Setup: **CREATE TABLE, PRIMARY KEY, AUTO_INCREMENT, INT, TEXT, VARCHAR, BIGINT**.

Cleaning: **UPDATE, SET, TRIM(), REPLACE(), ALTER TABLE MODIFY COLUMN**, handling **NULL**.

Basic Analysis: **SELECT, WHERE, COUNT(), AVG(), MAX(), GROUP BY, HAVING, ORDER BY, LIMIT**.

Advanced: **Window Functions (AVG() OVER, PARTITION BY), CTEs (WITH), JOIN, CASE WHEN/THEN/ELSE, LOWER(), LIKE, Stored Procedures (CREATE PROCEDURE, DELIMITER, CALL**).

## Step 1: Database and Table Setup
After creating the **Youtube_Analytics** (or any name you prefer) database , we can begin by creating the table (**Youtube_data** here)and defining the table's columns and their data types as follows: 
```sql
CREATE TABLE Youtube_data (
video_id INT PRIMARY KEY AUTO_INCREMENT,
`Rank` INT,
Video TEXT, 
Video_views VARCHAR(20),
Likes VARCHAR(20),
Dislikes VARCHAR(20), 
Category VARCHAR(50), 
Published INT)
```
Notice how we have backtics (`) when we define the Rank column, that is because **RANK** is a reserved keyword. Also, notice that even though we have data type for Video_views, likes, dislikes to be VARCHAR(20) even though they represent numeric values. The reason for that is the commas separating the numbers, hence for smooth import of the CSV data into the table, we need to be careful of these things. Therefore, it's really important to take a look at the raw csv file to see what kind of data does the columns contain.

## Step 2: Data Cleaning and Preparation
Firstly, we need to handle leading and trailing whitespace, so we can do the following:
```sql
UPDATE Youtube_data
SET 
    Video = TRIM(Video),
    Video_views = TRIM(Video_views),
    Likes = TRIM(Likes),
    Dislikes = TRIM(Dislikes),
    Category = TRIM(Category);
```
Notice how we've only used **TRIM()** on String data types such as TEXT and VARCHAR only.
After trimming, if any values were originally blank or just spaces, they might now be empty strings (''). We can set them to **NULL** (representing missing data) or 0. We'll use NULL here:
```sql
UPDATE Youtube_data SET Video = NULL WHERE Video = '';
UPDATE Youtube_data SET Video_views = NULL WHERE Video_views = '';
UPDATE Youtube_data SET Likes = NULL WHERE Likes = '';
UPDATE Youtube_data SET Dislikes = NULL WHERE Dislikes = '';
UPDATE Youtube_data SET Category = NULL WHERE Category = '';
```
Now, we need to handle the commas within the string columns (Video_views, Likes, Dislikes) and convert them to **BIGINT** for optimal calculation.
```sql
UPDATE Youtube_data
SET
    Video_views = REPLACE(Video_views, ',', ''),
    Likes = REPLACE(Likes, ',', ''),
    Dislikes = REPLACE(Dislikes, ',', '');
```
Next, we will convert the **VARCHAR** data type to **BIGINT** data type since we've trimmed the whitespace and replaced the commas within the values.
```sql
ALTER TABLE Youtube_data
MODIFY COLUMN Video_views BIGINT,
MODIFY COLUMN Likes BIGINT,
MODIFY COLUMN Dislikes BIGINT;
```
Now, with the data cleaning our table looks like this:
![Alt text](Step-2.png)

## Step 3: Exploratory Data Analysis 
Now that our data is clean and properly formatted, let's dive in and explore!

Let's begin with looking at rank and see if we have all the 1000 videos as the csv file says.
```sql
SELECT COUNT(`Rank`) AS total_videos 
FROM youtube_data;
```
![Alt text](count_rank.png)

This could mean that we're missing a rank or might have duplicate ranks. Also, if we look at our primary key column, **video_id**, we can see that the total number of rows is 999. 

![Alt text](count_rank_2.png)

We can check for duplicates:
```sql
SELECT `Rank`, COUNT(*)
FROM youtube_data
GROUP BY `Rank`
HAVING COUNT(*) > 1;
```
It gives us no output, which means there are no duplicates. Hence, we can proceed to next step with 999 rows confidently.

Now, let's move our attention towards the Category column. 

We can use GROUP BY to see how many unique categories do we have:
```sql
SELECT Category 
FROM youtube_data 
GROUP BY Category;
```
We can use the following query to see the total number of trending videos in each category:
```sql
SELECT Category, COUNT(*) AS video_count
FROM youtube_data
GROUP BY Category
ORDER BY video_count DESC;
```
![Alt text](category_res.png)

The result highlight 'Music' and 'People & Blogs' as the most frequent categories in this trending dataset.

Now, we can see statistics such as top five videos with maximum number of likes, dislikes and views:
```sql
SELECT Video, Video_views
FROM Youtube_data
ORDER BY Video_views DESC 
LIMIT 5;
```
![Alt text](views.png)
```sql
SELECT `Rank`, Video, Dislikes
FROM Youtube_data
ORDER BY Dislikes DESC 
LIMIT 5;
```
![Alt text](dislikes.png)
```sql
SELECT `Rank`, Video, Likes
FROM Youtube_data
ORDER BY Likes DESC 
LIMIT 5;
```
![Alt text](likes.png)

Music videos and major global artists appear frequently among the top performers in views, likes as well as dislikes.

We can see the peak view count achieved within each category by the following query. 
```sql
SELECT Category, 
MAX(Video_views) AS views
FROM Youtube_data
WHERE Category IS NOT NULL
GROUP BY Category
ORDER BY views DESC;
```
![Alt text](cat_views.png)

Next, we can see categories which got the most amount of likes on average. We will use the HAVING clause because it filters the groups based on the calculated average after aggregation.
```sql
SELECT Category, AVG(Likes) AS average_likes
FROM Youtube_data
WHERE Category IS NOT NULL
GROUP BY Category
HAVING AVG(Likes) > 100000
ORDER BY average_likes DESC;
```
![Alt text](cat_likes.png)

Moving on to Publish column, let's see the total number of videos and their total views per publication year:
```sql
SELECT Published, COUNT(*) AS total_videos, SUM(Video_views) AS total_views_for_year 
FROM Youtube_data
WHERE Published IS NOT NULL
GROUP BY Published
ORDER BY Published DESC;
```
![Alt text](publish.png)

## Step 4: Advanced SQL Techniques for Deeper Insights
Let's leverage more powerful SQL features to extract even richer insights that might be harder to get with basic queries.

If we want to see the average likes of the videos per category, we can use a window function as follows:
```sql
SELECT Category, Video, Video_views, AVG(Likes) OVER(PARTITION BY Category ) AS avg_likes
FROM Youtube_data
WHERE Category IS NOT NULL
ORDER BY Category, Video_views DESC;
```
![Alt text](Window_function.png)

Further, to show each video's title, its view count, and the total view count for its entire category, we can use a CTE(Common table Expression) and compare the tables using JOIN as follows:
```sql
WITH Cat_Tot_Views AS
(
SELECT Category, SUM(Video_views) AS cat_views, COUNT(*) AS vid_num    
FROM Youtube_data
WHERE Category IS NOT NULL AND Video_views IS NOT NULL 
GROUP BY Category 
)
SELECT Youtube_data.Video, Youtube_data.Category, Cat_Tot_Views.cat_views, Cat_Tot_Views.vid_num 
FROM Youtube_data
INNER JOIN Cat_Tot_Views ON Youtube_data.Category = Cat_Tot_Views.Category 
ORDER BY Youtube_data.Category, Youtube_data.Video_views DESC; 
```
![Alt text](CTE.png)

We can use CASE statements to see which videos with which animal were trending, dogs or cats?
```sql
SELECT Video, Video_views,  Category ,
CASE
    WHEN LOWER(Video) LIKE '%dog%' THEN 'Dog videos'
    WHEN LOWER(Video) LIKE '%cat%' THEN 'Cat videos'
    ELSE 'Other'
END AS DogVsCat 
FROM Youtube_data
ORDER BY DogVsCat;
```
![Alt text](case.png)

Also, if we want a quick method to lookup a Rank and the related details of that Rank, we can create a procedure and store it in the database itself as follows:
```sql
DELIMITER $$
CREATE PROCEDURE extract_video_rank(video_rank INT)
BEGIN
SELECT *
FROM Youtube_data
WHERE `Rank` = video_rank;
END $$
DELIMITER ;
```
After executing this query, we'll have a new procedure stored under Youtube_Analytics database named as **extract_video_rank**.

![Alt text](stored_proc.png)

We can use this procedure anytime we want by using the CALL clause.

![Alt text](call_store_proc.png)

## Conclusion
So we covered everything mentioned in the intro - setting up the table, cleaning the data, exploring with aggregates and grouping, and using advanced SQL like Window Functions, CTEs, CASE, and Stored Procedures to analyze the YouTube data in MySQL. I hope you learnt something and enjoyed going through this project as much as me!!

