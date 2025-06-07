---
title: League of Legends Ward Correlation Analysis
---

# **League of Legends Ward Correlation Analysis**

Welcome to our data science project exploring optimal ward placements in professional League of Legends games. 

## **Introduction**

**League of Legends (LoL)** is a highly strategic, team-based game in which two teams of five players compete to destroy the opposing team’s base. Success can be influenced not only on individual skill but also on factors like map control, resource management, and coordinated teamwork.

For this project, we are going to analyze match data from Oracle’s Elixir, a premier database for professional LoL esports stats, covering the 2024 competitive season. The dataset includes match outcomes and detailed statistics such as kills, deaths, assists, team gold, dragons secured, towers destroyed, and advanced vision metrics like wards placed and wards cleared.

Our primary research questions focus on the **impact of objective control and vision control on win rates**. For example, we wondered: How does objective control influence win rates? How much can vision control influence win rates? Are there particular metrics that are especially predictive of success? Eventually, we decided to explore the following question further:
**What is the optimal number of wards placed per minute (WPM) that maximizes a team’s probability of winning?**

In LoL, wards are vision tools used to reveal parts of the map, helping teams track enemy movements and avoid ambushes. While strong vision control is generally viewed as beneficial, it's unclear whether more warding always leads to better outcomes. Excessive warding may lead to resource inefficiencies, while too little can compromise map awareness.

By analyzing the relationship between WPM (wards placed per minute) and match outcomes, we aim to identify whether there's an optimal rate for WPM that most effectively contributes to winning — and if there's a point where additional warding yields diminishing returns.

Here, we will take a first look at our only dataset `df`: A League of Legends esports match data taken from OracleElixir during 2024.

<div style="overflow-x: auto; white-space: nowrap;">

| gameid             | datacompleteness   | side   | position   | playerid                                  | teamid                                  |   gamelength |   result |   wardsplaced |    wpm |   controlwardsbought |   visionscore |   vspm |   earned gpm |
|:-------------------|:-------------------|:-------|:-----------|:------------------------------------------|:----------------------------------------|-------------:|---------:|--------------:|-------:|---------------------:|--------------:|-------:|-------------:|
| 10660-10660_game_1 | partial            | Blue   | top        | oe:player:65ed20b21e2993fb00dbd21a2fd991b | oe:team:a9145b7711873f53e610fbba0493484 |         1886 |        0 |            14 | 0.4454 |                    5 |            24 | 0.7635 |     221.421  |
| 10660-10660_game_1 | partial            | Blue   | jng        | oe:player:57da8dfcfbdb4e5b019fe93003db1c4 | oe:team:a9145b7711873f53e610fbba0493484 |         1886 |        0 |            10 | 0.3181 |                   10 |            39 | 1.2407 |     143.574  |
| 10660-10660_game_1 | partial            | Blue   | mid        | oe:player:71e79ef80600d398d90cfebe3b0b758 | oe:team:a9145b7711873f53e610fbba0493484 |         1886 |        0 |             4 | 0.1273 |                    2 |            31 | 0.9862 |     210.605  |
| 10660-10660_game_1 | partial            | Blue   | bot        | oe:player:867e8957fae1cb59f0808dbcc3aada2 | oe:team:a9145b7711873f53e610fbba0493484 |         1886 |        0 |            22 | 0.6999 |                    4 |            44 | 1.3998 |     257.72   |
| 10660-10660_game_1 | partial            | Blue   | sup        | oe:player:a74c2977c1fc826e9e7bdb6b224a141 | oe:team:a9145b7711873f53e610fbba0493484 |         1886 |        0 |            47 | 1.4952 |                   12 |           111 | 3.5313 |      98.5578 |

</div>

This dataset seems quite impressive. Note that this is just a portion of the entire dataframe. There are 117647 rows, meaning that many entries reprsenting a player during a specific match. More importantly, there seems to be a lot of columns... **161** to be exact. Most of these columns are irrelevant at this stage of investigating our question, so we will include only examine, clean, and plot the relevant columns for now for a better understanding.

With a simple for loop, we can manually search and extract the relevant columns through a scrollable element:

```
for i in df.columns:
    print(i)
```

Upon a manual inspection, some columns appear to be relevant:

1. **`gameid`** (string): the id of the match that two teams played in against each other.
2. **`playerid`** (string): the player on that team for the current match.
3. **`teamid`** (string): the id of the team that played in that match.
4. **`wardsplaced`** (int): the total number of wards placed by everyone in that team for the entire match.
5. **`gamelength`** (int): the number of seconds that the match took.
6. **`wpm`** (float): the rate of number of wards placed per minute.
  - Note: this column was already given to us.
  - However, a person can get a similar value by converting the gamelength value into minutes, and then dividing wardsplaced by gamelength in minutes.
  - Example: 1886 seconds * 60 = 31.43 minutes.
  - 14 wards placed / 31.43 minutes = 0.45 wards placed per minute
7. **`result`** (int): whether a team won that match. 1 represents a win, 0 reprsents a loss.

Below is `relevant_df`, a dataframe with only the above 7 columns that are relevant for now.

<div style="overflow-x: auto; white-space: nowrap;">
  
| gameid             | playerid                                  | teamid                                  |   wardsplaced |   gamelength |    wpm |   result |
|:-------------------|:------------------------------------------|:----------------------------------------|--------------:|-------------:|-------:|---------:|
| 10660-10660_game_1 | oe:player:65ed20b21e2993fb00dbd21a2fd991b | oe:team:a9145b7711873f53e610fbba0493484 |            14 |         1886 | 0.4454 |        0 |
| 10660-10660_game_1 | oe:player:57da8dfcfbdb4e5b019fe93003db1c4 | oe:team:a9145b7711873f53e610fbba0493484 |            10 |         1886 | 0.3181 |        0 |
| 10660-10660_game_1 | oe:player:71e79ef80600d398d90cfebe3b0b758 | oe:team:a9145b7711873f53e610fbba0493484 |             4 |         1886 | 0.1273 |        0 |
| 10660-10660_game_1 | oe:player:867e8957fae1cb59f0808dbcc3aada2 | oe:team:a9145b7711873f53e610fbba0493484 |            22 |         1886 | 0.6999 |        0 |
| 10660-10660_game_1 | oe:player:a74c2977c1fc826e9e7bdb6b224a141 | oe:team:a9145b7711873f53e610fbba0493484 |            47 |         1886 | 1.4952 |        0 |
  
</div>

## **Data Cleaning and Exploratory Data Analysis**

We can now start cleaning this data frame by ensuring the following:

1. We must verify that `wardsplaced`, `gamelength`, and `wpm` are all greater than or equal to 0.
  - This is because there cannot be a negative number of wards placed in a game, the minimum is 0.
  - Likewise a game must last at least greater than 0 seconds. This makes sense intutively, and also because to calculate wpm, we cannot divide by 0.
  - And finally, `wpm`, representing a proportion, cannot be negative.
2. We must verify that all wpm is an accurate reprsentation of the wardsplaced and gamelength columns.
  - This means we must check whether `wpm` = `wardsplaced` / (`gamelength` / 60)
3. We must verify that the `results` column consists of either only 1s or 0s
  - Anything other than that would be incorrect, as it wouldn't represent a win or loss.

On a side note, we are leaving `gameid`, `playerid`, and `teamid` alone for now. They are strings with no inherent restrictions on them, unlike the columns with numerical representation of certain values. Even if there is missing data, we will address them in a later section.

### After Verification

**It turns out, there are no incorrectly formatted data within our relevant dataset that needs to be corrected at this time.** This means out dataset was already given to us clean. Thank goodness!

Before, we can begin aggregating this dataframe, we need to understand the influence of the column playerid. Let's take a snippet of just ONE game for example. We will use gameid `10660-10660_game_1` for this demonstration.

<div style="overflow-x: auto; white-space: nowrap;">

| gameid             | playerid                                  | teamid                                  |   wardsplaced |   gamelength |    wpm |   result |
|:-------------------|:------------------------------------------|:----------------------------------------|--------------:|-------------:|-------:|---------:|
| 10660-10660_game_1 | oe:player:65ed20b21e2993fb00dbd21a2fd991b | oe:team:a9145b7711873f53e610fbba0493484 |            14 |         1886 | 0.4454 |        0 |
| 10660-10660_game_1 | oe:player:57da8dfcfbdb4e5b019fe93003db1c4 | oe:team:a9145b7711873f53e610fbba0493484 |            10 |         1886 | 0.3181 |        0 |
| 10660-10660_game_1 | oe:player:71e79ef80600d398d90cfebe3b0b758 | oe:team:a9145b7711873f53e610fbba0493484 |             4 |         1886 | 0.1273 |        0 |
| 10660-10660_game_1 | oe:player:867e8957fae1cb59f0808dbcc3aada2 | oe:team:a9145b7711873f53e610fbba0493484 |            22 |         1886 | 0.6999 |        0 |
| 10660-10660_game_1 | oe:player:a74c2977c1fc826e9e7bdb6b224a141 | oe:team:a9145b7711873f53e610fbba0493484 |            47 |         1886 | 1.4952 |        0 |

</div>

Notice how each row represents a PLAYER'S statistics. Our goal is to find the optimal wards placed per minute by TEAM overall. Each player on a team has different roles, so naturally some players may place a different rate of wpm than others depending on how much they are supposed to support the team.

Thus, we will groupby the relevant dataframe to show us each team's average wpm, and whether they won or lost, in a specific game.

*Note: if a team won or lost, then all players in their team should share the same result value. Thus, results can only be a 1 or 0.*

### **Aggregation**

Let's now aggregate `relevant_df`.

<div style="overflow-x: auto; white-space: nowrap;">

| gameid             | teamid                                  |     wpm |   result |
|:-------------------|:----------------------------------------|--------:|---------:|
| 10660-10660_game_1 | oe:team:8516ca63facc91286d6c00212ca945e | 1.29372 |        1 |
| 10660-10660_game_1 | oe:team:a9145b7711873f53e610fbba0493484 | 1.02863 |        0 |
| 10660-10660_game_2 | oe:team:8516ca63facc91286d6c00212ca945e | 1.21405 |        1 |
| 10660-10660_game_2 | oe:team:a9145b7711873f53e610fbba0493484 | 0.921   |        0 |
| 10660-10660_game_3 | oe:team:8516ca63facc91286d6c00212ca945e | 1.13293 |        0 |

</div>

This looks way more organized! We also double checked to make sure that `result` only consisted of 1s or 0s. Let's perform a univarate analysis just to visualize the distribution of wpm, regardless of whether a team won or lost.

Now, `wpm` being a proportion makes itself a continuous variable, unlike `wardsplaced` or `gamelength` which are represented by discrete values. So, we will be visualizing these distributions with a KDE and box plot.

### **Univariate Analysis**

<iframe
  src="assets/kde-wpm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="assets/box-wpm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Now, let's see if there is a glaring difference at first sight between the wdm of teams that won, and the wdm of teams that lost.

### **Bivariate Analysis**

<iframe
  src="assets/kde-wpm-result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="assets/box-wpm-result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Hmmm... it seems like there is only a slight difference at a visual glance. However, we cannot conclude anything decisive yet, because we need to properly test it. Also, we need to take in account for the large number of samples when looking at this "small" difference...

Before we continue, we will now refer to the `grouped` dataframe as `wards_df` from this point on, unless we specifically need to refer back to `grouped` for some reason. This is because it helps organize our dataframes, and it sounds more meaningful than `grouped` for the context of this project.

## **Assessment of Missingness**

When we initially created the `ward_df` DataFrame (previously referred to as `grouped`), we made a critical design choice during aggregation: we excluded rows with missing values in teamid. Our reasoning at the time was practical: including incomplete team data could distort the distributions of key metrics like wards placed per minute (wpm) or match outcomes (result). For example, if a team disconnected mid-match or was partially recorded, its statistics might not reflect actual gameplay and could **introduce noise or bias** into our analysis.

However, this decision raises an important question: **was the missingness of teamid random, or was it systematically related to other variables in our dataset?** If it was not random (NMAR), then excluding those rows — even for good reasons — could unintentionally bias our results, since there is probably a good reason why those team ids are missing that we haven't seen clearly yet. 

In our earlier cleaning steps, we focused on ensuring that quantitative columns like wpm, result, and wardsplaced contained no missing values. We did not yet explore missingness in columns as `gameid`, `playerid`, and `teamid`, because of their irrelevance to aggregating the data at the time. Now, as we transition to a deeper analysis of missing data, we will revisit the full `relevant_df` and `df` to formally investigate the missingness mechanisms behind `teamid` and how they might affect our conclusions.

**As it turns out, only `playerid` and `teamid` has missing values.** Precisely, `playerid` has 20660 null values in `relevant_df`, and `teamid` has 2712 missing values. Let's analyze `playerid` first before we move on.

### **NMAR Analysis**

At first glance, the column `playerid` appears to be Not Missing At Random (NMAR). There are several reasons to suspect this.

First, there are many more missing `playerid` values than there are missing `teamid` values. So implies that `playerid` can be **missing independently of `teamid`** — which is important context. Moreover, many rows with missing `playerid` still contain a valid `teamid` and complete gameplay statistics — such as `wardsplaced`, `wpm`, and `result`. This suggests that the missingness is not tied to overall data corruption completely, but rather it could be to the identity of the player itself.

**Without additional context**, it seems the missingness can be tied to the player's identity itself. Examples that could explain this could be due to data privacy settings, untracked players, or deleted accounts. These are unobservable from within the dataset, which fits the definition of NMAR.

However, it is important to note that we are only basing our conclusion given only the column context within `wards_df`. If we were to incorporate broader metadata from the full dataset, such as datacompleteness, league, or split, we might find patterns that explain the missingness of `playerid`. For example, matches flagged as "partial" in the column `datacompleteness` **may** (not guaranteed though, because we don't know what exactly is missing) be systematically missing player identity information. In that case, the missingness could be modeled by observed variables, and `playerid` would instead be considered Missing At Random (MAR).

Thus, our conclusion is that `playerid` is NMAR without additional context outside of `relevant_df`, but it could be MAR when we incorporate metadata from the full dataset.

### **Missingness Dependency**

Next, we will explore the missingness of `teamid`.

Unlike playerid, `teamid` is central to our primary research question: how **team-level statistics** like wpm relate to winning. This makes its missingness more **consequential** — if rows with missing teamid differ systematically, their exclusion could bias our analysis. In other words, it really doesn't matter what type of missingness `playerid` has, since the objective of our main analysis does not care for it. That column was only supposed to give us a visual representation of how each team has multiple rows of data that needed to be aggregated into. The more imporant column of interst is `teamid`

While teamid also has missing values in **relevant_df**, we realized that to properly investigate its missingness mechanism, we needed to go beyond **relevant_df**. Many of the variables such as match level data like `datacompleteness` were likely to explain its absence. Othe arbitrary gameplay statistics like `earnedgpm` could also be analyzed in context too, but these columns were only available in the original full dataset (df).

Therefore, we extended our analysis and performed permutation tests using variables from the full dataset. Our goal was to determine whether the missingness of `teamid` is Missing Completely At Random (MCAR), Missing At Random (MAR), or potentially NMAR. If the missingness can be explained by known columns, it would support a MAR classification and justify our earlier decision to drop rows with missing teamid in `wards_df` (aka `grouped`). If not, we would need to re-evaluate our cleaning method.

The following permutation tests evaluate whether `teamid` missingness depends on observed columns.

### Permutation Tests: MCAR vs MAR

Our hypothesis is that `teamid` is Missing At Random (MAR), because it would justify our decision to remove those missing rows. To test this, we examined whether the missingness of teamid depends on:

- Another column in `df` that should be related (like `datacompleteness`, since if the data is only partially complete is missing, we might expect teamid to be incomplete)

- Another column in `df` that should not be related (like earnedgpm, since earned gold per minute shouldn't logically influence whether a team's name is recorded or not)

We conducted permutation tests to investigate whether the distributions of these variables change depending on whether teamid is missing or not. The basic idea of these permutaion tests is to get evidence that if our p-value is low, then the column is likely MAR. If it is not low, then the column is likely MCAR. Below, we present and interpret our results. **We set our alpha threshold at standard 0.05**.

**Note: We have to be careful on which columns we select to test!** If the column we are testing against also has missing values that overlap with teamid, then our permutation test can give artificially low p-values, not because there's a true association. This is because both columns could be missing in the same rows. We can  find which columns to pick using code.

### Earned GPM vs. TeamID Missingness

We formally define the following:
1. The distribution of `earned gpm` when `teamid` is missing.
2. The distribution of `earned gpm` when `teamid` is not missing.

- **Null Hypothesis (H₀)**: The distributions are similar, and the missingness of `teamid` is not dependent on the earned gold per minute `earned gpm`. (MCAR)
- **Alternative Hypothesis (Hₐ)**: The distributions are not similar, and the missingness of `teamid` is dependent on the earned gold per minute `earned gpm`. (MAR)
- **Test Statistic**: The difference in mean `earned gpm` between groups where `teamid` is missing, and not missing. We chose this because we needed a test statistic that can measure how different two numerical distributions are.
- **Significance Level**: 0.05

<iframe
  src="assets/permutation-earned-gpm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**The p-value was: 0.0860, fail to reject the null hypothesis.**

### Data Completeness vs. TeamID Missingness

We formally define the following:
1. The distribution of `datacompleteness` when `teamid` is missing.
2. The distribution of `datacompleteness` when `teamid` is not missing.

- **Null Hypothesis (H₀)**: The distributions are similar, and the missingness of `teamid` is not dependent on whether the `datacompleteness` for that entry is complete. (MCAR)
- **Alternative Hypothesis (Hₐ)**: The distributions are not similar, and the missingness of `teamid` is dependent on whether the `datacompleteness` for that entry is complete. (MAR)
- **Test Statistic**: The difference in mean `datacompleteness` (after converting to binary values) between groups where `teamid` is missing, and not missing. We chose this because we needed a test statistic that can measure how different two numerical distributions are.
- **Significance Level**: 0.05

<iframe
  src="assets/permutation-data-completeness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**The p-value was: 0.0000, reject the null hypothesis.**

### Results

After running our permutation tests:

The p-value for `earnedgpm` was above 0.05, which is not statistically significant at α = 0.05, suggesting we found at least one column without a meaningful relationship between missingness of teamid and gold earned per minute. This is a gameplay statistic unrelated to data collection quality.

The p-value for `datacompleteness` was below 0.05, indicating a strong relationship between whether a row has a missing teamid and whether the match was flagged as "complete" or "partial."

These results rule out MCAR (Missing Completely At Random) for `teamid`, because at least one observed column (`datacompleteness`) does explain the missingness. This supports the interpretation that teamid is Missing At Random (MAR). The probability of teamid being missing is highly related to other observed variables in the dataset (like metadata about match completeness), but not to unobservable factors.

Since the missingness is explainable by observed data, we can conclude that `teamid` does not have an NMAR missingness mechanism, and thus our earlier choice to drop rows with missing teamid is justified. Doing so likely does not introduce bias into our core analysis of WPM vs. win rate, because the mechanism driving the missingness is not related to the values of the statistics we're analyzing (like `earnedgpm` or `wpm`), but to data quality flags.

In contrast, we cannot make the same claim for `playerid` (as discussed previously), which may be NMAR due to factors that are not observable in the current context (`relevant_df`). Fortunately, since playerid is not central to our analysis, this does not impact the validity of our conclusions.

## **Hypothesis Testing**

In this section, to determine whether teams that place more wards per minute are more likely to win, we will now conduct a permutation test on `wards_df`. We will use a permutation test because we are trying to determine whether our two "samples" (the mean wpm from winners, and the mean wpm from the losers) were drawn from the same population. In other words, are those distributions similar by chance?

We formally define our null, alternative hypothesis, test statistic, and significance level as below:

- **Null Hypothesis (H₀)**: There is no relationship between a team's average Wards Placed per Minute (WPM) and their probability of winning. Any observed difference in average WPM between winning and losing teams is due to chance.
- **Alternative Hypothesis (Hₐ)**: Teams that win have higher average WPM than teams that lose. In other words, warding (as measured by WPM) is positively associated with win rate.
- **Test Statistic**: The difference in mean WPM between winning teams and losing teams. We chose this because we needed a test statistic that can measure how different two numerical distributions are (i.e. wpm from winners vs wpm from losers)
- **Significance Level**: 0.05

**Initial Statistics**:
- Mean WPM (Win): 1.1160762595287594
- Mean WPM (Loss): 1.070521721196868
- Observed Difference: 0.04555453833189138

<iframe
  src="assets/permutation-wpm-result.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Our permutation test yielded a p-value approximately equal to 0.00, that we reject the null hypothesis. We conclude that teams that win have a higher average wpm than teams that lose. 

In hindsight, this also makes sense intuitively. First, through a statistical perspective, although the visualiztion showed the distributions of mean_wpm from winners vs losers to be very similar, we have to account for the sheer size of the dataset. There are **19156** entries in `wards_df`. Even if the two distributions overlap a lot, the sheer size of our dataset (tens of thousands of games) increases our test's power. This can make even a small mean differences become detectable, and thus seem statistically significant.

Warding is a key strategic element in LoL. Teams that place more wards should, and now we confidently believe with evidence, gain better vision control, allowing them to track enemy movements, avoid ambushes, and secure objectives more effectively. This would increase the chances of winning. Therefore, it is expected that winning teams have consistently higher warding activity.

Hence, the extremely low p-value reflects a strong and consistent pattern: teams that ward more frequently are more likely to win, confirming the importance of vision control in LoL success.

## **Framing a Prediction Problem**

**Our goal is to predict whether a team wins a LoL match, given also other in-game features related to `wpm`, such as `side`, `position`, `vision score`, and of course, `wpm`.**

This is a **binary classification model**, where the response variable is `win` (1 for win, 0 for lose).
We will evaluate our models using accuracy, as our dataset is approximately balanced between the two classes.
Accuracy is appropriate in this context because it measures the proportion of correct predictions, which directly answers our modeling question.
If we find the classes are imbalanced, we will additionally report the F1-score to account for differences in precision and recall.
All features used in our model are available prior to the end of the match. We will not use any information that could reveal the outcome before prediction time.
Our modeling process follows a standard, reproducible pipeline:

1. Train/Test Split
We will split the data into training and test sets to separate model fitting from final evaluation.

2. Preprocessing and Pipeline
We will use ColumnTransformer and Pipeline to apply preprocessing (e.g., encoding categorical variables and scaling numeric ones) consistently across cross-validation and the final model.

3. Cross-Validation and Hyperparameter Tuning
We will apply grid search with cross-validation to tune model hyperparameters using only the training data.

4. Final Evaluation
After selecting the best model, we will evaluate its generalization performance on the held-out test set, which is only used once to report final metrics.

## **Baseline Model**

From now on, we need to approach this section with a different dataframe called `df_model`. The original dataframe `wards_df` showcased aggregated data that was appropriate for conducting the permutation tests in previous sections. However, our prediction problem demands us to use other features outside of `wards_df`. So here, we use `df_model`, a filtered version of the full dataset containing only complete observations (*in game play statistics, e.g. it ignores whether columns like `teamid` is filled or not, because it is unrelated to gameplay*), to train and evaluate a classification model.

Let's take a look at `df_model`. Note not all columns are shown, and this is a query that represents all rows that have `datacompleteness` as complete.

<div style="overflow-x: auto; white-space: nowrap;">

| side   | position   |   result |   wardsplaced |    wpm |   controlwardsbought |   visionscore |   vspm |
|:-------|:-----------|---------:|--------------:|-------:|---------------------:|--------------:|-------:|
| Blue   | top        |        1 |             7 | 0.2905 |                    3 |            17 | 0.7054 |
| Blue   | jng        |        1 |             7 | 0.2905 |                    7 |            40 | 1.6598 |
| Blue   | mid        |        1 |             6 | 0.249  |                    3 |            24 | 0.9959 |
| Blue   | bot        |        1 |            10 | 0.4149 |                    6 |            33 | 1.3693 |
| Blue   | sup        |        1 |            34 | 1.4108 |                   11 |            72 | 2.9876 |

</div>

This dataset allows us to include richer in-game features — such as:

1. **`side`** (str): which side the team was on ('Blue' or 'Red')
2. **`position`** (str): referring to team role or lane assignments..
3. **`wpm`** (float): the rate of number of wards placed per minute.
4. **`vspm`** (float): the rate of vision score per minute.

**Quantitative features**: 2

- *Discrete features*: 0 (none of our features were produced by counting)
- *Continuous features*: 2 (wpm and vspm)

**Categorical features**: 2

- *Ordinal features*: 0 (none of our features have a meaningful order, like ranks or ratings)
- *Nominal features*: 2 (side and position)

The goal is to predict the result (1 = win, 0 = loss), based on these features.

### Building the Model

We decided to build our model using only entries where the data is fully complete, as indicated by the `datacompleteness` column. Including rows that could potentially have missing crucial data in our scope of analysis could introduce **unwanted bias or noise** into the model. To avoid this, we filtered the dataset to retain only rows where `datacompleteness` == 'complete'.

For context, two of our input features: `side` and `position`, are categorical. To use them effectively in our model, we applied one-hot encoding, which converts each category into a binary feature. This allows categorical variables to be represented numerically and interpreted correctly by the model. The other features of interest, `wpm` and `vspm`, were already numerical. They were passed through without transformation.

We implemented this preprocessing using a ColumnTransformer, which applies different transformations to different feature types, and wrapped it into a pipeline for a clean and modular modeling workflow.

We followed a standard supervised learning workflow:

1. **Train/Test Split**:
We split the data into training and test sets using an 80/20 split, with stratify=y to preserve the class balance in both sets. This ensures that the distribution of wins and losses remains consistent across the training and testing partitions.

2. **Preprocessing**:
Since side and position are categorical variables, we **one-hot encode** them (dropping the first category to avoid multicollinearity). The numeric features (wpm, vspm) are passed through without transformation.

3. **Baseline Model — Decision Tree**:
We use a Decision Tree Classifier as our baseline model. The reason for using a decision tree is because **our predictive column `result` has binary representations(**. 1 = win, 0 = lose. So naturally, decision trees would be perfect for this job, because it is interpretable model that often serves as a good starting point in **binary classification tasks**.

4. **Pipeline**:
We combine preprocessing and modeling into a single Pipeline using make_pipeline, which helps ensure clean, repeatable transformations during both training and evaluation.

### Results

After building the model, here is what we found.

**Summary**:

- **Accuracy**: 0.538, overall correctness
- **Precision**: 0.538, how often predicted wins are correct
- **Recall**: 0.535, how many actual wins were caught
- **F1-score**: 0.537, a single score balancing precision & recall

Overall, the **quality of our baseline model** (without hyperparameter tuning) is **subpar, but expected**. Our model has an accuracy of only 0.538, meaning it can only **predict correctly 53.8% of the time**, which is almost as bad as flipping a coin. Our goal here was to just establish a working model with limited features. There is definitely room for improvement.

## **Final Model**

To improve upon our baseline model, we expanded the feature set to include additional vision-related variables:

**`wardsplaced`** (int): total number of wards placed

**`visionscore`** (int): an integer representing a broader measure of vision contribution

**`controlwardsbought`** (int): how many control wards were purchased

Based on these descriptions, we believe that these variables are relevant to `wpm`, and therefore may provide a new insight on improving the predictions and quality of our model. **These features capture a greater overall investment in vision and may provide more information than just per-minute metrics.** After all, our prediction goal remains the same: to determine whether a team wins based on features like `wpm` and related statistics. Since vision is a valid aspect of strategic gameplay in LoL (as demonstrated in previous sections with wpm), we believe these variables can strengthen our model's ability to predict match outcomes. These new features will be combined with those from the baseline model (side, position, wpm, vspm) to form the complete `features_final` set.

We will also reuse the same splitting strategy as in the baseline model:

1. 80/20 split into training and testing sets
2. Stratified sampling to maintain the class balance
3. Fixed random seed for reproducibility

This ensures a fair comparison between the baseline and final models, so we can evaluate our attempt at improving this model.

### Improvements from the Base Model

We have also updated our ColumnTransformer to accommodate the new feature types:

For more context about our features:

- `side` and `position` are **nominal categorical features**, and thus they need to be processed through one-hot encoding to get usable values (dropping first level)
- `wardsplaced`, `visionscore`, and `controlwardsbought` are discrete quantatative features; they need to be standardized using StandardScaler

The remaining numeric features (`wpm`, `vspm`) are passed through as-is.

We made this pipeline to ensure that preprocessing is applied consistently across both training and testing.

### Building the Final Model

For our final model, we continued using a **Decision Tree Classifier** due to its interpretability and ability to predict binary classifiers (i.e. `result`). However, unlike in the baseline model, we now perform **hyperparameter** tuning to improve the model's performance and avoid overfitting or underfitting.

The primary hyperparameter we want to tune is **max_depth**, which controls the maximum depth of the tree. We want to avoid a tree that is too shallow (which may underfit the model), and a tree that is too deep (which may overfit the model).

To find an appropriate depth, we use **GridSearchCV**, which performs an exhaustive search to find the optimal height/hyperparameter to use. In our case, we chose and tested five different depths: **[3, 5, 7, 10, 15]**.

To evaluate each option, we will use:

- A **5-fold cross-validation** (`cv=5`), which splits the training data into five parts and rotates the validation fold, ensuring a more reliable estimate of model performance

- **Accuracy as the scoring metric** (`scoring='accuracy'`), which aligns with our goal of maximizing correct predictions in a roughly balanced dataset

This process helps us select a model that generalizes well to new data by identifying the best tree depth in our given options.

### Results

After fitting the model with GridSearchCV, we used the best-found model to make predictions on the held-out test set.

Let's take a look at it's performance, and also create a confusion matrix to visualize these results.

<iframe
  src="assets/confusion-matrix-final-model.html"
  width="600"
  height="500"
  frameborder="0"
></iframe>

Accuracy: 0.606
Precision: 0.601
Recall: 0.628
F1-score: 0.615

What can we interpret from this?

The confusion matrix provides a breakdown of our Decision Tree model’s predictions:

- **Top-left (True Negatives)**: The number of times the model correctly predicted a loss.
- **Bottom-right (True Positives)**: The number of times the model correctly predicted a win.
- **Top-right (False Positives)**: Losses incorrectly predicted as wins.
- **Bottom-left (False Negatives)**: Wins incorrectly predicted as losses.

Judging based off of this, our final model accuracy seems to be mediocre, because although the number of false positives and false negatives are lower than the number of true positive and true negatives, it is low enough. There is only a thousand-ish difference between them. We expected more. However, it is acceptable nonentheless because it is an improvement over our base model.

**Summary**:

- The best cross-validated model used `max_depth = 10`.
- **Accuracy**: 0.606, overall correctness
- **Precision**: 0.601, how often predicted wins are correct
- **Recall**: 0.628, how many actual wins were caught
- **F1-score**: 0.614, a single score balancing precision & recall

These metrics show a clear improvement over the baseline model. **Accuracy has increased from 0.538 to 0.606, meaning our model can predict correctly about 60.6% of the time. This is a 6.8% improvement.** All other metrics have improved as well. This suggests that the new features provided additional predictive power regarding match outcome, especially in identifying wins (higher recall and F1-score for class 1). Although the accuracy is still modest (which means there is definitley room for future improvement), these gains are evidence that suggest the new features, especially those related to vision, add a predictive value to our final model.

## **Fairness Analysis**

In this final section, we will futher investigate the potential biases of our model. More specifically, we will investigate whether our model treats teams differently based on their side selection. In other words, whether it predicts wins more accurately for teams starting on the **Blue** (Group X) side versus the **Red** (Group Y) side.

In LoL, each match is split into two opposing teams: Red and Blue. Although both sides function symmetrically in layout at a first glance, their placements actually differ in objective positioning, and vision angles. These differences can **influence gameplay and outcomes**, which is why "side selection" is often a strategic consideration in professional play. We can further visualize this by taking a look at the `side` and `position` columns in our original dataset.

<div style="overflow-x: auto; white-space: nowrap;">

| side   | position   |
|:-------|:-----------|
| Blue   | top        |
| Blue   | jng        |
| Blue   | mid        |
| Blue   | bot        |
| Blue   | sup        |

</div>

In our fairness analysis, we will examine whether our model performs equitably across these two sides, or if it favors one over the other.

**We will focus on precision as our fairness metric**. Precision is especially relevant in this context because it measures the proportion of predicted wins that are actually wins. In other words, how trustworthy a "win" prediction is for each group.

### Observed Disparity

We will use scikit-learn's precision_score function to calculate the precision scores among the two groups.

- **Precision** (`Blue`): 0.627
- **Precision** (`Red`): 0.572
- **Observed Difference** (`Blue` − `Red`): 0.055

This slight ~5% difference suggests our model is more precise in predicting wins for Blue side teams than Red side teams.

Now, **to assess whether this difference could have arisen by chance, we ran a permutation test**. We randomly shuffled the side labels 1000 times and recalculated the precision difference each time. The resulting null distribution shows what kind of differences we'd expect if `side` had no impact on model precision.

Here is our permutation test formal declarations:

- **Null Hypothesis** (H₀): There is no difference in the model’s precision for predicting wins between Blue side teams and Red side teams.

    - In other words: Precision(Blue) == Precision(Red) 
 
- **Alternative Hypothesis** (Hₐ): There is a difference in the model’s precision between Blue side and Red side teams. 

    - In other words: Precision(Blue) == Precision(Red)

This is a two-sided test because we are open to finding a difference in either direction (i.e., either side could have higher precision).

Let's see how our observed difference of 0.055 falls in this distribution! We will use a standard significance level of 0.05.

<iframe
  src="assets/permutation-precision-diff.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

**The p-value was: 0.0000, reject the null hypothesis.**

Since our p-value is well below the alpha threshold of 0.05, **we reject the null hypothesis** and conclude that our model's precision is significantly different for Blue Side and Red Side. In particular, our model is more precise at predicting wins for Blue Side teams than for Red Side teams. **This suggests that our model may be unintentionally biased with towards teams depending on side selection**.

This finding raises a potential fairness concern: the model may be unintentionally biased toward teams starting on the Blue side. This could stem from:

- Systemic differences in how Blue and Red sides play (e.g. pick order advantages, objective positioning)
- Imbalanced data, if more Blue side teams win in the training set
- Feature encoding or gameplay dynamics that interact differently with side

Of course, other forms of bias may exist. For example, by league, region, or even individual players (if some are overrepresented in the data). A more comprehensive fairness exploration would be needed to evaluate these potential biases as well.

## Conclusion

In this project, we explored how League of Legends vision-related metrics, particularly wards placed per mite (wpm), relate to match outcomes in professional League of Legends games. We began by cleaning up a 2024 LoL Esports dataframe, handling missing data thoughtfully, and aggregated data in order to create a new, grouped, dataframe for further exploration.

We then used hypothesis testing to determine that wpm is significantly associated with a higher win rate, though the visual difference in distributions was subtle at a first glance. From there, we built a baseline classification model to predict match outcomes, using other gameplay features available that were related to wpm. We extended this into a final model by introducing more vision-related features, and tuning the model's hyperparameters using cross-validation and GridSearchCV.

Our final model showed modest improvement over the baseline (accuracy: 0.538 → 0.606), indicating these added features do improve predictive power. Finally, we conducted a fairness analysis and found the model is significantly more precise at predicting wins for Blue side teams than Red side, raising important questions about potential bias tied to gameplay dynamics or data imbalance.

Overall, this project gives an insight on preprocessing, statistical testing, and fairness analysis when building interpretable models in LoL esports analytics.