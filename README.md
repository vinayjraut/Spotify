# Spotify Advanced SQL Project and Query Optimization
Project Category: Advanced
[Click Here to get Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```
## Project Steps

### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### 4. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Queries are categorized into **easy**, **medium**, and **advanced** levels to help progressively develop SQL proficiency.

#### Easy Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### Medium Queries
- More complex queries involving grouping, aggregation functions, and joins.
  
#### Advanced Queries
- Nested subqueries, window functions, CTEs, and performance optimization.

### 5. Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---
-- Glancing through the data
```sql
SELECT *
FROM spotify
limit 100;
```

## Easy Level
** Retrieve the names of all tracks that have more than 1 billion streams.
```sql
SELECT track
FROM spotify
WHERE stream > 1000000000;
```

** List all albums along with their respective artists.
```sql
SELECT DISTINCT album, artist
FROM spotify;
```
** Get the total number of comments for tracks where licensed = TRUE.
```sql
SELECT sum(comments) as ttl_cmnts_lic_true
FROM spotify
WHERE licensed = 'true';
```
** Find all tracks that belong to the album type single.
```sql
SELECT track
FROM spotify
WHERE album_type ILIKE 'single';
```
** Count the total number of tracks by each artist.
```sql
SELECT artist, COUNT(track) AS ttl_tracks
FROM spotify
GROUP BY 1
ORDER BY ttl_tracks desc;
```
## Medium Level
** Calculate the average danceability of tracks in each album.
```sql
Select album,AVG(danceability) as avg_danceability
FROM spotify
GROUP BY album
ORDER BY 2 DESC;
```

** Find the top 5 tracks with the highest energy values.
```sql
SELECT track, energy
FROM spotify
order by energy desc
LIMIT 5;
```

** List all tracks along with their views and likes where official_video = TRUE.
```sql
SELECT track, 
		SUM(views) AS ttl_views,
		SUM(likes) AS ttl_likes
FROM spotify
WHERE official_video = 'true'
GROUP BY track
ORDER BY 2 DESC
LIMIT 5;
```

** For each album, calculate the total views of all associated tracks.
```sql
SELECT album,
		track,
		SUM(views) as ttl_views
from spotify
GROUP BY 1,2
ORDER BY 3 DESC;
```

** Retrieve the track names that have been streamed on Spotify more than YouTube.
```sql
SELECT *
	FROM
(SELECT track,
		--most_played_on,
		COALESCE(sum(CASE WHEN most_played_on = 'Youtube' THEN stream END),0) AS streamed_on_youtube,
		COALESCE(sum(CASE WHEN most_played_on = 'Spotify' THEN stream END),0) AS streamed_on_spotify
FROM spotify
group by 1) AS t1
WHERE 
	streamed_on_spotify > streamed_on_youtube
	AND
	streamed_on_youtube <> 0;
```

## Advanced Level
** Find the top 3 most-viewed tracks for each artist using window functions.
```sql
SELECT *
from
(SELECT 	
	artist,
	track,
	SUM(views) AS ttl_views,
	DENSE_RANK() OVER (PARTITION BY artist order by SUM(views) desc) as R1
FROM spotify
GROUP BY 1,2
ORDER BY 1,3 DESC
)
where R1 <=3;
```
** Write a query to find tracks where the liveness score is above the average.
```sql
Select track,
		liveness
from spotify
	where liveness > 
	(
	select 
	avg(liveness)
	FROM spotify
	);
```

** Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.
```sql
WITH cte
AS
	(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energy
FROM spotify
GROUP BY 1)
SELECT album,
	highest_energy - lowest_energy as energy_diff
FROM cte
ORDER BY 2 DESC;
```

** Find tracks where the energy-to-liveness ratio is greater than 1.2.
```sql
SELECT 
	DISTINCT track,
	energy,
	liveness,
	energy/liveness AS e2l_ratio
FROM spotify
WHERE 
	liveness > 0
	and 
	energy/liveness > 1.2
order by e2l_ratio;
```

** Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.
```sql
SELECT 
	distinct track,
	views,
	likes,
	SUM(likes) OVER (ORDER BY views ASC) as cumulative_likes
FROM spotify
ORDER BY views;
```

---

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **5.74 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_after_index.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%203.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%202.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%201.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---


## License
This project is licensed under the MIT License.
