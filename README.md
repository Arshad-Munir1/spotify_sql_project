# Spotify Advanced SQL Project using SQL

![Spotify Logo](https://github.com/Arshad-Munir1/spotify_sql_project/blob/main/spotify_logo.jpg)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyse the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyse content based on release years, countries, and durations.
- Explore and categorise content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
	show_id			VARCHAR(6),
	type			VARCHAR(10),
	title			VARCHAR(150),
	director 		VARCHAR(208),	
	casts			VARCHAR(1000),
	country			VARCHAR(150),
	date_added 		VARCHAR(50),	
	release_year 	INT,	
	rating			VARCHAR(10),
	duration		VARCHAR(15),
	listed_in		VARCHAR(100),
	description 	VARCHAR(250)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT
	TYPE,
	COUNT(*) AS TOTAL_CONTETNT
FROM
	NETFLIX
GROUP BY
	TYPE;	
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
SELECT
	TYPE,
	RATING,
	TOTAL_CONTENT
FROM
	(
		SELECT
			TYPE,
			RATING,
			COUNT(*) AS TOTAL_CONTENT,
			RANK() OVER (
				PARTITION BY
					TYPE
				ORDER BY
					COUNT(*) DESC
			) AS RANKING
		FROM
			NETFLIX
		GROUP BY
			TYPE,
			RATING
	) AS T1
WHERE
	RANKING = 1
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT
	TITLE,
	RELEASE_YEAR
FROM
	NETFLIX
WHERE
	TYPE = 'Movie'
	AND RELEASE_YEAR = 2020
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT
	UNNEST(STRING_TO_ARRAY(COUNTRY, ', ')) AS NEW_COUNTRY,
	COUNT(*) AS TOTAL_CONTENT
FROM
	NETFLIX
GROUP BY
	1
ORDER BY
	2 DESC LIMIT
	5
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
SELECT
	*
FROM
	NETFLIX
WHERE
	TYPE = 'Movie'
	AND DURATION = (
		SELECT
			MAX(DURATION)
		FROM
			NETFLIX
	)
```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT
	*
	FROM
	NETFLIX
WHERE
	TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT
	*
FROM
	NETFLIX
WHERE
	DIRECTOR ILIKE '%Rajiv Chilaka%'
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT
	*
FROM
	NETFLIX
WHERE
	TYPE = 'TV Show'
	AND SPLIT_PART(DURATION, ' ', 1)::NUMERIC > 5
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT
	UNNEST(STRING_TO_ARRAY(listed_in, ', ')) AS NEW_GENRE,
	COUNT(*) AS TOTAL_GENRE
FROM
	NETFLIX
GROUP BY
	1
ORDER BY
	2 DESC 
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

```sql
SELECT
	EXTRACT(
		YEAR
		FROM
			TO_DATE(DATE_ADDED, 'Month DD, YYYY')
	) AS YEAR,
	COUNT(*) AS YEARLY_CONTENT,
	ROUND(
		COUNT(*)::NUMERIC / (
			SELECT
				COUNT(*)
			FROM
				NETFLIX
			WHERE
				COUNTRY LIKE '%India%'
		)::NUMERIC * 100,
		2
	) AS AVG_CONTENT_PER_YEAR
FROM
	NETFLIX
WHERE
	COUNTRY LIKE '%India%'
GROUP BY
	1
```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
SELECT
	*
FROM
	NETFLIX
WHERE
	TYPE = 'Movie'
	AND LISTED_IN ILIKE '%Documentaries%'
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
SELECT
	*
FROM
	NETFLIX
WHERE
	DIRECTOR IS NULL
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
SELECT
	*
FROM
	NETFLIX
WHERE
	CASTS ILIKE '%Salman Khan%'
	AND RELEASE_YEAR > EXTRACT(
		YEAR
		FROM
			CURRENT_DATE
	) - 10
```

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
SELECT
	UNNEST(STRING_TO_ARRAY(CASTS, ', ')) AS NEW_ACTOR,
	COUNT(*)
FROM
	NETFLIX
WHERE
	COUNTRY ILIKE '%India%'
GROUP BY
	NEW_ACTOR
ORDER BY
	2 DESC
LIMIT
	10
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorise Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql
WITH
	NEW_TABLE AS (
		SELECT
			*,
			CASE
				WHEN DESCRIPTION ~* '\mkill(s|ed|ing)?\M'
				OR DESCRIPTION ILIKE '%voilence%' THEN 'Bad_content'
				ELSE 'Good_content'
			END CATEGORY
		FROM
			NETFLIX
	)
SELECT
	CATEGORY,
	COUNT(*) AS TOTAL_CONTENT
FROM
	NEW_TABLE
GROUP BY
	CATEGORY
```

**Objective:** Categorise content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorisation:** Categorising content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
