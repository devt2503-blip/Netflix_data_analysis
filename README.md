# Netflix_data_analysis 
This project involves a comprehensive analysis of Netflix's movie and TV show data using SQL. The goal is to extract valuable insights and answer key business ques
![netflix_logo](logo.png)

## Objectives

- Analyze the distribution of content types (Movies vs. TV Shows).
- Identify the most common ratings for different content types.
- List and analyze content based on release years, countries, and specific directors.
- Filter and categorize content based on specific keywords and criteria.

## Dataset
The data for this project is sourced from Kaggle:

*Dataset Link*: [Actual_Dataset](netflix_titles.csv)

## Business_problems 
```sql
-- 15 Business Problems & Solutions

1. Count the number of Movies vs TV Shows
2. Find the most common rating for movies and TV shows
3. List all movies released in a specific year (e.g., 2020)
4. Find the top 5 countries with the most content on Netflix
5. Identify the longest movie
6. Find content added in the last 5 years
7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
8. List all TV shows with more than 5 seasons
9. Count the number of content items in each genre
10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!
11. List all movies that are documentaries
12. Find all content without a director
13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.
```

## Schema
```sql
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

SELECT * FROM netflix;
```

## Problems & Solution Query

1. Count the Number of Movies vs TV Shows
 ```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```
2. Find the most common rating for movies and TV shows
```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```
3. List all movies released in a specific year (e.g., 2020)
```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```
4. Find the top 5 countries with the most content on Netflix
```sql
   SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```
5. Identify the longest movie
```sql
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```
6. Find content added in the last 5 years
```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

8. List all TV shows with more than 5 seasons
```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

9. Count the number of content items in each genre
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

10.Find each year and the average numbers of content release in India on netflix. 
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

11. List all movies that are documentaries
```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

12. Find all content without a director
```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.
```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```

## Key Findings
1. Content Dominance: The platform is typically Movie-dominant, with movies often accounting for roughly 70% of the total library compared to TV shows.
2. Geographic Leaders: The United States is the primary content producer, followed by India and the United Kingdom. While the US library is balanced, India’s contribution is heavily skewed toward movies.
3. arget Audience: A significant majority of content is geared toward mature audiences, with TV-MA and TV-14 being the most frequent ratings.
4. Production Trends: Content production saw a massive surge after 2010, peaking around 2018. Recent years show a slight decline, potentially due to the COVID-19 pandemic or a strategic shift toward quality over quantity.
. Viewing Preferences: Most successful movies have a duration between 90–120 minutes, while the vast majority of TV shows (approx. 67%) consist of only one season. 


## In a Nutshell
The analysis indicates that Netflix employs a data-driven content strategy focused on high-volume movie production and strategic regional investments, particularly in India. To maintain its market leadership, Netflix should continue diversifying into international genres and non-English language content to cater to its global subscriber base. Furthermore, the high prevalence of mature-rated content suggests an opportunity to expand into younger audience segments (under 18) to capture underserved markets.
