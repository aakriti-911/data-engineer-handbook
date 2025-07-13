SELECT * FROM public.player_seasons;

CREATE TYPE season_stats AS (
							season INTEGER,
							gp REAL,
							pts REAL,
							reb REAL,
							ast REAL
						)

CREATE TABLE players (
	player_name TEXT,
	height TEXT,
	college TEXT,
	country TEXT,
	draft_year TEXT,
	draft_round TEXT,
	draft_number TEXT,
	season_stats season_stats[],
	current_season INT,
	PRIMARY KEY(player_name, current_season)
)


SELECT MIN(season) FROM public.player_seasons;  -- 1996

-- use magic notebook example - each day we just keep adding a new page to the previous record [yesterday and today theory]

WITH yesterday AS (
	SELECT * FROM player_seasons
	WHERE season = 1995   --                       yesterday has 0 records for 1995 => no records => null values
),
	today AS (
		SELECT * FROM player_seasons
		WHERE season = 1996   --                   today has records for 1996; will be considered
	)
SELECT * FROM today t
FULL OUTER JOIN yesterday y 
	on t.player_name = y.player_name   --          shows null for y.cols and records as is for t.cols


-- SEED query for cumulation : because players for yesterday is null, so it still returns the records from today CTE
SELECT 
	COALESCE(t.player_name, y.player_name) AS player_name,
	COALESCE(t.height, y.height) AS height,
	COALESCE(t.college, y.college) AS college,
	COALESCE(t.country, y.country) AS country,
	COALESCE(t.draft_year, y.draft_year) AS draft_year,
	COALESCE(t.draft_round, y.draft_round) AS draft_round,
	COALESCE(t.draft_number, y.draft_number) AS draft_number,
FULL OUTER JOIN yesterday y 
	on t.player_name = y.player_name 


-- Now, we need to add in seasons array
SELECT 
	COALESCE(t.player_name, y.player_name) AS player_name,
	COALESCE(t.height, y.height) AS height,
	COALESCE(t.college, y.college) AS college,
	COALESCE(t.country, y.country) AS country,
	COALESCE(t.draft_year, y.draft_year) AS draft_year,
	COALESCE(t.draft_round, y.draft_round) AS draft_round,
	COALESCE(t.draft_number, y.draft_number) AS draft_number,
    COALESCE(t.season, y.season + 1) AS current_season,
    	CASE 
    	WHEN y.player_name IS NULL THEN 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
    	ELSE 
        	ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats] || 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
	END AS season_stats
	-- :: will cast it to the type that we are looking for (STRUCT); if y.season_stats is null => simply put the incoming today's data
	-- || will concat the yesterday's y.season_stats with todays incoming records if y.season_stats is not null
FROM today t
FULL OUTER JOIN yesterday y 
	on t.player_name = y.player_name 


-- There is 1 more CASE here becasue we do not want to add to the array if today's value is null for a player that has retired, becasue we would want to hold on to that players data but we don't want to keep on adding more nulls to the array
	CASE 
    	WHEN y.player_name IS NULL THEN 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
    	WHEN t.season IS NOT NULL THEN  
        	ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats] || 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
		ELSE ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats]
	END AS season_stats


-- Now we use the same selectq query as above and insert into our table that we created : players
INSERT INTO players
SELECT 
	COALESCE(t.player_name, y.player_name) AS player_name,
	COALESCE(t.height, y.height) AS height,
	COALESCE(t.college, y.college) AS college,
	COALESCE(t.country, y.country) AS country,
	COALESCE(t.draft_year, y.draft_year) AS draft_year,
	COALESCE(t.draft_round, y.draft_round) AS draft_round,
	COALESCE(t.draft_number, y.draft_number) AS draft_number,
    COALESCE(t.season, y.season + 1) AS current_season,
    CASE 
    	WHEN y.player_name IS NULL THEN 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
    	WHEN t.season IS NOT NULL THEN  
        	ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats] || 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
		ELSE ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats]
	END AS season_stats
	-- :: will cast it to the type that we are looking for (STRUCT); if y.season_stats is null => simply put the incoming today's data
	-- || will concat the yesterday's y.season_stats with todays incoming records if y.season_stats is not null
FROM today t
FULL OUTER JOIN yesterday y 
	on t.player_name = y.player_name 


-------- now we do an incremental load : year 1996 as yesterday , year 1997 as today :
WITH yesterday AS (
	SELECT * FROM player_seasons
	WHERE season = 1996
),
today AS (
	SELECT * FROM player_seasons
	WHERE season = 1997
)

INSERT INTO players 
SELECT 
	COALESCE(t.player_name, y.player_name) AS player_name,
	COALESCE(t.height, y.height) AS height,
	COALESCE(t.college, y.college) AS college,
	COALESCE(t.country, y.country) AS country,
	COALESCE(t.draft_year, y.draft_year) AS draft_year,
	COALESCE(t.draft_round, y.draft_round) AS draft_round,
	COALESCE(t.draft_number, y.draft_number) AS draft_number,
	CASE 
    	WHEN y.player_name IS NULL THEN 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
    	WHEN t.season IS NOT NULL THEN  
        	ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats] || 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
		ELSE ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats]
	END AS season_stats,
	COALESCE(t.season, y.season + 1) AS current_season
FROM today t
FULL OUTER JOIN yesterday y 
	ON t.player_name = y.player_name;

SELECT * FROM players;

--- if we join this table now with some other table, it can give us some more info --> players can easily become player_seasons again [explode the column]
WITH unnested AS(
SELECT player_name,
		UNNEST(season_stats)::season_stats AS season_stats
FROM players
WHERE current_season = 2001
AND player_name = 'Michael Jordan')

SELECT player_name,
	(season_stats::season_stats).*
FROM unnested








-- what if we want to look at some other stuff 
DROP TABLE players;

-- CREATE again by adding 2 more columns 
-- 1. scoring class column [essentially to see if it's a good player/ a bad player] -- based all on the points
-- 2. years_cince_last_season

CREATE TYPE scoring_class AS ENUM ('star', 'good', 'avg', 'bad');

CREATE TABLE players (
    player_name TEXT,
    height TEXT,
    college TEXT,
    country TEXT,
    draft_year TEXT,
    draft_round TEXT,
    draft_number TEXT,
    season_stats season_stats[],
    scoring_class scoring_class,
    years_since_last_season INTEGER,
    current_season INT,
    PRIMARY KEY(player_name, current_season)
);

WITH yesterday AS (
	SELECT * FROM player_seasons
	WHERE season = 1996
),
today AS (
	SELECT * FROM player_seasons
	WHERE season = 1997
)

INSERT INTO players 
SELECT 
	COALESCE(t.player_name, y.player_name) AS player_name,
	COALESCE(t.height, y.height) AS height,
	COALESCE(t.college, y.college) AS college,
	COALESCE(t.country, y.country) AS country,
	COALESCE(t.draft_year, y.draft_year) AS draft_year,
	COALESCE(t.draft_round, y.draft_round) AS draft_round,
	COALESCE(t.draft_number, y.draft_number) AS draft_number,
	CASE 
    	WHEN y.player_name IS NULL THEN 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
    	WHEN t.season IS NOT NULL THEN  
        	ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats] || 
        	ARRAY[ROW(t.season, t.gp, t.pts, t.reb, t.ast)::season_stats]
		ELSE ARRAY[ROW(y.season, y.gp, y.pts, y.reb, y.ast)::season_stats]
	END AS season_stats,

	CASE 
		WHEN t.season IS NOT NULL THEN 
			CASE 
				WHEN t.pts > 20 THEN 'star'
				WHEN t.pts > 15 THEN 'good'
				WHEN t.pts > 10 THEN 'avg'
				WHEN t.pts > 5 THEN 'bad'
				ELSE 'bad'
			END::scoring_class
		ELSE y.scoring_class
	END,
	CASE WHEN t.season IS NOT NULL THEN 0
		ELSE y.years_since_last_season + 1
	END as years_since_last_season,
	COALESCE(t.season, y.season + 1) AS current_season
FROM today t
FULL OUTER JOIN yesterday y 
	on t.player_name = y.player_name 



--- LET'S DO ANALYSIS ON WHICH PLAYER DID THE MOST IMPROVEMENT FROM THEIR FIRST SEASON TO THEIR MOST REASON SEASON

SELECT
	player_name,
	(season_stats[CARDINALITY(season_stats)]::season_stats).pts/
	CASE WHEN (season_stats[1]::season_stats).pts = 0 THEN 1 ELSE season_stats[1]::season_stat END
FROM players
WHERE current_season = 2021
ORDER BY 2 DESC

SELECT
	player_name,
	(season_stats[CARDINALITY(season_stats)]::season_stats).pts/
	CASE WHEN (season_stats[1]::season_stats).pts = 0 THEN 1 ELSE season_stats[1]::season_stat END
FROM players
WHERE current_season = 2021
AND scoring_class = 'star'

-- You can do easy analysis on cumulative tables and you don't have to do any shuffle, any group bys
