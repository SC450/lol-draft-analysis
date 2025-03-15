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

## Cleaning the Dataset

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

## Univariate Analysis

Let's take a look at the distribution of the `ckpm` column in our cleaned dataset, which is the combined team kills per minute of a game. This statistic is useful for quantifying how bloody or action-packed a game was. The plot exhibits a normal distribution, with a slight skew to the right, suggesting that high intensity games can be quite rare.

<iframe
  src="assets/ckpm_plot.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

## Bivariate Analysis

Now, we'll look at the relationship between `damage taken/min` and `ckpm` by plotting them against each other. The scatter plot shows a positive correlation with a generally linear relationship, meaning that the death toll of a game will generally increase as players take more damage per minute.

<iframe
  src="assets/dmg_taken_vs_ckpm_plot.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

## Interesting Aggregates

To conclude our exploration into the factors that make a game action-packed, let's look at a pivot table between `dragonkills` and `baronkills`. The index of this pivot table are `dragonkills` and the columns are `baronkills`, with values in the table being a team's average amount of damage taken per minute (`damage taken/min`) for that combination of `dragonkills` and `baronkills`. We can see that as the amount of dragons and barons killed in a game increases, there is generally more damage taken per minute by teams. However, this is not always the case: for example, the average damage taken per minute for games with 5 `dragonkills` and 4 `baronkills` is 4,023.9, while the average damage taken per minute for games with 7 `dragonkills` and 5 `baronkills` is 3,589.74, which means that higher amounts of dragons and barons killed does not necessarily result in a team taking more damage. This hints at the fact that slaying dragons and barons does not have to be a significantly perilous endeavor, if teams play carefully enough.

|   dragonkills |       0 |       1 |       2 |       3 |       4 |       5 |
|--------------:|--------:|--------:|--------:|--------:|--------:|--------:|
|             0 | 3609.85 | 3568.59 | 3840.85 | 3229.05 |  nan    |  nan    |
|             1 | 3614.18 | 3459.33 | 3590.57 | 4435.54 |  nan    |  nan    |
|             2 | 3546.34 | 3365.41 | 3543.26 | 3736.29 | 4671.17 |  nan    |
|             3 | 3558.33 | 3377.87 | 3453.2  | 3791.37 | 3737.49 |  nan    |
|             4 | 3451.12 | 3302.13 | 3487.54 | 3643.63 | 4126.33 | 4320.46 |
|             5 | 3425.73 | 3420.84 | 3431.95 | 3873.22 | 4023.9  |  nan    |
|             6 | 3822.63 | 3338.37 | 4066.1  | 3820.12 | 3134.91 |  nan    |
|             7 |  nan    |  nan    | 3204.86 |  nan    |  nan    | 3589.74 |

# Assessment of Missingness

## NMAR Analysis

Looking at the cleaned League of Legends match dataset, I do not believe that there is a column which is NMAR (not missing at random). After analyzing the missingness of all of the columns in my dataset, the only columns which contain missing data are `damage mitigated/min` and columns containing early game metrics (`goldat10`, `xpat10`, `csat10`, etc.). Further exploration of these columns shows that their missingness is dependent on other columns, so their missingness is MAR (missing at random). For example, the early game metrics only seem to be present in games of higher leagues, which makes sense, since games in lower leagues are likely to be less competitive and engaging than games played in high tiers, and thus, early game metrics are not collected for these games.

## Missingness Dependency

We know that the missingness for early game metrics depends on the league that a game was played in, but what about `damage mitigated/min`? Does this column's missingness depend on any other columns? Let's first see how the distribution of `kills` looks when it is dependent on the missingness of `damage mitigated/min`.
- Null Hypothesis: The missingness of `damage mitigated/min` does not depend on the values in `kills`.
- Alternative Hypothesis: The missingness of `damage mitigated/min` depends on values in `kills`.
- Test Statistic: KS-statistic.

<iframe
  src="assets/kills_by_missing_dmg_mit_plot.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

After running the permutation test, I found an extremely small p-value of 8.95 × 10^(-10), so I reject the null and conclude that the missingness of `damage mitigated/min` **does** depend on the values in `kills`, so `damage mitigated/min` is MAR, conditional on `kills`.

Now, let's see how the distribution of `dragonkills` looks like when it is dependent on the missingness of `damage mitigated/min`.

<iframe
  src="assets/dragon_kills_by_missing_dmg_mit_plot.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

I want to determine if the missingness of `damage mitigated/min` depends on `dragonkills`, so I will conduct a permutation test.
- Null Hypothesis: The missingness of `damage mitigated/min` does not depend on the values in `dragonkills`.
- Alternative Hypothesis: The missingness of `damage mitigated/min` depends on values in `dragonkills`.
- Test Statistic: Difference in means.

<iframe
  src="assets/dist_of_means_dragon_kills_vs_dmg_mit_plot.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

The result of the permutation test gives a p-value of 0.314, so I fail to reject the null hypothesis and conclude that the missingness of `damage mitigated/min` **does not** depend on the values in `dragonkills`. This means that `damage mitigated/min` is MCAR (missing completely at random) when the column is conditioned on `dragonkills`.
