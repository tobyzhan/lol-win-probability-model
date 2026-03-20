---
layout: default
title: Predicting League of Legends Match Outcomes from Champion Drafts
---

# Predicting League of Legends Match Outcomes from Champion Drafts

**Toby Zhang**

This project studies whether champion draft choices in professional League of Legends matches can help explain and predict whether a team wins.

## Introduction

In this project, I studied whether champion draft choices in professional League of Legends matches can help predict whether a team wins. In particular, I focused on how champion picks and bans relate to match outcomes, and whether draft information contains useful predictive signal before the game begins.

This question is interesting because champion select is one of the most important strategic parts of a League of Legends match. Teams make choices about which champions to pick and ban before the game starts, and these decisions can shape team composition, playstyle, and matchup advantages. Because of this, draft data provides a natural setting for both hypothesis testing and prediction.

The dataset comes from Oracle’s Elixir and contains professional League of Legends match data. After filtering to complete team-level records, the main dataset used for analysis contained **17,616 rows** and **165 columns**.

The most relevant columns for this project were:

- `pick1` through `pick5`: the champions picked by a team
- `ban1` through `ban5`: the champions banned by a team
- `result`: whether the team won or lost
- `side`: whether the team played on blue side or red side
- `league`: the professional league the match belongs to
- `patch`: the game patch used for the match

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To prepare the dataset for analysis, I first filtered the original data to only include rows where `datacompleteness == 'complete'`. I did this so that my analysis would focus on matches with more reliable recorded information.

Because my project focuses on draft decisions and team outcomes, I then restricted the main dataset used in the rest of the analysis to rows where `position == 'team'`. This is important because champion picks, bans, side, and match results are team-level variables.

After cleaning, I worked with a complete team-level dataset that still contained all of the original columns, including the draft-related columns most relevant to my analysis such as `pick1` through `pick5`, `ban1` through `ban5`, `side`, `patch`, `league`, and `result`.

### Univariate Analysis

My first univariate visualization shows the most frequently picked champions. This helps identify which champions appeared most often in drafts and gives context for champion popularity in the dataset.

My second univariate visualization shows the most frequently banned champions. This helps identify which champions were removed most often during champion select and gives a sense of which champions were most contested in the draft.

Together, these plots suggest that drafting is concentrated around a smaller set of highly important champions.

<iframe src="{{ '/assets/plots/most-picked-champions.html' | relative_url }}" width="100%" height="500" frameborder="0"></iframe>

<iframe src="{{ '/assets/plots/most-banned-champions.html' | relative_url }}" width="100%" height="500" frameborder="0"></iframe>

### Bivariate Analysis

For my first bivariate analysis, I examined the relationship between banned champions and team win rate by grouping teams according to which champion they banned and calculating the corresponding win rate. This helps show whether banning certain champions is associated with stronger or weaker outcomes.

<iframe src="{{ '/assets/plots/top-bans-smoothed-winrate.html' | relative_url }}" width="100%" height="500" frameborder="0"></iframe>

For my second bivariate analysis, I examined how win rate differs by `side`. In the cleaned team-level dataset, blue-side teams had a win rate of about **0.529**, while red-side teams had a win rate of about **0.471**. This is useful context because side is known before the game begins and may influence match outcomes.

<iframe src="{{ '/assets/plots/win-rate-by-side.html' | relative_url }}" width="100%" height="500" frameborder="0"></iframe>

### Grouped Table

I created a grouped table summarizing banned champions by:

- `games_banned`
- `win_rate`
- `smoothed_winrate`

This table is useful because it combines both frequency and performance. Raw win rates alone can be misleading when a champion was banned only a small number of times, so the smoothed win rate provides a more stable summary.

| champion | games_banned | win_rate | smoothed_winrate |
|---|---:|---:|---:|
| Ashe | 4295 | 0.4866 | 0.4880 |
| Rumble | 4094 | 0.4787 | 0.4811 |
| Kalista | 3251 | 0.4599 | 0.4652 |
| Vi | 2702 | 0.5100 | 0.5084 |
| Varus | 2450 | 0.5029 | 0.5024 |
| Tristana | 2384 | 0.4690 | 0.4743 |
| Skarner | 2152 | 0.5014 | 0.5011 |
| Orianna | 2122 | 0.5231 | 0.5187 |
| Nautilus | 1928 | 0.5290 | 0.5231 |
| Maokai | 1889 | 0.5156 | 0.5123 |

### Summary

Overall, the exploratory analysis shows that draft decisions are concentrated around a relatively small group of highly contested champions. It also suggests that the relationship between bans and winning is more nuanced than raw win rate alone, which motivates the later hypothesis testing and prediction steps.

## Assessment of Missingness

One column with missing values in my dataset is `pick1`.

A possible **NMAR** explanation is:

- draft information may be missing because some match records were not fully recorded or extracted
- if that is true, the missingness could depend on information not directly observed in the dataset

To study missingness more carefully, I tested whether the missingness of `pick1` depends on other observed columns. Since `league` and `side` are both categorical, I used **total variation distance (TVD)** as the test statistic in permutation tests.

### Test 1: Missingness of `pick1` vs `league`

- **Null hypothesis:** The missingness of `pick1` does not depend on `league`.
- **Alternative hypothesis:** The missingness of `pick1` does depend on `league`.
- **Observed TVD:** **0.0411**
- **p-value:** **0.0**

Conclusion:

- Since the p-value is less than **0.05**, I reject the null hypothesis.
- This suggests that the missingness of `pick1` appears to depend on `league`.

### Test 2: Missingness of `pick1` vs `side`

- **Null hypothesis:** The missingness of `pick1` does not depend on `side`.
- **Alternative hypothesis:** The missingness of `pick1` does depend on `side`.
- **Observed TVD:** **0.0**
- **p-value:** **1.0**

Conclusion:

- Since the p-value is greater than **0.05**, I fail to reject the null hypothesis.
- This suggests that there is not enough evidence that the missingness of `pick1` depends on `side`.

Overall, these results suggest that the missingness of `pick1` is related to `league` but not to `side`.

<iframe src="{{ '/assets/plots/missingness-plot.html' | relative_url }}" width="100%" height="500" frameborder="0"></iframe>

## Hypothesis Testing

In this step, I tested whether banning `Vi` is associated with a higher team win rate.

- **Null hypothesis:** Teams that ban `Vi` have the same win rate as teams overall in the dataset.
- **Alternative hypothesis:** Teams that ban `Vi` have a higher win rate than teams overall in the dataset.
- **Test statistic:** The observed win rate of teams that banned `Vi`.

To test this, I simulated the null hypothesis using a binomial model with the overall win rate as the probability of success. I then compared the observed win rate for teams that banned `Vi` to the simulated null distribution.

- **Observed win rate for teams that banned `Vi`:** about **0.510**
- **p-value:** **0.1556**

Conclusion:

- Since the p-value is greater than **0.05**, I fail to reject the null hypothesis.
- There is not enough evidence to conclude that teams that ban `Vi` have a higher win rate than teams overall.

<iframe src="{{ '/assets/plots/hypothesis-test-plot.html' | relative_url }}" width="100%" height="500" frameborder="0"></iframe>

## Framing a Prediction Problem

In this project, I will build a **binary classification** model to predict whether a League of Legends team wins a match. The response variable is `result`, where `1` means a win and `0` means a loss.

This is a binary classification problem because each team has only two possible outcomes: win or lose.

At the **time of prediction**, I would only know information available before the game starts, specifically during champion select. Because of this, I will only use pre-game features such as the champions picked and banned by each team. I will not use in-game statistics like kills, gold, or damage, since those are not known before the match begins.

I will evaluate my model mainly using **accuracy**, because the target variable is binary and wins and losses are fairly balanced, so accuracy gives a simple and interpretable measure of how often the model predicts correctly. I will also report **F1-score** as a secondary metric, since it helps account for the balance between precision and recall.


## Baseline Model

For my baseline model, I used the draft-related columns `pick1` through `pick5` and `ban1` through `ban5` to predict the response variable `result`. These features are all nominal categorical variables, since champion names do not have a natural ordering, so I encoded them using one-hot encoding.

I used logistic regression as my baseline model because it is a standard and interpretable method for binary classification. I trained the model in a pipeline that first one-hot encodes the categorical features and then fits the logistic regression classifier.

To evaluate performance, I split the data into training and test sets and measured performance on the held-out test set. My baseline model achieved an accuracy of about **0.55**. Since this is a binary classification problem with outcomes of win or loss, this is slightly better than chance. This suggests that champion picks and bans contain some predictive information, but the model is still limited and leaves room for improvement.


## Final Model

To improve my baseline model, I kept the original draft features `pick1` through `pick5` and `ban1` through `ban5`, and I also included the pre-game columns `side` and `patch`. These are all known before the match begins, so they are valid features for prediction.

I then created two engineered features: `avg_pick_winrate` and `avg_pick_popularity`. The feature `avg_pick_winrate` is the average historical win rate of the five champions picked by a team, while `avg_pick_popularity` is the average frequency with which those champions were picked in the training data. I chose these features to summarize both the historical success and the overall popularity of a team’s draft.

To avoid leakage, I calculated these engineered features using only the training data. I again used logistic regression and tuned the hyperparameter `C` using `GridSearchCV` with 5-fold cross-validation.

The final model achieved a test accuracy of about **0.542** and an F1-score of about **0.542**, which is slightly worse than the baseline model. One possible reason is that the baseline already used the full draft information through one-hot encoded picks and bans, which is more detailed than these summary features. In contrast, the engineered features compress the draft into averages, which may lose useful information about specific champion combinations. These added features therefore did not improve performance in this case.


<iframe src="{{ '/assets/plots/final-model-plot.html' | relative_url }}" width="100%" height="500" frameborder="0"></iframe>

## Fairness Analysis

For my fairness analysis, I evaluated whether my final model performs similarly for teams on blue side and teams on red side. I used `side` to define the two groups and measured model performance using **accuracy**.

- **Null hypothesis:** The model is equally accurate for blue-side teams and red-side teams, and any observed difference is due to random chance.
- **Alternative hypothesis:** The model is not equally accurate for blue-side teams and red-side teams.


The test statistic is the absolute difference in accuracy between blue-side teams and red-side teams on the test set. In my test set, the model’s accuracy was **0.547** for blue-side teams and **0.537** for red-side teams, for an observed difference of **0.010**. I used a permutation test by randomly shuffling the `side` labels many times and recomputing the difference in group accuracies each time to simulate the null distribution.

After running the permutation test, I obtained a p-value of **0.57**. Since this p-value is greater than my significance level of **0.05**, I fail to reject the null hypothesis. This suggests that there is not statistically significant evidence that my model performs differently for blue-side teams and red-side teams.


<iframe src="{{ '/assets/plots/fairness-plot.html' | relative_url }}" width="100%" height="500" frameborder="0"></iframe>
