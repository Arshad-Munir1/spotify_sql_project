# Spotify Advanced SQL Project using SQL

![Spotify Logo](https://github.com/Arshad-Munir1/spotify_sql_project/blob/main/spotify_logo.jpg)

## Overview
This project involves a comprehensive analysis of a real Spotify dataset using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems and solutions.

## Objectives

Answer the following questions:

## Easy Level
1. Retrieve the names of all tracks that have more than 1 billion streams.
2. List all albums along with their respective artists.
3. Get the total number of comments for tracks where licensed = TRUE.
4. Find all tracks that belong to the album type single.
5. Count the total number of tracks by each artist.
## Medium Level
6. Calculate the average danceability of tracks in each album.
7. Find the top 5 tracks with the highest energy values.
8. List all tracks along with their views and likes where official_video = TRUE.
9. For each album, calculate the total views of all associated tracks.
10. Retrieve the track names that have been streamed on Spotify more than YouTube.
## Advanced Level
11. Find the top 3 most-viewed tracks for each artist using window functions.
12. Write a query to find tracks where the liveness score is above the average.
13. Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.
14. Find tracks where the energy-to-liveness ratio is greater than 1.2
15. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Spotify Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

## Schema

```sql
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
	
Artist 				VARCHAR(255),	
Track				VARCHAR(255),
Album				VARCHAR(255),
Album_type			VARCHAR(50),
Danceability		FLOAT,
Energy				FLOAT,
Loudness			FLOAT,
Speechiness			FLOAT,
Acousticness		FLOAT,
Instrumentalness	FLOAT,	
Liveness			FLOAT,
Valence				FLOAT,	
Tempo				FLOAT,
Duration_min		FLOAT,
Title				VARCHAR(255),
Channel				VARCHAR(255),
Views				FLOAT,
Likes				BIGINT,
Comments			BIGINT,
Licensed			BOOLEAN,
official_video		BOOLEAN,
Stream				BIGINT,
EnergyLiveness		FLOAT,
most_playedon		VARCHAR(50)

);
```

## Business Problems and Solutions

### 1. Retrieve the names of all tracks that have more than 1 billion streams.

```sql
SELECT
	TRACK,
	SUM(VIEWS)
FROM
	SPOTIFY
WHERE
	VIEWS > 1000000000
GROUP BY
	TRACK ORDERBY
	2
```

### 2. List all albums along with their respective artists.

```sql
SELECT
    album,
    STRING_AGG(artist, ', ') AS artists
FROM spotify
GROUP BY album
```

### 3. Get the total number of comments for tracks where licensed = TRUE.

```sql
SELECT
	TRACK,
	SUM(COMMENTS) AS TOTAL_COMMENTS
FROM
	SPOTIFY
WHERE
	LICENSED = TRUE
GROUP BY
	TRACK
ORDER BY 2 DESC
```

### 4. Find all tracks that belong to the album type single.

```sql
SELECT DISTINCT
	(TRACK),
	ALBUM_TYPE
FROM
	SPOTIFY
WHERE
	ALBUM_TYPE = 'single'
```

### 5. Count the total number of tracks by each artist.

```sql
SELECT
	ARTIST,
	COUNT(TRACK)
FROM
	SPOTIFY
GROUP BY
	ARTIST
ORDER BY
	2 DESC
```

### 6. Calculate the average danceability of tracks in each album.

```sql
SELECT
	ALBUM,
	AVG(DANCEABILITY)
FROM
	SPOTIFY
GROUP BY
	ALBUM
ORDER BY
	2 DESC
```

### 7. Find the top 5 tracks with the highest energy values.

```sql
SELECT
	TRACK,
	ENERGY
FROM
	SPOTIFY
ORDER BY
	ENERGY DESC LIMIT
	5
```

### 8. List all tracks along with their views and likes where official_video = TRUE.

```sql
SELECT
	TRACK,
	SUM(VIEWS) AS TOTAL_VIEWS,
	SUM(LIKES) AS TOTAL_LIKES
FROM
	SPOTIFY
WHERE
	OFFICIAL_VIDEO = TRUE
GROUP BY
	1
ORDER BY
	2 DESC
```

### 9. For each album, calculate the total views of all associated tracks.

```sql
SELECT
	ALBUM,
	SUM(VIEWS) AS TOTAL_ALBUM_VIEWS
FROM
	SPOTIFY
GROUP BY
	1
ORDER BY
	2 DESC
```

### 10.Retrieve the track names that have been streamed on Spotify more than YouTube.

```sql
SELECT
	*
FROM
	(
		SELECT
			TRACK,
			COALESCE(
				SUM(
					CASE
						WHEN MOST_PLAYEDON = 'Youtube' THEN STREAM
					END
				),
				0
			) AS STREAMED_ON_YOUTUBE,
			COALESCE(
				SUM(
					CASE
						WHEN MOST_PLAYEDON = 'Spotify' THEN STREAM
					END
				),
				0
			) AS STREAMED_ON_SPOTIFY
		FROM
			SPOTIFY
		GROUP BY
			1
	) AS T1
WHERE
	STREAMED_ON_SPOTIFY > STREAMED_ON_YOUTUBE
	AND STREAMED_ON_YOUTUBE <> 0
```

### 11. Find the top 3 most-viewed tracks for each artist using window functions.

```sql
WITH
	RANKING_ARTIST AS (
		SELECT
			ARTIST,
			TRACK,
			SUM(VIEWS) AS TOTAL_VIEW,
			DENSE_RANK() OVER (
				PARTITION BY
					ARTIST
				ORDER BY
					SUM(VIEWS) DESC
			) AS RANK
		FROM
			SPOTIFY
		GROUP BY
			1,
			2
		ORDER BY
			1,
			3 DESC
	)
SELECT
	*
FROM
	RANKING_ARTIST
WHERE
	RANK <= 3
```

### 12. Write a query to find tracks where the liveness score is above the average.

```sql
SELECT
	ARTIST,
	TRACK,
	LIVENESS
FROM
	SPOTIFY
WHERE
	LIVENESS > (
		SELECT
			AVG(LIVENESS)
		FROM
			SPOTIFY
	)
```

### 13. Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.

```sql
WITH
	CTE AS (
		SELECT
			ALBUM,
			MAX(ENERGY) AS HIGHEST_ENERGY,
			MIN(ENERGY) AS LOWEST_ENERGY
		FROM
			SPOTIFY
		GROUP BY
			1
	)
SELECT
	ALBUM,
	ROUND((HIGHEST_ENERGY - LOWEST_ENERGY)::NUMERIC, 2) AS ENERGY_DIFFERENCE
FROM
	CTE
ORDER BY
	2 DESC
```

### 14. Find tracks where the energy-to-liveness ratio is greater than 1.2.

```sql
SELECT
	TRACK,
	ROUND((ENERGY / LIVENESS)::NUMERIC, 2) AS RATIO
FROM
	SPOTIFY
WHERE
	ENERGY / LIVENESS > 1.2
```

### 15. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions

```sql
SELECT
	TRACK,
	SUM(LIKES) AS TOTAL_LIKES,
	VIEWS
FROM
	SPOTIFY
GROUP BY
	1,
	3
ORDER BY
	3 DESC
```

