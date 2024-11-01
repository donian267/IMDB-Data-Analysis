# IMDB Profitable movie analysis

### Project Overview
This data analysis project was created to provide insights that will lead to a commercially successful movie. By analyzing  various aspects of this IMDB dataset, I'm seeking to find trends, make data-driven recommendations, and gain a better understanding of the path that leads to a profitable movie.

### Data Source

The dataset was created by 'Chidambara Raju G' from kaggle. https://www.kaggle.com/datasets/rajugc/imdb-top-250-movies-dataset/data

### Tools
 - SQL Server
 - Tableau

### Data Cleaning/Preparation
- Make a copy of the dataset
- Standardizing the data
- Check for errors, outliers, and Inconsistencies
- check for blank values

### Exploratory Data analysis
Exploring the data to answer key questions such as:
-Which movies have the highest box office?
-Which movies have the best return on investment?
What trends lead to a movie having high box office numbers?

### Data Analysis

```SQL
-- creating a new table to work on
create table movies
like `imdb top 250 movies`

insert movies
select * 
from `imdb top 250 movies`

--Creating a columnn that shows return on investment
alter table movies
add ROI int

update movies
set ROI = (box_office/budget) * 100

--Standardizing the data
update movies
set certificate = 'PG'
where certificate like '%gp%'

update movies
set certificate = 'R'
where certificate like '%13+%' or certificate like '%18+%' or certificate like 'unrated'

-- Some movies had money that needed to translate to dollars
select *
from movies
where roi <= 1

update movies 
set budget = 15749478
where name like '%Princess Mononoke%'

update movies 
set budget = 6570272
where name like '%3 Idiots%'

update movies 
set box_office = 31500000
where name like '%Barry Lyndon%'

--creating a row for movies with unusable data
alter table movies
add compromised nvarchar

update movies
set compromised = case
					when year < 1968 then 'compromised'
					when name = 'Dersu Uzala' then 'compromised'
					when name = 'Princess Mononoke' then 'compromised'
					when budget is null then 'compromised'
					when box_office is null then 'compromised'
						else 'good'
						end
				  from movies

-- movies with the best return on investment(international movies and movies before 1968 didn't have reliable box office numbers)

select top 20  year, name, roi, budget, box_office
from movies
where compromised = 'good'
order by roi desc




-- checking to see all of the different movie genre types
SELECT genre, COUNT(*) AS count
FROM (
    SELECT TRIM(value) AS genre
    FROM movies
    CROSS APPLY STRING_SPLIT(genre, ',')
) AS split_genres
GROUP BY genre
ORDER BY count DESC
 
 --checking how well movies did based on their certificate
select certificate, round(avg(budget),2) as average_budget,  round(avg(box_office),2) as average_box_office , avg(roi) as average_return_on_investment
from movies
where compromised = 'good'
group by certificate
order by average_box_office desc

--checking the success of lower budget movies vs higher budget movies

select round(avg(budget),2) as avg_budget, round(avg(box_office),2) as avg_box_office
from movies

select name, box_office, ROI, year
from movies
where budget > 38383595.72 and compromised = 'good'
order by box_office desc

select name, box_office, ROI, year
from movies
where budget < 38383595.72 and compromised = 'good'
order by box_office desc

--Checking the average year to see how that affects results

select round(avg(year),0)
from movies
where box_office > 38383595.72 and compromised = 'good'

select round(avg(year),0)
from movies
where budget > 38383595.72 and compromised = 'good'

select round(avg(year),0)
from movies
where box_office < 38383595.72 and compromised = 'good'

select round(avg(year),0)
from movies
where budget < 38383595.72 and compromised = 'good'

--Use CTE get an average box office per genre
WITH SplitGenres AS (
    SELECT TRIM(value) AS genre, box_office, budget
    FROM movies
    CROSS APPLY STRING_SPLIT(genre, ',')
	where compromised = 'good'
),
GenreStats AS (
    SELECT genre, 
           COUNT(*) AS count,
           round(AVG(budget),2) AS avg_budget, round(AVG(box_office),2) AS avg_box_office
    FROM SplitGenres
    GROUP BY genre
)
SELECT genre, count, avg_budget, avg_box_office
FROM GenreStats
ORDER BY avg_box_office DESC;

--Checking to see if higher rated movies make more money

select rating, name, box_office, ROI
from movies
where compromised = 'good'
order by box_office desc

--checking which movie had the highest box office per year
WITH rMovies AS (
    SELECT 
        name, 
        box_office, 
        year,
		genre,
        RANK() OVER (PARTITION BY year ORDER BY box_office desc) as rank
    FROM movies
	where compromised = 'good'
)
SELECT 
    name, 
    box_office, 
    year,
	genre
FROM rMovies
WHERE rank = 1;


WITH SplitGenres AS (
    SELECT TRIM(value) AS genre, box_office, budget, year
    FROM movies
    CROSS APPLY STRING_SPLIT(genre, ',')
    WHERE compromised = 'good' AND year >= YEAR(GETDATE()) -60
),
GenreStats AS (
    SELECT genre, 
	       year,
           COUNT(*) AS count,
           ROUND(AVG(budget), 2) AS avg_budget, 
           ROUND(AVG(box_office), 2) AS avg_box_office
		   
    FROM SplitGenres
    GROUP BY genre, year
)
SELECT genre,year, avg_budget, avg_box_office
FROM GenreStats
ORDER BY year, avg_box_office DESC;
```
