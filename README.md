# Spotify SQL Project and Query Optimization

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

### Data Exploration
Before diving into SQL, we understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. 

#### Easy Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### Medium Queries
- More complex queries involving grouping, aggregation functions, and joins.
  
#### Advanced Queries
- Nested subqueries, window functions, CTEs, and performance optimization.

### Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---

## Question Sets

1. Retrieve the names of all tracks that have more than 1 billion streams.
```sql
select track, stream from spotify
where stream < 1000000000;
```
2. List all albums along with their respective artists.
```sql
select distinct album, artist 
from spotify
order by 1;
```
3. Get the total number of comments for tracks where `licensed = TRUE`.
```sql
select sum(comments) as total_comments 
from spotify
where licensed = TRUE;
```
4. Find all tracks that belong to the album type `single`.
```
select track, album_type from spotify
where album_type= 'single';
```
5. Count the total number of tracks by each artist.
```sql
select artist, count(track) as total_number_of_songs
from spotify
group by artist
order by 2;
```
6. Calculate the average danceability of tracks in each album.
```sql
select album, avg(danceability) as average_danceability
from spotify
group by album
order by average_danceability desc;
```
7. Find the top 5 tracks with the highest energy values.
```sql
select track, max(energy) as highest_energy_value
from spotify
group by 1
order by 1 desc
limit 5;
```
8. List all tracks along with their views and likes where `official_video = TRUE`.
```sql
select track,
sum(views)as total_views, 
sum(likes)as total_likes
from spotify
where official_video = 'TRUE'
group by 1;
```
9. For each album, calculate the total views of all associated tracks.
```sql
select album, track, 
sum(views) as total_views
from spotify
group by 1,2
order by 3 desc ;
```
10. Retrieve the track names that have been streamed on Spotify more than YouTube.
```sql
select * from
(select track,
coalesce(sum(case when most_played_on= 'Spotify' then stream End),0) as stream_on_spotify,
coalesce(sum(case when most_played_on= 'Youtube' then stream End),0)as stream_on_Youtube
from spotify
group by 1) as t1
where stream_on_spotify> stream_on_Youtube
and stream_on_Youtube <>0 ;
```

11. Find the top 3 most-viewed tracks for each artist using window functions.
```sql
with rank_artist
as (
select 
artist, 
track,
sum(views) as total_viewed,
dense_rank() over(partition by artist order by sum(views) desc) as rank
from spotify 
group by 1,2
order by 1,3 desc) 
select * from rank_artist
where rank <=3;
```
12. Write a query to find tracks where the liveness score is above the average.
```sql
select track, 
artist, liveness
from spotify
where liveness> (select avg(liveness) as avg_liveness from spotify)
```
13. Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.
```sql
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC
```
   

---

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **8.155 ms**
        - Planning time (P.T.): **100.415 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      [EXPLAIN Before Index] [image](https://github.com/user-attachments/assets/0719ad88-eaa3-42c7-9cf4-4509f7c89932)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
 create index artist_index on spotify (artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.721 ms**
        - Planning time (P.T.): **0.801 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index]![image]![image](https://github.com/user-attachments/assets/38eba09f-7172-4e10-a52f-f0ca9c67469d)

![image](https://github.com/user-attachments/assets/2d472d7c-c043-409b-a78a-eaf7ef15ec87)


- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph]![image](https://github.com/user-attachments/assets/1093a4be-51bb-4112-950f-2f5cae9610c9)


This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---







