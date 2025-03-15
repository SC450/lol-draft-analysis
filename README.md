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
- **Null Hypothesis**: The missingness of `damage mitigated/min` does not depend on the values in `kills`.
- **Alternative Hypothesis**: The missingness of `damage mitigated/min` depends on values in `kills`.
- **Test Statistic**: KS-statistic.
- **Significance Level**: 0.05

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
- **Null Hypothesis**: The missingness of `damage mitigated/min` does not depend on the values in `dragonkills`.
- **Alternative Hypothesis**: The missingness of `damage mitigated/min` depends on values in `dragonkills`.
- **Test Statistic**: Difference in means.
- **Significance Level**: 0.05

<iframe
  src="assets/dist_of_means_dragon_kills_vs_dmg_mit_plot.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

The result of the permutation test gives a p-value of 0.314, so I fail to reject the null hypothesis and conclude that the missingness of `damage mitigated/min` **does not** depend on the values in `dragonkills`. This means that `damage mitigated/min` is MCAR (missing completely at random) when the column is conditioned on `dragonkills`.

## Hypothesis Testing

There are many factors that can contribute to a team's win or loss in a game. In this section, I seek to determine if damage mitigation is an important factor in deciding the outcome of a game. Here are the pair of hypotheses I will be testing, along with my test statistic:
- **Null Hypothesis**: Winning teams and losing teams both have the same average amount of damage mitigated per minute.
- **Alternative Hypothesis**: Winning teams have a significantly higher average amount of damage mitigated per minute than losing teams.
- **Test Statistic**: Difference in means of damage mitigated per minute between winning and losing teams.
- **Significance Level**: 0.05

Our observed statistic has a value of around 54. We need to conduct a hypothesis test to see if this value can be considered significant or not. To do this, we will:
- Generate samples from the League of Legends match data.
- For each generated sample, compute the difference in means between the sampled distribution and our actual (observed) distribution.
- Lastly, determine if our observed difference in means of 54 can be considered "extreme" or not in our empirical distribution of differences in means.

First, we should determine how big each sample should be. A good sample size should be representative of how many matches are played in a timeframe. For our purposes, a good timeframe will be a season, as there are multiple seasons in a year for competitive League of Legends. The `split` column in our raw League of Legends match data tells us the ranked season that a game was played in. Additionally, we will only use matches that have been played in playoffs when calculating our sample size, as playoffs are likely to be more competitive, and statistics like damage mitigation may matter more in these high-stakes games. Calculating the average amount of playoffs matches in a season, I obtain a sample size of 1600 matches. We can now proceed to running the hypothesis test.

<iframe
  src="assets/dist_of_means_dmg_mit_win_vs_lose_plot.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

The above histogram displays the distribution of generated differences in means from the hypothesis test. The p-value of this hypothesis test is 0.50044, so I fail to reject the null hypothesis.

Although our observed difference in means of `damage mitigated/min` between winning and losing teams shows that winning teams do have a higher amount of damage mitigation per minute than losing teams, our hypothesis test shows that winning teams do not have a **significantly** higher amount of damage mitigation per minute than losing teams. Recall that our null hypothesis is "Winning teams and losing teams both have the same average amount of damage mitigated per minute." The conclusion of our hypothesis test is that we fail to reject the null hypothesis. This could have a number of implications:
- Damage mitigation is not as an important of a factor in winning a game compared to other factors.
- Damage mitigation may be a "niche" strategy to winning a game.
- Teams can safely ignore damage mitigation when picking their champions, without jeopardizing their chances of winning a game.

We will ultimately have to conduct additional hypothesis tests to find out if these implications are true or not. However, the result of this hypothesis test is informative and may be interesting to some—I thought damage mitigation would have played a crucial role in determining which team wins a game.

# Framing a Prediction Problem

A team's picks and bans are crucial to determining the outcome of a game. At its core, League of Legends is a game about strategy and teamwork, and thus, choosing—and banning—champions that will coordinate well with each other can mean the difference between victory and defeat. In this section, I aim to build a binary classification model that predicts whether a team will win or lose a game, using a game's played and banned champions—that is, the picks and bans from both teams in a game—and the metrics of a game after 10 minutes of that game have passed. Concretely, I will be predicting the `victory` column using data from the `picks` and `bans` columns, as well as the 10 minute game metric columns (`goldat10`, `xpat10`, `csat10`, `opp_goldat10`, `opp_xpat10`, `opp_csat10`, `golddiffat10`, `xpdiffat10`, `csdiffat10`, `killsat10`, `assistsat10`, `deathsat10`, `opp_killsat10`, `opp_assistsat10`, and `opp_deathsat10`) in the cleaned 2024 League of Legends match data.

Why am I using these columns? Well, if the time of prediction was anytime before the start of a game, we only know each teams' chosen champions and their bans. This is very limited information to go off of; while a team's champion picks and bans are significant to determining whether or not they will win a game, players' skill and performance in the early game are also vital factors in affecting the outcome of a game. Using only the information of a team's `picks` and `bans` will result in a model with low accuracy. So, rather than just use the `picks` and `bans` columns solely, the next best thing to do is to use the earliest possible metrics we can obtain of a game, so that there is significantly more information to go off of. The earliest we can obtain information about a game's metrics is 10 minutes after a game has started. Thus, my time of prediction will be **10 minutes after a game has started**. All of the other columns (besides `team` and `gameid`) in the cleaned 2024 League of Legends match data contain data that can only be formulated **after** a game concludes.

As for why I want to predict the `victory` variable, a motivating goal here is to find any potential patterns in `picks` and `bans`, and figure out which combinations of picks and bans are most likely to result in victory or defeat. The challenge is to make the most out of what data I can extract from `picks`, `bans`, and early game data, and create a fairly accurate model using just those extracted features. Therefore, I do not want to rely on any features other than `picks`, `bans`, and early game metrics. I want to build a model with the highest accuracy possible whilst limiting a game's information as much as I can. Accuracy will be the most suitable metric for my model, as the values of `victory` are evenly split; 50% of the values in this column are `True`, and 50% of the values are `False`. Because imbalance of values is not an issue here, metrics like F1 score or metrics for probability predictions, like log loss, will not be the most suitable metrics for my model.

# Baseline Model

To create my baseline model, I used a `SGDClassifier`, which is a linear classification model that is optimized by stochastic gradient descent. The features for my model are as follows: 
- Nominal features: `picks` and `bans`
- Quantitative features: `goldat10`, `xpat10`, `csat10`, `opp_goldat10`, `opp_xpat10`, `opp_csat10`, `golddiffat10`, `xpdiffat10`, `csdiffat10`, `killsat10`, `assistsat10`, `deathsat10`, `opp_killsat10`, `opp_assistsat10`, and `opp_deathsat10`

In total, I am using 2 nominal features, 15 quantitative features, and no ordinal features. For the transformation of my data, I will only be transforming the `picks` and `bans` columns, as they are in a unique data format where values are lists of five strings. The rest of the columns are quantitative and are already in a usable format. To transform the `picks` and `bans` columns, I one-hot encoded them using a custom `ListOneHotEncoder` class that I created using the `BaseEstimator` and `TransformerMixin` classes from `scikit-learn`. This process transformed `picks` and `bans` so that each element in the lists of these columns would be one-hot encoded individually, rather than directly one-hot encoding the lists themselves. 

The accuracy score of my baseline model is 0.64, which sounds decent, but still not quite good in my opinion. Given a total of 17 features, I would have expected my baseline model to perform slightly better, but the low accuracy is understandable, as this model is just a baseline model, and is not fully optimized to the dataset.
