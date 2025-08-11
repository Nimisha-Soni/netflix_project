# Netflix movies and TV shows Data Analysis using SQL
![Netflix Logo](https://github.com/Nimisha-Soni/netflix_project/blob/main/logo.png)

## Overview:
For the purpose of this study, a thorough SQL analysis of Netflix's movie and TV program data is being conducted.  The objective is to use the dataset to derive insightful information and provide answers to a range of business queries.  A thorough description of the project's goals, business issues, solutions, findings, and conclusions may be found in the README that follows.

## Objectives:
-Examine how different content kinds are distributed (movies vs. TV series).
-Determine the most popular movie and TV program ratings.
-Sort and evaluate material according to nations, years of release, and lengths.
-Examine and group information according to particular standards and keywords.

## DataSet:
The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:**[Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema:
```sql
--NETFLIX PROJECT
CREATE TABLE netflix(
show_id VARCHAR(6),
type VARCHAR(10),
title VARCHAR(150),
director VARCHAR(208),
casts VARCHAR(1000),
country VARCHAR(150),
date_added VARCHAR(50),
release_year INT,
rating VARCHAR(10),
duration VARCHAR(15),
listed_in VARCHAR(100),
description VARCHAR(250)
);

```

## Business Problems and Solutions

## --1. Count the number of Movies vs TV Shows
```sql
Select type, 
COUNT(*) AS total_content
from netflix
Group by type;
```

## --2. Find the most common rating for movies and TV shows
```sql
Select 
  type,
  rating
from(
Select 
type,
rating,
COUNT(*),
Rank() OVER(Partition by type Order by COUNT(*) DESC) as ranking
from netflix
Group by 1,2
)
   as t1
Where ranking = 1
```

## --3. List all movies released in a specific year (e.g., 2020)
```sql
select * from netflix
where type = 'Movie'
AND
release_year = 2020;
```
## --4. Find the top 5 countries with the most content on Netflix
```sql
--First we converted the country from strint to array and then we un-nested it
Select  UNNEST(String_to_Array(country,',')) as new_country,
count(show_id) as total_count
from netflix
group by 1
order by 2 DESC
Limit(5)
```

## --5. Identify the longest movie
```sql
Select * from netflix
  where type = 'Movie'
  And
  duration = (Select MAX(duration) from netflix);
```

## --6. Find content added in the last 5 years
```sql
Select 
   *
from netflix
where 
   To_date(date_added, 'Month DD, YYYY') >= Current_date - Interval '5 years'
```

## --7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
```sql
Select * from netflix
where director ILIKE '%Rajiv Chilaka%'
```

## --8. List all TV shows with more than 5 seasons
```sql
--:: this is a type caster. Here it convert String to numeric
Select 
  *
From netflix 
where type = 'TV Show'
  AND  SPLIT_PART(duration,' ', 1):: numeric> 5

## --9. Count the number of content items in each genre
Select UNNEST(STRING_TO_ARRAY(listed_in,',')) AS genre,
Count(show_id) as total_count
From netflix
Group by 1
```

## --10.Find each year and the average numbers of content release in India on netflix.
## --return top 5 year with highest avg content release!
```sql
Select 
   Extract(Year from TO_DATE(date_added , 'Month DD, YYYY')) as year,
   Count(*) as yearly,
   ROUND(
      Count(*)::numeric/(Select count(*) from netflix where country = 'India')::numeric *100,2)
	  as avg_content_per_year
from netflix
where country = 'India'
Group by 1
```

## --11. List all movies that are documentaries
```sql
Select * from netflix 
where listed_in ILIKE '%Documentaries%'
```
## --12. Find all content without a director
```sql
Select * from netflix where director IS NULL
```

## --13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
Select * 
from netflix 
where 
       casts ILIKE '%salman khan%'
	   And
	   release_year > Extract(Year from Current_date)-10
```
## --14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
Select
--casts,show_id,
UNNEST(STRING_TO_ARRAY(casts,',')) as actors,
COUNT(*) as total_content
FROM netflix
where country ILIKE '%India%'
group by 1
order by 2 DESC
LIMIT 10
```

## --15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
## --the description field. Label content containing these keywords as 'Bad' and all other 
## --content as 'Good'. Count how many items fall into each category.
```sql
With new_table
AS(
Select 
* ,
  CASE
  WHEN
       description ILIKE '%kill%'
	   OR
       description ILIKE '%violent%' then 'BAD_CONTENT'
	   else 'good_content'

  End category   
from netflix
)
Select category,count(*) as total_content
From new_table
Group by 1
```
## Findings and Conclusion

- **Content Distribution:** The dataset includes a wide variety of TV series and films with different genres and ratings.
 - **Common Ratings:** Knowledge of the most popular ratings helps determine who the content is intended for.
 **Geographical Insights:** Regional content distribution is highlighted by the top nations and the average content releases by India.
 **Content Categorization:** Sorting content according to particular keywords aids in comprehending the type of content that is accessible on Netflix.
