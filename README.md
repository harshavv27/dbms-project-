# dbms-project

we have created a project in dbms , which is cricket tournament database managment system 

the code is mentioned below 

-- 1. CREATE TABLES (SQLite compatible)
DROP TABLE IF EXISTS ball;
DROP TABLE IF EXISTS over;
DROP TABLE IF EXISTS inning;
DROP TABLE IF EXISTS match;
DROP TABLE IF EXISTS player_team_season;
DROP TABLE IF EXISTS player;
DROP TABLE IF EXISTS team;
DROP TABLE IF EXISTS venue;
DROP TABLE IF EXISTS season;

CREATE TABLE team (
  team_id INTEGER PRIMARY KEY AUTOINCREMENT,
  name VARCHAR(100) UNIQUE,
  city VARCHAR(100)
);

CREATE TABLE player (
  player_id INTEGER PRIMARY KEY AUTOINCREMENT,
  full_name VARCHAR(120),
  role VARCHAR(50),
  batting_style VARCHAR(50),
  bowling_style VARCHAR(50)
);

CREATE TABLE season (
  season_id INTEGER PRIMARY KEY AUTOINCREMENT,
  year INT UNIQUE
);

CREATE TABLE player_team_season (
  pts_id INTEGER PRIMARY KEY AUTOINCREMENT,
  player_id INT,
  team_id INT,
  season_id INT,
  FOREIGN KEY (player_id) REFERENCES player(player_id),
  FOREIGN KEY (team_id) REFERENCES team(team_id),
  FOREIGN KEY (season_id) REFERENCES season(season_id),
  UNIQUE(player_id, season_id)
);

CREATE TABLE venue (
  venue_id INTEGER PRIMARY KEY AUTOINCREMENT,
  name VARCHAR(100),
  city VARCHAR(100)
);

CREATE TABLE match (
  match_id INTEGER PRIMARY KEY AUTOINCREMENT,
  season_id INT,
  match_no INT,
  match_date DATETIME,
  venue_id INT,
  team1_id INT,
  team2_id INT,
  toss_winner_team_id INT,
  toss_decision VARCHAR(10),
  winner_team_id INT,
  completed BOOLEAN DEFAULT 0,
  FOREIGN KEY (season_id) REFERENCES season(season_id),
  FOREIGN KEY (venue_id) REFERENCES venue(venue_id),
  FOREIGN KEY (team1_id) REFERENCES team(team_id),
  FOREIGN KEY (team2_id) REFERENCES team(team_id),
  FOREIGN KEY (toss_winner_team_id) REFERENCES team(team_id),
  FOREIGN KEY (winner_team_id) REFERENCES team(team_id)
);

CREATE TABLE inning (
  inning_id INTEGER PRIMARY KEY AUTOINCREMENT,
  match_id INT,
  inning_no INT,
  batting_team_id INT,
  bowling_team_id INT,
  total_runs INT DEFAULT 0,
  total_wickets INT DEFAULT 0,
  overs_played DECIMAL(4,1) DEFAULT 0,
  FOREIGN KEY (match_id) REFERENCES match(match_id),
  FOREIGN KEY (batting_team_id) REFERENCES team(team_id),
  FOREIGN KEY (bowling_team_id) REFERENCES team(team_id),
  UNIQUE(match_id, inning_no)
);

CREATE TABLE over (
  over_id INTEGER PRIMARY KEY AUTOINCREMENT,
  inning_id INT,
  over_no INT,
  bowler_player_id INT,
  FOREIGN KEY (inning_id) REFERENCES inning(inning_id),
  FOREIGN KEY (bowler_player_id) REFERENCES player(player_id)
);

CREATE TABLE ball (
  ball_id INTEGER PRIMARY KEY AUTOINCREMENT,
  over_id INT,
  ball_no INT,
  batsman_id INT,
  bowler_id INT,
  runs INT DEFAULT 0,
  extras INT DEFAULT 0,
  wicket_player_out INT,
  FOREIGN KEY (over_id) REFERENCES over(over_id),
  FOREIGN KEY (batsman_id) REFERENCES player(player_id),
  FOREIGN KEY (bowler_id) REFERENCES player(player_id),
  FOREIGN KEY (wicket_player_out) REFERENCES player(player_id)
);

-- 2. INSERT METADATA (Season & Venue)
INSERT INTO season(year) VALUES (2025);
INSERT INTO venue(name, city) VALUES ('Wankhede Stadium', 'Mumbai');

-- 3. INSERT 4 TEAMS
INSERT INTO team(name, city) VALUES 
 ('Mumbai Indians', 'Mumbai'),       -- ID: 1
 ('Chennai Super Kings', 'Chennai'), -- ID: 2
 ('Royal Challengers', 'Bangalore'), -- ID: 3
 ('Kolkata Riders', 'Kolkata');      -- ID: 4

-- 4. INSERT PLAYERS (3 Key Players per Team for Demo)
INSERT INTO player(full_name, role, batting_style, bowling_style) VALUES
 -- MI (Team 1)
 ('Rohit Sharma', 'Batsman', 'RHB', 'Offbreak'),
 ('Jasprit Bumrah', 'Bowler', 'RHB', 'Right-arm Fast'),
 ('Hardik Pandya', 'Allrounder', 'RHB', 'Right-arm Fast'),
 -- CSK (Team 2)
 ('MS Dhoni', 'Wicketkeeper', 'RHB', 'Medium'),
 ('Ravindra Jadeja', 'Allrounder', 'LHB', 'Left-arm Spin'),
 ('Ruturaj Gaikwad', 'Batsman', 'RHB', 'Offbreak'),
 -- RCB (Team 3)
 ('Virat Kohli', 'Batsman', 'RHB', 'Medium'),
 ('Glenn Maxwell', 'Allrounder', 'RHB', 'Offbreak'),
 ('Mohd Siraj', 'Bowler', 'RHB', 'Right-arm Fast'),
 -- KKR (Team 4)
 ('Shreyas Iyer', 'Batsman', 'RHB', 'Legbreak'),
 ('Andre Russell', 'Allrounder', 'RHB', 'Right-arm Fast'),
 ('Sunil Narine', 'Bowler', 'LHB', 'Offbreak');

-- 5. ASSIGN PLAYERS TO TEAMS (Season 1)
INSERT INTO player_team_season(player_id, team_id, season_id) VALUES
 -- MI
 (1, 1, 1), (2, 1, 1), (3, 1, 1),
 -- CSK
 (4, 2, 1), (5, 2, 1), (6, 2, 1),
 -- RCB
 (7, 3, 1), (8, 3, 1), (9, 3, 1),
 -- KKR
 (10, 4, 1), (11, 4, 1), (12, 4, 1);

-- SIMULATE A COMPLETE MATCH (MI vs CSK)
-- Match: 2 Overs per side

-- 6. CREATE MATCH
INSERT INTO match(season_id, match_no, match_date, venue_id, team1_id, team2_id, toss_winner_team_id, toss_decision)
VALUES (1, 1, CURRENT_TIMESTAMP, 1, 1, 2, 1, 'Bat');

-- 7. CREATE INNINGS
INSERT INTO inning(match_id, inning_no, batting_team_id, bowling_team_id) VALUES
 (1, 1, 1, 2), -- MI Batting
 (1, 2, 2, 1); -- CSK Batting

-- ------------------------------------------
-- INNING 1: MI Batting (Target Setting)
-- ------------------------------------------

-- Over 1: Jadeja bowling to Rohit
INSERT INTO over(inning_id, over_no, bowler_player_id) VALUES (1, 1, 5); -- Jadeja

INSERT INTO ball(over_id, ball_no, batsman_id, bowler_id, runs) VALUES
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=1), 1, 1, 5, 4), -- Rohit hits 4
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=1), 2, 1, 5, 0), -- Dot
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=1), 3, 1, 5, 1), -- Single
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=1), 4, 3, 5, 6), -- Hardik hits 6
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=1), 5, 3, 5, 1), -- Single
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=1), 6, 1, 5, 4); -- Rohit hits 4
 -- Total: 16 runs

-- Over 2: Gaikwad (Part-time) bowling to Hardik
INSERT INTO over(inning_id, over_no, bowler_player_id) VALUES (1, 2, 6); -- Gaikwad

INSERT INTO ball(over_id, ball_no, batsman_id, bowler_id, runs) VALUES
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=2), 1, 3, 6, 6), -- Hardik hits 6
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=2), 2, 3, 6, 0), -- Wicket!
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=2), 3, 1, 6, 1), -- Rohit Single
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=2), 4, 2, 6, 0), -- Bumrah Dot
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=2), 5, 2, 6, 0), -- Bumrah Dot
 ((SELECT over_id FROM over WHERE inning_id=1 AND over_no=2), 6, 2, 6, 4); -- Bumrah hits 4
 -- Total Inning 1 Score: 16 + 11 = 27 Runs

-- ------------------------------------------
-- INNING 2: CSK Batting (Chasing 28 to win)
-- ------------------------------------------

-- Over 1: Bumrah bowling to Dhoni
INSERT INTO over(inning_id, over_no, bowler_player_id) VALUES (2, 1, 2); -- Bumrah

INSERT INTO ball(over_id, ball_no, batsman_id, bowler_id, runs) VALUES
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=1), 1, 4, 2, 0), -- Dhoni Dot
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=1), 2, 4, 2, 1), -- Single
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=1), 3, 6, 2, 0), -- Gaikwad Dot
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=1), 4, 6, 2, 4), -- Gaikwad 4
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=1), 5, 6, 2, 1), -- Single
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=1), 6, 4, 2, 6); -- Dhoni 6
 -- Total: 12 runs

-- Over 2: Hardik bowling to Dhoni
INSERT INTO over(inning_id, over_no, bowler_player_id) VALUES (2, 2, 3); -- Hardik

INSERT INTO ball(over_id, ball_no, batsman_id, bowler_id, runs) VALUES
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=2), 1, 4, 3, 6), -- Dhoni 6
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=2), 2, 4, 3, 6), -- Dhoni 6
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=2), 3, 4, 3, 4), -- Dhoni 4 (WINNING SHOT!)
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=2), 4, 4, 3, 0), -- Dead rubber
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=2), 5, 4, 3, 0), -- Dead rubber
 ((SELECT over_id FROM over WHERE inning_id=2 AND over_no=2), 6, 4, 3, 0); -- Dead rubber
 -- Total Inning 2 Score: 12 + 16 = 28 Runs (CSK Wins)

-- 8. FINALIZE MATCH manually since stored procedures aren't supported in SQLite
UPDATE match SET completed = 1, winner_team_id = 2 WHERE match_id = 1;

-- 9. SHOW RESULTS (Leaderboard & Stats manually instead of views)
SELECT p.full_name AS Player, SUM(b.runs) AS Runs
FROM player p
JOIN ball b ON p.player_id = b.batsman_id
GROUP BY p.full_name
ORDER BY Runs DESC;

SELECT p.full_name AS Player, COUNT(b.ball_id) AS Balls_Faced
FROM player p
JOIN ball b ON p.player_id = b.batsman_id
GROUP BY p.full_name
ORDER BY Balls_Faced DESC;
