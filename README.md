# SQL_olympic_dataset_analysis

## Project Overview

**Project Title**: Olympic history data analysis  
**Level**: Beginner to Advanced 
**Database**: `SQL_olympic_db_project3`

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore, clean, and analyze 126 years of olympic history data. The project involves setting up a olympic history database, performing exploratory data analysis (EDA), and answering specific business questions through SQL queries. This project is for educational purspose by doing data analysis on multiple tables. In this project PostgreSQL 17 has been used.

## Objectives

1. **Set up a olympic history database**: Create and populate a olympic history database with the provided athlete, events, location and medal tally data.
2. **Data Cleaning**: Identify wheather complete data has been imported or not.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Data Analysis**: Use SQL to answer specific analytical questions and derive insights from the onlympic data.

## Project Structure

**Creating a table for athlete biography in SQL_olympic_db_project3 database**
```sql
DROP TABLE IF EXISTS althete_bio;
CREATE TABLE athlete_bio (
	athlete_id VARCHAR,
	name VARCHAR,
	sex VARCHAR,
	born VARCHAR,
	height VARCHAR,
	weight VARCHAR,
	country VARCHAR,
	country_noc VARCHAR
)
```
**Checking if data is imported correctly or not**
```sql
SELECT * FROM athlete_bio
LIMIT 10;
```
**Checking count of all imported entries**
```sql
SELECT count(*) FROM athlete_bio
```
**Creating table for event details**
```sql

DROP TABLE IF EXISTS event_details;
CREATE TABLE event_details (
	edition VARCHAR,
	edition_id VARCHAR,
	country_noc VARCHAR,
	sport VARCHAR,
	event VARCHAR,
	result_id VARCHAR,
	athlete VARCHAR,
	athlete_id VARCHAR,
	pos VARCHAR,
	medal VARCHAR,
	isTeamSport VARCHAR
)
```
**Checking if data is imported correctly or not**
```sql
SELECT * FROM event_details
LIMIT 10;
```
**Checking count of all imported entries**
```sql
SELECT COUNT(*) FROM event_details
```
-- Creating table for event_results

DROP TABLE IF EXISTS event_results;
CREATE TABLE event_results (
	result_id VARCHAR,
	event_title VARCHAR,
	edition VARCHAR,	
	edition_id VARCHAR,
	sport VARCHAR,
	sport_url VARCHAR,
	result_date VARCHAR,
	result_location VARCHAR,
	result_participants VARCHAR,
	result_format VARCHAR,
	result_detail VARCHAR,
	result_description VARCHAR
)

-- Checking if data is imported correctly

SELECT * FROM event_results
LIMIT 10;

-- Checking count of all imported entries

SELECT COUNT(*) FROM event_results

-- Creating table for game summary

DROP TABLE IF EXISTS game_summary;
CREATE TABLE game_summary (
	edition VARCHAR,	
	edition_id VARCHAR,
	edition_url VARCHAR,
	year VARCHAR,
	city VARCHAR,
	country_flag_url VARCHAR,
	country_noc VARCHAR,
	start_date VARCHAR,
	end_date VARCHAR,
	competition_date VARCHAR,
	isHeld VARCHAR
)

-- Checking imported data 

SELECT * FROM game_summary
LIMIT 10;

-- Counting all the imported entries

SELECT COUNT(*) FROM game_summary

-- Creating table for medal tally history

DROP TABLE IF EXISTS medal_tally;
CREATE TABLE medal_tally (
	edition VARCHAR,
	edition_id VARCHAR,
	year VARCHAR,
	country VARCHAR,
	country_noc VARCHAR,
	gold INT,
	silver INT,
	bronze INT,
	total INT
 )

 -- Cheking imported data

 SELECT * FROM medal_tally
 LIMIT 10;

 -- Counting all the entries 

 SELECT COUNT(*) FROM medal_tally

 -- **All five tables have been imported**

 -- ##DATA ANALYSIS

 -- ** Beginner Level(5 questions) **
 -- 1. **List all the unique sports played in the Olympics**

 SELECT DISTINCT(sport) FROM event_details;

 -- 2. **Find the total number of athletes in the dataset.**

 SELECT COUNT(name) FROM athlete_bio;

 -- 3. **Retrieve the top 10 countries with the highest total medal count.**  

 SELECT 
    country, 
    SUM(gold + silver + bronze) AS total_medals
 FROM medal_tally
 GROUP BY country
 ORDER BY total_medals DESC
 LIMIT 10;

-- 4. **Find all events where "Swimming" was played.**

SELECT edition,
		sport
FROM event_details
WHERE sport = 'Swimming'

-- 5. **Count how many times the Olympic Games were held in London.**  

SELECT * FROM game_summary
WHERE city = 'London'

-- ### **Intermediate Level (10 Questions)**  

-- 6. **Find the average height and weight of male and female athletes.**

SELECT 
    sex, 
    ROUND(AVG(height::NUMERIC), 2) AS avg_height, 
    ROUND(AVG(weight::NUMERIC), 2) AS avg_weight
FROM athlete_bio
WHERE height ~ '^[0-9.]+$'   -- Ensures only digits and dots in height
  AND weight ~ '^[0-9.]+$'   -- Ensures only digits and dots in weight
GROUP BY sex;

-- 7. **Get the country that won the most gold medals in any single Olympic edition.**

SELECT 
	edition,
	country,
	MAX(gold) AS gold_medals
FROM medal_tally
GROUP BY edition, country
ORDER BY gold_medals DESC
LIMIT 1;

-- 8. **Find the number of medals won by athletes from 'USA' in the '100m Sprint' event.**

SELECT  
		count(ed.country_noc) AS total_medals
FROM event_details ed
LEFT JOIN event_results er ON ed.result_id = er.result_id
WHERE event_title IN ('100 metres, Men', '100 metres, Women')
	AND ed.country_noc = 'USA'

-- 9. **Find the top 5 athletes who have won the most medals.** 

SELECT 
	ab.athlete_id,
	ab.name,
	COUNT(ed.medal IS NOT NULL) AS total_medal
FROM athlete_bio ab
LEFT JOIN event_details ed ON ab.athlete_id = ed.athlete_id 
GROUP BY ab.athlete_id, ab.name, ed.medal
ORDER BY total_medal DESC
LIMIT 5;

-- 10. **Find the top 5 years with the highest number of participating athletes.**

SELECT 
		gs.year AS year,
		COUNT(ed.athlete) AS participants
FROM event_details ed
LEFT JOIN game_summary gs USING(edition_id)
GROUP BY year
ORDER BY COUNT(ed.athlete) DESC
LIMIT 5;

-- 11. **Retrieve all athletes who have won medals in multiple sports.**

SELECT 
	athlete,
	COUNT(DISTINCT sport)  AS No_of_sports
FROM event_details
WHERE medal IS NOT NULL
GROUP BY athlete
HAVING COUNT(DISTINCT sport) > 1
ORDER BY COUNT(sport) DESC

-- 12. **Get the Olympic Games edition where the most medals were awarded.**  

SELECT 
	edition,
	COUNT(medal) AS no_of_medals
FROM event_details
WHERE medal IS NOT NULL
GROUP BY edition
ORDER BY COUNT(medal) DESC

-- 13. **Find the total number of medals won by each country across all editions.**

SELECT 
	country,
	SUM(total) AS total_medals
FROM medal_tally
GROUP BY country
ORDER BY SUM(total) DESC

-- 14. **Find the most successful athlete (with the most gold medals).**

SELECT 
	athlete,
	COUNT(medal) AS gold_medals
FROM event_details
WHERE medal = 'Gold'
GROUP BY athlete
ORDER BY COUNT(medal) DESC

-- 15. **Find the top 5 Olympic events with the most participants across all editions.**  

SELECT 
	event,
	COUNT(DISTINCT athlete) AS participents
FROM event_details
GROUP BY event
ORDER BY participents DESC
LIMIT 5;

-- 16. **Find the country that improved its medal tally the most from one Olympic edition to the next.**

WITH Medal_Change AS (
    SELECT 
        country_noc, 
        edition_id, 
        total,
        LAG(total) OVER (PARTITION BY country_noc ORDER BY edition_id) AS prev_total
    FROM medal_tally
)
SELECT country_noc, (total - prev_total) AS medal_increase, edition_id
FROM Medal_Change
WHERE prev_total IS NOT NULL
ORDER BY medal_increase DESC
LIMIT 1;

-- 17. **Find the sport with the highest ratio of gold medals to total medals.**

SELECT 
	sport,
	SUM(CASE WHEN medal = 'Gold' THEN 1 ELSE 0
		END) * 1.0 / COUNT(*) AS gold_ratio
FROM event_details
GROUP BY sport
ORDER BY gold_ratio DESC
LIMIT 1;

-- 18. Find the earliest and latest Olympic medalists.

SELECT 							-- Earliest olympic medalist
	ed.athlete, 
	ed.edition
FROM event_details ed
WHERE ed.medal IS NOT NULL
ORDER BY ed.edition ASC
LIMIT 1;

SELECT 							-- Latest olympic medalist
	ed.athlete,
	ed.edition
FROM event_details ed
WHERE ed.medal IS NOT NULL 
ORDER BY ed.edition DESC
LIMIT 1;

-- 19. **Determine if hosting the Olympics helps a country win more medals.**


WITH host_medals AS (
    SELECT 
        gs.country_noc AS host_country,
        gs.edition_id,
		COALESCE(mt.total,0) AS total_medals
    FROM game_summary gs
	LEFT JOIN medal_tally mt ON gs.country_noc = mt.country_noc AND gs.edition_id = mt.edition_id
),
non_host_medals AS (
    SELECT
        mt.country_noc,
        AVG(mt.total) AS avg_medals
    FROM medal_tally mt
    LEFT JOIN host_medals hm  
        ON mt.country_noc = hm.host_country  
        AND mt.edition_id = hm.edition_id  
    WHERE hm.edition_id IS NULL  -- Ensures we exclude hosting years
    GROUP BY mt.country_noc
) 
SELECT 
    hm.host_country,
    ROUND(AVG(hm.total_medals), 2) AS avg_hosting_medals,  
    ROUND(COALESCE(nhm.avg_medals, 0), 2) AS avg_non_hosting_medals,  
    ROUND((AVG(hm.total_medals) - COALESCE(nhm.avg_medals, 0))) AS impact  
FROM host_medals hm
LEFT JOIN non_host_medals nhm  
    ON hm.host_country = nhm.country_noc  
GROUP BY hm.host_country, nhm.avg_medals
ORDER BY impact DESC;

-- 20. ** Find the correlation between athlete height and winning gold medals in Basketball.**

SELECT 
    CASE WHEN medal = 'Gold' THEN 'Gold Medalists' ELSE 'Non-Gold Athletes' END AS category,
    ROUND(AVG(height::FLOAT)) AS avg_height
FROM athlete_bio ab
JOIN event_details ed ON ab.athlete_id = ed.athlete_id
WHERE ed.sport = 'Basketball' AND height SIMILAR TO '[0-9]+(\.[0-9]*)?'  -- Filters valid numeric heights
GROUP BY category;

