# Netflix Movies and TV Shows Data Analysis using SQL

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Problems and Solutions

### Creating Table
```sql
create table netflix (
    show_id text primary key,
    type varchar(20),
    title text,
    director text,
    casts text,
    country text,
    date_added date,
    release_year int,
    rating varchar(10),
    duration varchar(20),
    listed_in text,
    description text
);
```

### 1. Count the Number of Movies vs TV Shows
```sql
select 
	type, count(*)
from 
	netflix
group by 1;
```

### 2. Find the Most Common Rating for Movies and TV Shows
```sql
with rating_cte as
(
	select type, rating, count(*), rank() over(partition by type order by count(*) desc) as most_common_rating
	from netflix
	group by 1, 2
	order by 1
)
select 
	type, rating
from 
	rating_cte
where 
	most_common_rating = 1;
```

### 3. List All Movies Released in a Specific Year (e.g., 2020)
```sql
select * 
from 
	netflix
where 
	type = 'Movie' and release_year = '2020';
```

### 4. Find the Top 5 Countries with the Most Content on Netflix
```sql
select * 
from
(
	select unnest(string_to_array(country, ',')) as country, count(*) as total_content
	from netflix
	group by 1
)
where 
	country is not null
order by 
	total_content desc
limit 5;
```

### 5. Identify the Longest Movie
```sql
select *
from 
	netflix
where 
	type = 'Movie'
order by 
	split_part(duration, ' ', 1)::int desc;
```

### 6. Find Content Added in the Last 5 Years
```sql
select * 
from 
	netflix
where 
	date_added >= current_date - interval '5 years';
```

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql
select * 
from
(
	select *, unnest(string_to_array(director, ',')) as director_name
	from netflix
)
where 
	director_name = 'Rajiv Chilaka';
```

### 8. List All TV Shows with More Than 5 Seasons
```sql
select * 
from 
	netflix
where 
	type = 'TV Show'
and 
	split_part(duration, ' ', 1)::int > 5;
```

### 9. Count the Number of Content Items in Each Genre
```sql
select 
	trim(unnest(string_to_array(listed_in, ','))) as genre, 
	count(*) as total_content
from 
	netflix
group by 1
order by 2 desc;
```

### 10. Find the number of content released each year in India on netflix.
```sql
select 
	extract(year from date_added), 
	count(show_id)
from 
	netflix
where 
	country like '%India%'
group by 1
order by 1;
```

### 11. List All Movies that are Documentaries
```sql
select * 
from 
	netflix
where 
	type = 'Movie'
and 
	listed_in like '%Documentaries%';
```

### 12. Find All Content Without a Director
```sql
select * 
from 
	netflix
where 
	director is null;
```

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years
```sql
select * 
from 
	netflix 
where 
	"cast" like '%Salman Khan%'
and 
	release_year >= extract (year from current_date) - 10;
```

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies / TV Shows Produced in UK
```sql
select 
	unnest(string_to_array("cast", ',')) as actors, 
	count(*) as appearances
from 
	netflix
where 
	country like '%United Kingdom%'
group by 1
order by 2 desc
limit 10;
```

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
```sql
with categorized_content_cte as
(
select *, 
	case 
		when description  ~* '\mKill'  or description ~* '\mViolence' then 'Bad'
		else 'Good' 
	end as category
from netflix
)
select category, count(*)
from categorized_content_cte
group by 1;
```
