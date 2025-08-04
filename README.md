# Netflix Movies and TV Shows Data Analysis using SQL

![Netflix Logo](https://github.com/devanshiverma10/Netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

### 1. Count the number of Movies vs TV Shows

```sql
SELECT type, COUNT(*) as total_content
from netflix
group by type
```
	
### 2. Find the most common rating for movies and TV shows

```sql
select
	type,
	rating
from
(
	select 
		type, 
		rating,
		count(*),
		rank() over(partition by type order by count(*) desc) as ranking
	from netflix
	group by 1, 2
) as t1
	where ranking=1
```	
	
### 3. List all movies released in a specific year (e.g., 2020)

```sql
--filter 2020
--movies

select * from netflix
	where 
	type='Movie'
	and
	release_year=2020
```
	
### 4. Find the top 5 countries with the most content on Netflix

```sql
select 
	unnest (string_to_array(country, ', ')) as new_country,
	count(showid) as total_content
from netflix
group by 1
order by 2 desc
limit 5
```	
	
### 5. Identify the longest movie

```sql
select * from 
 (select distinct title as movie,
  split_part(duration,' ',1):: numeric as duration 
  from netflix
  where type ='Movie') as subquery
where duration = (select max(split_part(duration,' ',1):: numeric ) from netflix)
```
		
### 6. Find content added in the last 5 years

```sql
select *
from netflix
where
	to_date(date_added, 'Month DD,YYYY') >= current_date - interval '5years'
```	
			
## 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

```sql
select * from netflix
where director ilike '%Rajiv Chilaka%'
```	
	
### 8. List all TV shows with more than 5 seasons

```sql
select * from netflix
where 
	type = 'TV Show'
	and
	split_part(duration, ' ', 1)::numeric > 5
```
	
### 9. Count the number of content items in each genre

```sql
select 
	unnest(string_to_array(listed_in, ',')) as genre,
	count(showid) as total_content
from netflix
group by 1
```	

	
### 10.Find each year and the average numbers of content release in India on netflix. 
     return top 5 year with highest avg content release!


```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```
	
	
### 11. List all movies that are documentaries

```sql
select * from netflix
where listed_in ilike '%documentaries%'
```	

	
### 12. Find all content without a director

```sql
select * from netflix
where director is null
```
	
### 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

```sql
select * from netflix
where 
	casts ilike '%Salman Khan%'
	and
	release_year > extract(year from CURRENT_DATE) - 10
```

### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

```sql
select 
--showid,
--casts,
unnest(string_to_array(casts,',')) as actors,
count(*) as total_content
from netflix
where country ilike '%india%'
group by 1
order by 2 desc
limit 10
```	

### 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
### the description field. Label content containing these keywords as 'Bad' and all other 
### content as 'Good'. Count how many items fall into each category.

```sql
with new_table
as 
(
select
*,
	case 
	when 
	   description ilike '%kill%' or
	   description ilike '%violence%' then 'Bad Content'
	   else 'Good Content'
	end category
from netflix
)
select 
	category,
	count(*) as total_content
from new_table
group by 1
```	


## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.


