# [SQL ZOO](https://sqlzoo.net/wiki/SQL_Tutorial) tasks

## [SELECT within SELECT Tutorial](https://sqlzoo.net/wiki/SELECT_within_SELECT_Tutorial):

1) List each country name where the population is larger than that of 'Russia'.

```SQL 
SELECT name FROM world
  WHERE population >
     (SELECT population FROM world
      WHERE name='Russia')
```

2) Show the countries in Europe with a per capita GDP greater than 'United Kingdom'.

```SQL
SELECT name 
FROM world
WHERE continent = 'Europe' and gdp/population > (SELECT gdp/population 
                                                 FROM world 
						 WHERE name= 'United Kingdom')
```

3) List the name and continent of countries in the continents containing either Argentina or Australia. Order by name of the country.

```SQL
SELECT name, continent
FROM world
WHERE continent in (SELECT continent 
		    FROM world 
		    WHERE name in ('Argentina', 'Australia'))
```

4) Which country has a population that is more than United Kingdom but less than Germany? Show the name and the population.

```SQL
SELECT name, population
FROM world 
WHERE population > (SELECT population 
		    FROM world 
                    WHERE name = 'United Kingdom') and 
      population < (SELECT population 
		    FROM world 
		    WHERE name = 'Germany')
```

5) Show the name and the population of each country in Europe. Show the population as a percentage of the population of Germany.

```SQL
SELECT name, CONCAT(ROUND(population/
            (SELECT population 
             FROM world 
             WHERE name = 'Germany'), 0)
             , '%') as percentage
FROM world
WHERE continent = 'Europe' # ALL uses while comparing table and table 
```

6) Which countries have a GDP greater than every country in Europe? [Give the name only.] (Some countries may have NULL gdp values)

```SQL
SELECT name
FROM world
WHERE gdp > ALL(SELECT gdp 
             FROM world 
             WHERE continent = 'Europe')
```

7) Find the largest country (by area) in each continent, show the continent, the name and the area:

```SQL
SELECT continent, name, area FROM world x
  WHERE area >= ALL
    (SELECT area FROM world y
        WHERE y.continent=x.continent
          AND population>0)  # good example of using ALL function just take a look on script to define largest conutry by continent 
```


8) List each continent and the name of the country that comes first alphabetically.

```SQL
SELECT continent, name 
FROM world x
WHERE name <= ALL(SELECT name
                 FROM world y
                 WHERE x.continent = y.continent)
```

9) Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.

```SQL
SELECT name, continent, population 
FROM world x
WHERE continent not in (SELECT continent 
                        FROM world y
                        WHERE population >25000000)
```

10) Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.

```SQL
SELECT name, continent, population FROM world x
WHERE population/3 >= ALL(SELECT population FROM world y
                       WHERE x.continent = y.continent and x.name != y.name)
```





## [SUM and COUNT](https://sqlzoo.net/wiki/SUM_and_COUNT)

1) Show the total population of the world.

```SQL
SELECT SUM(population)
FROM world
```

2) List all the continents - just once each.

```SQL
SELECT DISTINCT continent
FROM world 
````

3) Give the total GDP of Africa

```SQL
SELECT SUM(gdp)
FROM world
WHERE continent = 'Africa'
```

4) How many countries have an area of at least 1000000

```SQL
SELECT COUNT(name)
FROM world 
WHERE area > 1000000
```

5) What is the total population of ('Estonia', 'Latvia', 'Lithuania')

```SQL
SELECT SUM(population)
FROM world
WHERE name in ('Estonia', 'Latvia', 'Lithuania')
```

6) For each continent show the continent and number of countries.

```SQL
SELECT continent, COUNT(name)
FROM world 
GROUP BY continent 
```

7) For each continent show the continent and number of countries with populations of at least 10 million.

```SQL
SELECT continent, COUNT(name)
FROM world
WHERE population > 10000000
GROUP BY continent 
```

8) List the continents that have a total population of at least 100 million.

```SQL 
WITH table AS(SELECT continent, SUM(population) AS total
              FROM world
              GROUP BY continent
              HAVING total >= 10000000)
SELECT continent 
FROM table
```

## [The JOIN operation](https://sqlzoo.net/wiki/The_JOIN_operation)

1) Modify it to show the matchid and player name for all goals scored by Germany. To identify German players, check for: teamid = 'GER'

``` SQL
SELECT matchid, player 
FROM goal 
WHERE teamid LIKE '%GER'
```

2) Show id, stadium, team1, team2 for just game 1012

```SQL
SELECT id,stadium,team1,team2
FROM game
WHERE id = 1012
```

3) Modify it to show the player, teamid, stadium and mdate for every German goal.

```SQL
SELECT player, teamid, stadium, mdate
FROM game JOIN goal ON (id=matchid)
WHERE teamid like '%GER'
```

4) Show the team1, team2 and player for every goal scored by a player called Mario player LIKE 'Mario%'

```SQL
SELECT team1, team2, player
FROM game JOIN goal ON (id=matchid)
WHERE player like 'Mario%'
```

5) Show player, teamid, coach, gtime for all goals scored in the first 10 minutes gtime<=10

```SQL 
SELECT player, teamid,coach, gtime
FROM goal JOIN eteam ON (teamid = id)
WHERE gtime<=10
```

6) List the dates of the matches and the name of the team in which 'Fernando Santos' was the team1 coach.

```SQL
SELECT mdate, teamname
FROM game JOIN eteam ON (game.team1 = eteam.id)
WHERE coach = 'Fernando Santos'
```

7) List the player for every goal scored in a game where the stadium was 'National Stadium, Warsaw'

```SQL
SELECT player 
FROM goal JOIN game ON (matchid = id)
WHERE stadium = 'National Stadium, Warsaw'
```

8) Instead show the name of all players who scored a goal against Germany.

```SQL
SELECT DISTINCT player
FROM game JOIN goal ON id = matchid
WHERE (team1 = 'GER' or team2 = 'GER') and teamid != 'GER'
```

9) Show teamname and the total number of goals scored.

```SQL
SELECT teamname, COUNT(player)
FROM eteam JOIN goal ON id=teamid
GROUP BY teamname
ORDER BY teamname
```

10) Show the stadium and the number of goals scored in each stadium.

```SQL
SELECT stadium, COUNT(player)
FROM game JOIN goal ON id = matchid
GROUP BY stadium 
```

11) For every match involving 'POL', show the matchid, date and the number of goals scored.

```SQL
SELECT matchid,mdate, COUNT(player)
FROM game JOIN goal ON matchid = id 
WHERE (team1 = 'POL' OR team2 = 'POL')
GROUP BY matchid, mdate
```

12) For every match where 'GER' scored, show matchid, match date and the number of goals scored by 'GER'

```SQL
SELECT matchid, mdate, COUNT(player)
FROM goal JOIN game ON (game.id = goal.matchid)
WHERE teamid = 'GER'
GROUP BY matchid, mdate
```

13) List every match with the goals scored by each team as shown. This will use "CASE WHEN" which has not been explained in any previous exercise

```SQL
SELECT mdate, team1,
SUM(CASE WHEN teamid = team1 THEN 1 ELSE 0 END) AS score1,
team2,
SUM(CASE WHEN teamid = team2 THEN 1 ELSE 0 END) AS score2 
FROM game JOIN goal ON (id = matchid)
GROUP BY mdate,team1,team2
ORDER BY mdate, team1, team2
```

## [More JOIN operations](https://sqlzoo.net/wiki/More_JOIN_operations)

1) List the films where the yr is 1962 [Show id, title]

```SQL
SELECT id, title
FROM movie
WHERE yr=1962
```

2) Give year of 'Citizen Kane'.
```SQL
SELECT yr
FROM movie 
WHERE title = 'Citizen Kane'
```

3) List all of the Star Trek movies, include the id, title and yr (all of these movies include the words Star Trek in the title). Order results by year.

```SQL 
SELECT id, title, yr
FROM movie
WHERE title like 'Star Trek%'
```

4) What id number does the actor 'Glenn Close' have?

```SQL
SELECT id
FEOM actor 
WHERE name = 'Glenn Close'
```

5) What is the id of the film 'Casablanca'

```SQL
SELECT id 
FROM movie 
WHERE title = 'Casablanca'
```

6) Obtain the cast list for 'Casablanca'.

```SQL 
SELECT name
FROM actor JOIN casting ON actor.id = casting.actorid
WHERE casting.movieid = 27
```

7) Obtain the cast list for the film 'Alien'

```SQL
SELECT name
FROM actor JOIN casting ON actor.id = casting.actorid 
           JOIN movie ON casting.movieid = movie.id
WHERE movie.title = 'Alien'
```

8) List the films in which 'Harrison Ford' has appeared

```SQL
SELECT title 
FROM actor JOIN casting ON actor.id = casting.actorid 
           JOIN movie ON casting.movieid = movie.id
WHERE actor.name = 'Harrison Ford'
```

9)List the films where 'Harrison Ford' has appeared - but not in the starring role. [Note: the ord field of casting gives the position of the actor. If ord=1 then this actor is in the starring role]

```SQL
SELECT title 
FROM actor JOIN casting ON actor.id = casting.actorid 
           JOIN movie ON casting.movieid = movie.id
WHERE actor.name = 'Harrison Ford' and casting.ord != 1
```

10) List the films together with the leading star for all 1962 films.

```SQL
SELECT movie.title, actor.name
FROM actor JOIN casting ON actor.id = casting.actorid 
           JOIN movie ON casting.movieid = movie.id
WHERE casting.ord = 1 and movie.yr = 1962
```

11) Which were the busiest years for 'Rock Hudson', show the year and the number of movies he made each year for any year in which he made more than 2 movies.

```SQL
SELECT yr,COUNT(title) 
FROM movie JOIN casting ON movie.id=movieid
           JOIN actor   ON actorid=actor.id
WHERE name='Doris Day'
GROUP BY yr
HAVING COUNT(title) > 1
```

12)List the film title and the leading actor for all of the films 'Julie Andrews' played in.

```SQL
SELECT movie.title, actor.name
FROM movie JOIN casting ON movie.id = casting.movieid
           JOIN actor ON actor.id = casting.actorid
WHERE casting.ord = 1 
and movie.id in (
         SELECT movie.id 
         FROM movie JOIN casting ON movie.id = casting.movieid
                    JOIN actor ON actor.id = casting.actorid
         WHERE actor.name = 'Julie Andrews')
```

13) Obtain a list, in alphabetical order, of actors who've had at least 15 starring roles.

```SQL
SELECT actor.name
FROM movie JOIN casting ON movie.id = casting.movieid
           JOIN actor ON actor.id = casting.actorid
WHERE casting.ord=1
GROUP BY actor.name
HAVING count(actor.name) >= 15
```

14) List the films released in the year 1978 ordered by the number of actors in the cast, then by title.

```SQL
SELECT movie.title, count(actor.name)
FROM movie JOIN casting ON movie.id = casting.movieid
           JOIN actor ON actor.id = casting.actorid
WHERE movie.yr = 1978
GROUP BY movie.title
ORDER BY COUNT(actor.name) DESC, movie.title ASC
```

15) List all the people who have worked with 'Art Garfunkel'.

```SQL
SELECT actor.name
FROM movie JOIN casting ON movie.id = casting.movieid
           JOIN actor ON actor.id = casting.actorid
WHERE actor.name != 'Art Garfunkel' AND movie.id IN (
                  SELECT movie.id 
                  FROM movie JOIN casting ON movie.id = casting.movieid
                             JOIN actor ON actor.id = casting.actorid
                  WHERE actor.name = 'Art Garfunkel')
```

## [Using Null](https://sqlzoo.net/wiki/Using_Null)

1) List the teachers who have NULL for their department.

```SQL
SELECT name 
FROM teacher 
WHERE dept is NULL
```

2) 

```SQL
SELECT teacher.name, dept.name
FROM teacher INNER JOIN dept ON (teacher.dept=dept.id)
```

3) Use a different JOIN so that all teachers are listed.

```SQL
SELECT teacher.name, dept.name
FROM teacher LEFT JOIN dept ON (teacher.dept=dept.id)
```

4) Use a different JOIN so that all departments are listed.

```SQL 
SELECT teacher.name, dept.name 
FROM teacher RIGHT JOIN dept ON (teacher.dept = dept.id)
```

5) Show teacher name and mobile number or '07986 444 2266'

```SQL 
SELECT name, coalesce(mobile, '07986 444 2266') -- coalesce function used to get first not null value:
FROM teacher                                    -- if mobile = 0 it will be replaced by '07986 444 2266'
```

6) Use the COALESCE function and a LEFT JOIN to print the teacher name and department name. Use the string 'None' where there is no department.

```SQL
SELECT teacher.name, coalesce(dept.name, 'None')           -- dept.name with null values replaced by 'None'
FROM teacher LEFT JOIN dept ON teacher.dept = dept.id
```

7) Use COUNT to show the number of teachers and the number of mobile phones.

```SQL 
SELECT COUNT(name), COUNT(mobile)
FROM teacher
```

8) Use COUNT and GROUP BY dept.name to show each department and the number of staff. Use a RIGHT JOIN to ensure that the Engineering department is listed.

```SQL 
SELECT dept.name, count(teacher.id)
FROM dept LEFT JOIN teacher ON dept.id = teacher.dept  -- you also can use RIGHT JOIN but you also have to put teacher first
GROUP BY dept.name
```

9) Use CASE to show the name of each teacher followed by 'Sci' if the teacher is in dept 1 or 2 and 'Art' otherwise.

```SQL
SELECT teacher.name,
       CASE WHEN teacher.dept in (1, 2) THEN 'Sci' ELSE 'Art' END
FROM teacher LEFT JOIN dept ON teacher.dept = dept.id
```

10) Use CASE to show the name of each teacher followed by 'Sci' if the teacher is in dept 1 or 2, show 'Art' if the teacher's dept is 3 and 'None' otherwise.
```SQL
SELECT teacher.name,
       CASE WHEN teacher.dept in (1, 2) THEN 'Sci' ELSE 'None' END
FROM teacher LEFT JOIN dept ON teacher.dept = dept.id
```

## [8+ NSS Tutorial](https://sqlzoo.net/wiki/NSS_Tutorial)

1) Show the the percentage who STRONGLY AGREE

```SQL
SELECT A_STRONGLY_AGREE
  FROM nss
 WHERE question='Q01'
   AND institution='Edinburgh Napier University'
   AND subject='(8) Computer Science'
```

2) Show the institution and subject where the score is at least 100 for question 15.

```SQL
SELECT institution, subject
  FROM nss
 WHERE question='Q15'
   AND score >= 100
```

3) Show the institution and score where the score for '(8) Computer Science' is less than 50 for question 'Q15'

```SQL
SELECT institution,score
  FROM nss
 WHERE question='Q15'
   AND score <= 50
   AND subject='(8) Computer Science'
```

4) Show the subject and total number of students who responded to question 22 for each of the subjects '(8) Computer Science' and '(H) Creative Arts and Design'.

```SQL
SELECT subject, SUM(response)
FROM nss
WHERE question='Q22'
      AND subject in ('(8) Computer Science', '(H) Creative Arts and Design')
GROUP BY subject
```

5) Show the subject and total number of students who A_STRONGLY_AGREE to question 22 for each of the subjects '(8) Computer Science' and '(H) Creative Arts and Design'.
```SQL
SELECT subject, SUM(A_STRONGLY_AGREE*response /100)
FROM nss
WHERE question='Q22'
      AND subject in ('(8) Computer Science', '(H) Creative Arts and Design')
GROUP BY subject
```

6) Show the percentage of students who A_STRONGLY_AGREE to question 22 for the subject '(8) Computer Science' show the same figure for the subject '(H) Creative Arts and Design'.

```SQL
SELECT subject, ROUND(AVG(A_STRONGLY_AGREE), 0)
FROM nss
WHERE question='Q22'
      AND subject in ('(8) Computer Science', '(H) Creative Arts and Design')
GROUP BY subject
```

7) Show the average scores for question 'Q22' for each institution that include 'Manchester' in the name.

```SQL
SELECT institution, ROUND(AVG(score),0)
  FROM nss
 WHERE question='Q22'
   AND (institution LIKE '%Manchester%')
GROUP BY institution
ORDER BY institution
```

8) Show the institution, the total sample size and the number of computing students for institutions in Manchester for 'Q01'.

```SQL
SELECT institution, SUM(sample),
       SUM(CASE WHEN subject ='(8) Computer Science' THEN sample ELSE 0 END)
FROM nss
WHERE institution LIKE '%Manchester%'
      AND question = 'Q01'
GROUP BY institution
```

## [Window functions](https://sqlzoo.net/wiki/Window_functions)

1) Show the lastName, party and votes for the constituency 'S14000024' in 2017.

```SQL
SELECT lastName, party, votes
  FROM ge
 WHERE constituency = 'S14000024' AND yr = 2017
ORDER BY votes DESC
```

2) Show the party and RANK for constituency S14000024 in 2017. List the output by party

```SQL
SELECT party, votes,
       RANK() OVER (ORDER BY votes DESC) as posn
FROM ge
WHERE constituency = 'S14000024' AND yr = 2017
ORDER BY party
```

3) Use PARTITION to show the ranking of each party in S14000021 in each year. Include yr, party, votes and ranking (the party with the most votes is 1).

```SQL
SELECT yr,party, votes,
      RANK() OVER (PARTITION BY yr ORDER BY votes DESC) as posn
FROM ge
WHERE constituency = 'S14000021'
ORDER BY party,yr
```
4) Use PARTITION BY constituency to show the ranking of each party in Edinburgh in 2017. Order your results so the winners are shown first, then ordered by constituency.

```SQL
SELECT constituency,party, votes, 
       RANK() OVER (PARTITION BY constituency ORDER BY votes DESC) as posn
FROM ge
WHERE constituency BETWEEN 'S14000021' AND 'S14000026'
      AND yr  = 2017
ORDER BY posn, constituency,votes DESC
```

5) Show the parties that won for each Edinburgh constituency in 2017

```SQL
SELECT constituency, party
FROM ge x 
WHERE constituency BETWEEN 'S14000021' AND 'S14000026'
      AND yr  = 2017
      AND votes >= ALL(SELECT votes FROM ge y
                       WHERE x.constituency = y.constituency 
                       AND yr = 2017)
```

6) Show how many seats for each party in Scotland in 2017.

```SQL
SELECT party,count(party) 
FROM (SELECT constituency,party ,votes,
      RANK() OVER (PARTITION BY constituency ORDER BY votes DESC) as posn
      FROM ge
      WHERE constituency LIKE 'S%' AND yr = 2017) tb
WHERE posn =1
GROUP BY party
ORDER BY party
```

## [9+ COVID](https://sqlzoo.net/wiki/Window_LAG) 

1) Modify the query to show data from Spain

```SQL 
SELECT name, DAY(whn),
 confirmed, deaths, recovered
 FROM covid
WHERE name = 'Spain'
AND MONTH(whn) = 3 AND YEAR(whn) = 2020
ORDER BY whn
```

2) Modify it to show the player, teamid, stadium and mdate for every German goal.

```SQL
SELECT name, DAY(whn), confirmed,
       LAG(confirmed, 1) OVER (PARTITION BY name ORDER BY whn)
FROM covid
WHERE name = 'Italy'
      AND MONTH(whn) = 3 
      AND YEAR(whn) = 2020
ORDER BY whn
```

3) Show the number of new cases for each day, for Italy, for March.

```SQL 
SELECT name, DAY(whn),
   (confirmed - LAG(confirmed, 1) OVER (PARTITION BY name ORDER BY whn))
FROM covid
WHERE name = 'Italy'
      AND MONTH(whn) = 3 AND YEAR(whn) = 2020
ORDER BY whn
```

4) Show the number of new cases in Italy for each week in 2020 - show Monday only.

```SQL 
DATE_ FROMAT unsuported 
```

8) For each country that has had at last 1000 new cases in a single day, show the date of the peak number of new cases.

```SQL
 SELECT name,  max(confirmed) as conf
FROM (SELECT name, whn, 
      (confirmed - LAG(confirmed, 1) OVER (PARTITION BY name ORDER BY whn)) as confirmed
      FROM covid
      WHERE MONTH(whn) = 3 AND YEAR(whn) = 2020) tb
GROUP BY name
having max(confirmed) >= 1000
```

## [9 Self join](https://sqlzoo.net/wiki/Self_join)

1) How many stops are in the database.

```SQL
SELECT count(id)
FROM stops
```

2) Find the id value for the stop 'Craiglockhart'

```SQL 
SELECT id 
FROM stops 
WHERE name = 'Craiglockhart'
```

3) Give the id and the name for the stops on the '4' 'LRT' service.

```SQL
SELECT id, name 
FROM stops JOIN route ON stops.id = route.stop
WHERE company = 'LRT' AND num = '4'
```

4) Routes and stops

```SQL
SELECT company, num, COUNT(*)
FROM route WHERE stop=149 OR stop=53
GROUP BY company, num
HAVING COUNT(*)=2
```


5) Execute the self join shown and observe that b.stop gives all the places you can get to from Craiglockhart, without changing routes. Change the query so that it shows the services from Craiglockhart to London Road.

```SQL
SELECT a.company, a.num, a.stop, b.stop
FROM route a JOIN route b ON
  (a.company=b.company AND a.num=b.num)
WHERE a.stop=53 and b.stop = 149
```

6) The query shown is similar to the previous one, however by joining two copies of the stops table we can refer to stops by name rather than by number. Change the query so that the services between 'Craiglockhart' and 'London Road' are shown. If you are tired of these places try 'Fairmilehead' against 'Tollcross'

```SQL
SELECT a.company, a.num, stopa.name, stopb.name
FROM route a JOIN route b ON
  (a.company=b.company AND a.num=b.num)
  JOIN stops stopa ON (a.stop=stopa.id)
  JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopa.name='Craiglockhart' and stopb.name = 'London Road'
```

7)Give a list of all the services which connect stops 115 and 137 ('Haymarket' and 'Leith')

```SQL
SELECT distinct a.company, a.num
FROM route a JOIN route b ON
  (a.company=b.company AND a.num=b.num)
WHERE a.stop = 115 and b.stop = 137
```

8) Give a list of the services which connect the stops 'Craiglockhart' and 'Tollcross'

```SQL
SELECT a.company, a.num
FROM route a JOIN route b ON
  (a.company=b.company AND a.num=b.num)
  JOIN stops stopa ON (a.stop=stopa.id)
  JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopa.name='Craiglockhart' and stopb.name = 'Tollcross'
```

9) Give a distinct list of the stops which may be reached from 'Craiglockhart' by taking one bus, including 'Craiglockhart' itself, offered by the LRT company. Include the company and bus no. of the relevant services.

```SQL
SELECT stopa.name, a.company, a.num
FROM route a JOIN route b ON
  (a.company=b.company AND a.num=b.num)
  JOIN stops stopa ON (a.stop=stopa.id)
  JOIN stops stopb ON (b.stop=stopb.id)
WHERE a.company = 'LRT' AND stopb.name = 'Craiglockhart'
````

10) Find the routes involving two buses that can go from Craiglockhart to Lochend.
Show the bus no. and company for the first bus, the name of the stop for the transfer,
and the bus no. and company for the second bus.

```SQL
SELECT bus1.numb, bus1.compn, bus1.stop2, bus2.numb, bus2.compn
FROM 
(SELECT a.num as numb, a.company as compn, stopa.name as stop1, stopb.name as stop2
FROM route a JOIN route b ON
  (a.company=b.company AND a.num=b.num)
  JOIN stops stopa ON (a.stop=stopa.id)
  JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopa.name = 'Craiglockhart') as bus1
JOIN 
(SELECT a.num as numb, a.company as compn, stopa.name as stop2, stopb.name as stop3
FROM route a JOIN route b ON
  (a.company=b.company AND a.num=b.num)
  JOIN stops stopa ON (a.stop=stopa.id)
  JOIN stops stopb ON (b.stop=stopb.id)
WHERE stopb.name = 'Lochend') as bus2
ON bus1.stop2 = bus2.stop2
```

## [11 Tutorial Student Records](https://sqlzoo.net/wiki/DDL_Student_Records)

![](https://hackmd.io/_uploads/rklKWQdOh.png)

My script:
```SQL
--CREATE student
create table test.student (
	matric_no char(8) not null,
	first_name varchar(50),
	last_name varchar(50),
	date_of_birth date
);

--Add some students to the database
insert into test.student values('40001010','Daniel', 'Radcliffe', '1989-07-23')
insert into test.student values('40001011', 'Emma', 'Watson', '1990-04-15');
insert into test.student values('40001012', 'Rupert', 'Grint', '1988-10-24');

-- CREATE module
create table test.module(
	module_code char(8),
	module_title varchar(50),
	level int,
	credits int not null 
);

-- Add some modules
insert into 
	test.module
values 
	('HUF07101', 'Herbology', 7),
	('SLY07102', 'Defense Against the Dark Arts', 7),
	('HUF08102', 'History of Magic', 8);

--CREATE registration
create table test.registration (
	matric_no char(8),
	matric_code char(8),
	result decimal(4,1)
);


--Adjust keys
ALTER TABLE test.module ADD CONSTRAINT module_pk PRIMARY KEY (module_code);
ALTER TABLE test.student ADD CONSTRAINT student_pk PRIMARY KEY (matric_no);
ALTER TABLE test.registration  ADD CONSTRAINT registration_pk PRIMARY KEY (matric_no);
ALTER TABLE test.registration  ADD CONSTRAINT registration_pk PRIMARY KEY (matric_code);
-- Foreign keys
ALTER TABLE test.registration ADD CONSTRAINT registration_fk FOREIGN KEY (matric_code) REFERENCES test.module(module_code);
ALTER TABLE test.registration ADD CONSTRAINT registration_fks FOREIGN KEY (matric_no) REFERENCES test.student(matric_no);

--Add some data

insert into 
	test.registration 
values
	('40001010', 'SLY07102', 90),
	('40001010','HUF07101',40),
	('40001010','HUF08102',null),
	('40001011','SLY07102',99);
```

## [2015 UK General Election using](https://sqlzoo.net/wiki/2015_UK_General_Election_using_mysql)

```SQL
--Creating table ge
create table public.ge(
	ons_id VARCHAR(10),
	ons_region_id VARCHAR(10),
	constituency_name VARCHAR(50),
	county_name VARCHAR(50),
	region_name VARCHAR(50),
	country_name VARCHAR(50),
	constituency_type VARCHAR(10),
	party_name VARCHAR(50),
	party_abbreviation VARCHAR(50),
	firstname VARCHAR(50),
	surname VARCHAR(50),
	gender VARCHAR(6),
	sitting_mp VARCHAR(3),
	former_mp VARCHAR(3),
	votes INT,
	share FLOAT,
	change VARCHAR(20),
	PRIMARY KEY(ons_id,firstname,surname)
);
```
To solve this problem i used postgresql database

To load data to database i used Pentaho Data integration tool in this repository i added file Load-csv-postgres.ktr
table.csv is data added 

To launch ktr file in PDI adjust conncetion
```
<connection>
    <server>******</server> - put server host
    <type>POSTGRESQL</type>
    <access>Native</access>
    <database>*****</database> - database name
    <port>****</port>  - put port 
    <username>****</username> - put username
    <password>*******</password> - put password
```


