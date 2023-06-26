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













