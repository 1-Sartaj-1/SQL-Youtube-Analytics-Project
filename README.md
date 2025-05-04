# Welcome to my Youtube Data Analytics Project!!!

After creating the youtube_trends (or any name you prefer) database , we can begin by creating the table and defining the table's columns and their data types as follows: 
```sql
CREATE TABLE Youtube_data (`Rank` INT,
Video TEXT, 
Video_views VARCHAR(20),
Likes VARCHAR(20),
Dislikes VARCHAR(20), 
Category VARCHAR(50), 
Published INT)
```
Notice how we have backtics (`) when we define the Rank column, that is because RANK is a reserved keyword. Also, notice that even though we have data type for Video_views, likes, dislikes to be VARCHAR(20) even though they represent numeric values. The reason for that is the commas separating the numbers, hence for smooth import of the CSV data into the table, we need to be careful of these things. Therefore, it's really important to take a look at the raw csv file to see what kind of data does the columns contain.
