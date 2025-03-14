# Overview

This project will use the [2024 League of Legends eSports Match Data](https://drive.google.com/drive/u/1/folders/1gLSw0RLjBbtaNy0dgnGQDAZOHIgCe-HH) to analyze bans and picks in League of Legends. I aim to analyze the effectiveness and most optimal combos of champion bans and picks, and determine the extent of their effect on match behavior and outcome. This project is made for DSC 80 at UCSD.

# Introduction

League of Legends is an extremely popular multiplayer online battle arena game developed by Riot Games. The game is especially known for its eSports scene, which has to grown to be extensively competitive and rigorous. The dataset used in this project comes from *Oracle's Elixir*, and contains detailed information about League of Legends matches played throughout 2024. 

At the most basic level, League of Legends is a game about **strategy** and **teamwork**. With 170 champions to choose from, there are thousands of champion combinations and synergies. This makes a team's composition of champions a crucial factor in determining the outcome of a game—the better the synergy of a team, the more likely it is that they will win a game. However, at more competitive levels of League of Legends, there is also a mechanic known as the **ban phase**, where players can ban up to 5 champions for the other team, meaning that the opposing team will not be able to choose those champions. This mechanic can be vital for limiting the opposing team's synergy and ability to work well together. However, the ban phase is only as beneficial as the players make it, meaning that teams must choose their bans strategically. Collectively, the process of picking and banning champions is known as the **pick/ban phase**, and the results of this phase are what will be explored in detail in this project.

The core question for this project is: **"How do a team's picks and bans affect their likelihood of winning a game?"** First, we want gain a broad understanding of the shape of the dataset. Then, we will move onto the main goal of this project, which will be building a classification model to predict whether or not a team won a game, given the team's picks and bans, as well as data from the early stages of a match.

The dataset I will be working with is enormous, as it contains 117,576 rows and 161 columns. I will only be using the columns most relevant to the goal of this project.

| Column | Description |
|---|---|
| `'gameid'` | Unique identifier for a match. |
| `'side'` | Color of a team. |
| `'result'` | Boolean indicating whether or not a team won a match. |
| `'kills'` | The amount of player kills a player or team has. |
| `'deaths'` | The amount of deaths a player of team has. |
| `'assists` | The amount of assists (kills a player helped achieve, but did not get directly) a player or team has. |
| `'gamelength'` | Length of a match in seconds. |
| `'team kpm'` | A team's amount of kills per minute. |
| `'ckpm'` | The combined amount of kills per minute of both teams. |
| `'damagetakenperminute'` | The amount of damage taken per minute by a player or team. |
| `'damagemitigatedperminute'` | The amount of damage mitigated (damage taken that is reduced) per minute by a player or team. |
| `'champion'` | A player's chosen champion. |
| `'ban1'`, `'ban2'`, `'ban3'`, `'ban4'`, `'ban5'` | A banned champion which a team's opponents cannot choose. |
| `'monsterkills'` | The amount of monsters a player or team has killed. |
| `'minionkills'` | The amount of minions a player has killed. |
| `'dragons'` | The amount of dragons a player or team has slain. |
| `'barons'` | The amount of barons a player or team has slain. |
| `goldat10` | A player or team's amount of gold 10 minutes into a game. |
| `xpat10` | A player or team's amount of xp 10 minutes into a game. |
| `'csat10'` | A player or team's creep score (number of minions, neutral monsters, and wards that a player has last-hit over the length of a game) 10 minutes into a game. |
| `'opp_goldat10'` | The opponent's amount of gold 10 minutees into a game. |
| `'opp_xpat10'` | The opponent's amount of xp 10 minutes into a game. |
| `'opp_csat10'` | The opponent's creep score 10 minutes into a game. |
| `'golddiffat10'` | The difference between both teams' amount of gold 10 minutes into a game. |
| `'xpdiffat10'` | The difference between both teams' amount of xp 10 minutes into a game. |
| `'csdiffat10'` | The difference between both teams' creep score 10 minutes into a game. |
| `'killsat10'` | A player or team's amount of kills 10 minutes into a game. |
| `'assistsat10'` | A player or team's amount of assists 10 minutes into a game. |
| `'deathsat10'` | A player or team's amount of deaths 10 minutes into a game. |
| `'opp_killsat10'` | The opponent's amount of kills 10 minutes into a game. |
| `'opp_assistsat10'` | The opponent's amount of assists 10 minutes into a game. |
| `'opp_deathsat10'` | The opponent's amount of deaths 10 minutes into a game. |

# Data Cleaning and Exploratory Data Analysis

## Cleaning 

The format of the dataset I will be working with is in a unique format, where every 12 rows represents a match: the first 10 rows are player data, and the remaining 2 rows are data for the teams. I will keep the rows for the teams, as it is more comprehensive and further compresses the size of the dataset. I will be using **all** of the columns described above in my cleaned dataset. The steps I took to clean the dataset are as follows:
- The team rows have missing values for `champion` and `minionkills`. I filled in this missing data with lists of a team's five chosen champions for `champion`, and the sum of each player's amount of minions respective to their team for `minionkills`.
- I dropped the rows corresponding to player data, only keeping rows that correspond to team data.
- There are five separate columns representing banned champions, which are labeled `ban1` - `ban5`. Missing values in any of these ban columns are filled in with "N/A". I condensed these columns into a single column called `bans`, which contains lists of a team's five chosen bans for their opposing team.
- I converted the `result` column from an integer type to a boolean type, converted `gamelength` to minutes, and made sure that `minionkills`, `dragonkills`, and `baronkills` were integer columns.
- Finally, I renamed and organized the columns for better visual appearance.

The cleaned DataFrame contains 32 columns total: `gameid`, `team`, `victory`, `picks`, `bans`, `gamelength`, `kills`, `deaths`, `assists`, `team kpm`, `ckpm`, `damage taken/min`, `damage mitigated/min`, `monsterkills`, `minionkills`, `dragonkills`, `baronkills`, `goldat10`, `xpat10`, `csat10`, `opp_goldat10`, `opp_xpat10`, `opp_csat10`, `golddiffat10`, `xpdiffat10`, `csdiffat10`, `killsat10`, `assistsat10`, `deathsat10`, `opp_killsat10`, `opp_assistsat10`, and `opp_deathsat10`. Here are the first few rows of the cleaned DataFrame. For the purposes of display, only the first 10 columns are displayed.

| gameid             | team   | victory   | picks                                                  | bans                                                        |   gamelength |   kills |   deaths |   assists |   team kpm |
|:-------------------|:-------|:----------|:-------------------------------------------------------|:------------------------------------------------------------|-------------:|--------:|---------:|----------:|-----------:|
| 10660-10660_game_1 | Blue   | False     | ['Aatrox', 'Maokai', 'Orianna', 'Kalista', 'Senna']    | ['Akali', 'Nocturne', "K'Sante", 'Lee Sin', 'Wukong']       |      31.4333 |       3 |       16 |         7 |     0.0954 |
| 10660-10660_game_1 | Red    | True      | ['Rumble', 'Rell', 'LeBlanc', 'Varus', 'Renata Glasc'] | ['Poppy', 'Ashe', 'Neeko', 'Vi', 'Jarvan IV']               |      31.4333 |      16 |        3 |        43 |     0.509  |
| 10660-10660_game_2 | Blue   | False     | ['Kennen', "Bel'Veth", 'Neeko', 'Senna', 'Tahm Kench'] | ['Nocturne', 'Udyr', 'Renata Glasc', 'Nautilus', 'Lee Sin'] |      31.85   |       3 |       17 |         8 |     0.0942 |
| 10660-10660_game_2 | Red    | True      | ['Jax', 'Jarvan IV', 'LeBlanc', 'Kalista', 'Rell']     | ['Poppy', 'Ashe', 'Rumble', 'Tristana', 'Lucian']           |      31.85   |      17 |        3 |        42 |     0.5338 |
| 10660-10660_game_3 | Blue   | True      | ['Jax', "Bel'Veth", 'Neeko', 'Caitlyn', 'Lux']         | ['Rell', 'Nocturne', 'Tristana', 'Jarvan IV', 'Rumble']     |      22.0667 |      21 |        3 |        32 |     0.9517 |
