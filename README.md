
# Olympic History Data Analysis Project

This project involves cleaning and analyzing data in MySQL; from 120 years of Olympic history, focusing on trends in athlete participation. The dataset, obtained from Kaggle, contains information on athletes and events, covering both Summer and Winter Olympics.

## Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Data Exploration](#data-exploration)
- [Analysis Questions](#analysis-questions)
- [SQL Queries](#sql-queries)
- [Results](#results)
- [Getting Started](#getting-started)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## Project Overview

The **Olympic History Data Analysis Project** aims to explore and analyze trends in the Olympic Games over 120 years. This project utilizes a comprehensive dataset to answer specific questions about athlete participation, performance, and demographic trends.

## Dataset

The dataset used for this project, `athlete_events_raw.xlsx`, contains the following:

- **ID:** Unique identifier for each athlete
- **Name:** Athlete's name
- **Sex:** Gender (M or F)
- **Age:** Age at the time of the Olympics
- **Height:** Athlete's height (in centimeters)
- **Weight:** Athlete's weight (in kilograms)
- **Team:** Name of the team or country
- **NOC:** National Olympic Committee 3-letter code
- **Games:** Year and season of the Olympics
- **Year:** Year of the Olympics
- **Season:** Season (Summer or Winter)
- **City:** Host city
- **Sport:** Sport category
- **Event:** Specific event
- **Medal:** Medal won (Gold, Silver, Bronze, or NA)

The dataset comprises 271,116 rows and 15 columns, representing individual athlete events across different Olympic Games.

The dataset used for this project, `noc_regions.xlsx`, contains the following:

- **NOC:** National Olympic Committee 3-letter code
- **Region:** Corresponding NOC Region

## Data Exploration

Before diving into the analysis, the dataset was explored to understand its structure, identify missing values, and perform necessary data cleaning steps. Key insights from this exploration phase include:

- Distribution of athletes by gender
- Age distribution of participants
- Trends in athlete height and weight over the years
- Participation trends by country and sport

## Analysis Questions

The project aims to answer the following questions:

1. How many olympic games have been held?
```
select count(distinct games) as total_games from athlete;
```
2. List down all Olympics games held so far.
```
select distinct games as all_games from athlete;
```
3. Mention the total no of nations who participated in each olympics game?
```
select games, count(distinct noc) as count from athlete group by games;
```
4. Which year saw the highest and lowest no of countries participating in olympics?
```
select 
 (select games from athlete group by games order by count(distinct noc) asc limit 1) as lowest, 
 (select games from athlete group by games order by count(distinct noc) desc limit 1) as highest from athlete limit 1;
```
5. Which nation has participated in all of the olympic games?
```
select max(athlete_noc_regions.region) as countries from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc group by athlete.noc having count(distinct games) = (select count(distinct games) from athlete);
```
6. Identify the sport which was played in all summer olympics.
```
select sport from athlete where season = 'summer' group by sport having count(distinct year) = (select count(distinct year) from athlete where season = 'summer');
```
7. Which Sports were just played only once in the olympics?
```
select sport, count(distinct games) as no_of_games, min(games) from athlete group by sport having no_of_games = 1;
```
8. Fetch the total no of sports played in each olympic games.
```
select games, count(distinct sport) as count from athlete group by games order by count desc;
```
9. Fetch details of the oldest athletes to win a gold medal.
```
select * from athlete where medal = 'gold' order by age desc;
```
10. Find the Ratio of male and female athletes participated in all olympic games.
```
select concat(1, ':', round((select count(distinct name) from athlete where sex = 'M')/(select count(distinct name) from athlete where sex = 'F'), 2)) as ratio 
from athlete limit 1;
```
11. Fetch the top 5 athletes who have won the most gold medals.
```
select name, count(medal) as total_medals from athlete where medal = 'gold' group by name order by count(medal) desc limit 5;
```
12. Fetch the top 5 athletes who have won the most medals (gold/silver/bronze).
```
select name, count(medal) as total_medals from athlete where medal in ('gold', 'silver', 'bronze') group by name order by count(medal) desc limit 5;
```
13. Fetch the top 5 most successful countries in olympics. Success is defined by no of medals won.
```
select max(athlete_noc_regions.region) as countries, count(medal) as total_medals from athlete left join athlete_noc_regions on athlete.noc = 
athlete_noc_regions.noc where medal in ('gold', 'silver', 'bronze') group by athlete.noc order by count(medal) desc limit 5;
```
14. List down total gold, silver and broze medals won by each country.
```
select region as country, 
 sum(case when medal = 'gold' then 1 else 0 end) as gold, 
 sum(case when medal = 'silver' then 1 else 0 end) as silver, 
 sum(case when medal = 'bronze' then 1 else 0 end) as bronze 
 from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc group by region order by gold desc;
```
15. List down total gold, silver and broze medals won by each country corresponding to each olympic games.
```
select games, region as country, 
 sum(case when medal = 'gold' then 1 else 0 end) as gold, 
 sum(case when medal = 'silver' then 1 else 0 end) as silver, 
 sum(case when medal = 'bronze' then 1 else 0 end) as bronze 
 from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc group by games, region order by games asc, region asc;
```
16. Identify which country won the most gold, most silver and most bronze medals in each olympic games.
```
select games, 
 max(case when medal = 'gold' then region end) as gold_country, 
 max(case when medal = 'silver' then region end) as silver_country, 
 max(case when medal = 'bronze' then region end) as bronze_country from (select games, region, medal, count(*) as medal_count,
 row_number() over (partition by games, medal order by count(*) desc) as rank_medals 
 from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc 
 where medal in ('gold', 'silver', 'bronze') group by games, region, medal) as ranked_country 
 where rank_medals = 1 group by games order by games;
```
17. Identify which country won the most gold, most silver, most bronze medals and the most medals in each olympic games.
```
select games, 
 (select region from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc where medal = 'gold' and a.games = games group by region 
order by count(id) desc limit 1) as gold_country, 
 (select region from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc where medal = 'silver' and a.games = games group by region 
order by count(id) desc limit 1) as silver_country, 
 (select region from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc where medal = 'bronze' and a.games = games group by 
region order by count(id) desc limit 1) as bronze_country, 
 (select region from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc where medal in ('gold', 'silver', 'bronze') and a.games = 
games group by region order by count(id) desc limit 1) as most_medal 
 from (select distinct games from athlete) as a 
 order by games;
```
18. Which countries have never won gold medal but have won silver/bronze medals?
```
select country 
 from 
 (select region as country, 
 sum(case when medal = 'gold' then 1 else 0 end) as gold, 
 sum(case when medal = 'silver' then 1 else 0 end) as silver, 
 sum(case when medal = 'bronze' then 1 else 0 end) as bronze 
 from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc group by region order by region asc) as new_athlete 
 where gold = 0;
```
19. In which Sport/event, India has won highest medals.
```
select sport, count(*) as no_of_medals from athlete where noc = 'IND' and medal in ('gold', 'silver', 'bronze') group by sport limit 1;
```
20. Break down all olympic games where india won medal for Hockey and how many medals in each olympic games.
```
select region, sport, games, count(medal) as medal_count from athlete left join athlete_noc_regions on athlete.noc = athlete_noc_regions.noc where region = 
'india' and sport = 'hockey' and medal in ('gold', 'silver', 'bronze') group by games order by medal_count desc;
```
## SQL Queries

The analysis was conducted using MySQL. The SQL queries used are mentioned with the questions above.

## Results

The results of the analysis provide insights into:

- The growth in the number of participants over the years
- Gender-based trends in Olympic participation
- Medal distribution across different countries
- Changes in athlete demographics over time
- The evolution of sports in the Olympics

## Getting Started

### Prerequisites

To run this project, you will need:

- [MySQL](https://dev.mysql.com/downloads/mysql/)
- Dataset- From the repository or from the [Kaggle](https://www.kaggle.com/datasets/heesoo37/120-years-of-olympic-history-athletes-and-results) website (optional)
- A spreadsheet application (e.g., MS Excel) for initial data cleaning

### Installation

1. **Clone the repository:**

   ```bash
   git clone https://github.com/RohitPradhan108/Olympic_History_DataAnalysis_Project.git
   ```

2. **Set up the MySQL database:**

   Import the dataset into your MySQL environment.

   ```sql
   CREATE DATABASE olympic_history;
   USE olympic_history;
   SOURCE path/to/athlete_events.csv;
   ```

3. **Run SQL queries:**

   Execute the provided SQL queries to perform the analysis.

### Usage

- Use the SQL queries to analyze trends in Olympic participation.
- Modify the queries or dataset to explore further insights or test new hypotheses.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Thanks to [Kaggle](https://www.kaggle.com/) for providing the dataset.
- Special thanks to the open-source community for tools and libraries that made this project possible.
