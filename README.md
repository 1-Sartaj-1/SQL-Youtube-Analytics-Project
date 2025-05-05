# Welcome to my Youtube Data Analytics Project!!!

## Step 1: Database and Table Setup
After creating the youtube_trends (or any name you prefer) database , we can begin by creating the table and defining the table's columns and their data types as follows: 
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
Notice how we have backtics (`) when we define the Rank column, that is because RANK is a reserved keyword. Also, notice that even though we have data type for Video_views, likes, dislikes to be VARCHAR(20) even though they represent numeric values. The reason for that is the commas separating the numbers, hence for smooth import of the CSV data into the table, we need to be careful of these things. Therefore, it's really important to take a look at the raw csv file to see what kind of data does the columns contain.

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
Notice how we've only used TRIM() on String data types such as TEXT and VARCHAR only.
After trimming, if any values were originally blank or just spaces, they might now be empty strings (''). We can set them to NULL (representing missing data) or 0. We'll use NULL here:
```sql
UPDATE Youtube_data SET Video = NULL WHERE Video = '';
UPDATE Youtube_data SET Video_views = NULL WHERE Video_views = '';
UPDATE Youtube_data SET Likes = NULL WHERE Likes = '';
UPDATE Youtube_data SET Dislikes = NULL WHERE Dislikes = '';
UPDATE Youtube_data SET Category = NULL WHERE Category = '';
```
Now, we need to handle the commas within the string columns (Video_views, Likes, Dislikes) and convert them to BIGINT for optimal calculation.
```sql
UPDATE Youtube_data
SET
    Video_views = REPLACE(Video_views, ',', ''),
    Likes = REPLACE(Likes, ',', ''),
    Dislikes = REPLACE(Dislikes, ',', '');
```
Next, we will convert the VARCHAR data type to BIGINT data type since we've trimmed the whitespace and replaced the commas within the values.
```sql
ALTER TABLE Youtube_data
MODIFY COLUMN Video_views BIGINT,
MODIFY COLUMN Likes BIGINT,
MODIFY COLUMN Dislikes BIGINT;
```
Now, with the data cleaning our table looks like this:
![Alt text](Step-2.png)

## Step 3: Exploratory Data Analysis 
Let's begin with looking at rank and see if we have all the 1000 videos as the csv file says.
```sql
SELECT COUNT(`Rank`) AS total_videos 
FROM youtube_data;
```
![Alt text](count-rank.png)

This could mean that we're missing a rank or might have duplicate ranks. Also, looking at however to be sure let's check:
```sql
SELECT `Rank`, COUNT(*)
FROM youtube_data
GROUP BY `Rank`
HAVING COUNT(*) > 1;
```
It gives no output, which means there are no duplicates. Hence, we can proceed to next step with 999 rows.
