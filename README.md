## 2020 Singapore General Election Sentiments across Two Social Media Platforms

### David Tan & [Fabian Kong](https://github.com/fabskong)
### 02 August 2020

### Introduction

The Parliament in Singapore may be dissolved before the expiry of its five-year term by the President on the advice of the Prime Minister. The general election must be held within three months of the Parliament’s dissolution. There are two types of constituency or electoral division: single member constituency (SMC) and group representation constituency (GRC). SMC, as the name suggests, has only one member of parliament (MP), while GRC has three to six MPs. The GRC scheme was introduced in the 1988 general election to ensure that minority groups are represented in Parliament.<sup>[1](#footnote1)</sup>

Below are the percentage of votes between incumbent and opposition, and of voters turnout for parliamentary general elections between 1988 (when GRC scheme is introduced) and 2020. Data for 2015 and earlier are scraped from Data.gov.sg using CKAN API while data for 2020 is scraped from Wikipedia (which is sourced from Elections Department Singapore and Channel News Asia) using rvest package.

![GE Plot](/image/ge_plot.png)

The election process consists of the following:

Nomination Day - Prospective candidates submit their nomination documents and deposits between 11:00 am and 12:00 pm. If there is only one candidate or one group of candidates for SMC and GRC respectively, it will be declared that the candidate or the group of candidates have been elected as MP(s). This is known as walkover.

Campaigning - Candidates can only campaign up to the start of Cooling-off Day.

Cooling-off Day - This is the eve before Polling Day. Campaigning is prohibited, and the electorate is given some time to reflect on the issues raised before going to the polls.

Polling Day - Electors will go to their allotted polling stations to cast their vote any time between 8:00 am to 8:00 pm. After the close of polls, ballot boxes will be sealed and transported to counting centres.

The election process consists of the following:

1. Nomination Day - Prospective candidates submit their nomination documents and deposits between 11:00 am and 12:00 pm. If there is only one candidate or one group of candidates for SMC and GRC respectively, it will be declared that the candidate or the group of candidates have been elected as MP(s). This is known as walkover.

2. Campaigning - Candidates can only campaign up to the start of Cooling-off Day.

3. Cooling-off Day - This is the eve before Polling Day. Campaigning is prohibited, and the electorate is given some time to reflect on the issues raised before going to the polls.

4. Polling Day - Electors will go to their allotted polling stations to cast their vote any time between 8:00 am to 8:00 pm. After the close of polls, ballot boxes will be sealed and transported to counting centres.

For 2020 General Election, Nomination Day is 30 June 2020 and Polling Day is 10 July 2020 with nine days of campaigning. Therefore, Cooling-off Day is 9 July 2020. The COVID-19 pandemic has changed how an election is done in 2020. There are no public rallies, electors are encouraged to cast their votes within their two-hour time slot with safe distancing measures, and polling hours were extended until 10:00 pm.

In our project the election process is split into three distinct phases:

* First phase: Dissolution of Parliment Date to Nomination Date
* Second phase: Day after Nomination Date to Polling Date
* Third phase: Day after Polling Date to the subsequent day

![Singapore Parlimentary General Election Process and its Phases](/image/Tan_Kong-election.png)

### Problem Statement

In our project, we will be analysing the sentiments of the 2020 Singapore General Election across Reddit and Twitter:

1. Will there be differences in sentiments across social media platforms?

2. Will there be differences in sentiments across the election period?

3. Are there any differences in the sentiment of the words used to describe the incumbent party versus the opposition party?

4. How does the 2020 Singapore General Election compare to previous elections? Are there any differences in sentiments across the election process? The 2020 Singapore General Election will be compared to the 2015 Singapore General Election.

For our data science project, we activated the following packages, using the tidyverse approach.

```
RedditExtractoR
rtweet
tidyverse
knitr
tidytext
lubridate
ggrepel
car
ggpubr
jtools
huxtable
haven
broom
modelr
interactions
WRS2
rstatix
shiny
shinycssloaders
```

### Import

There are two sources for our data: (a) Reddit; and (b) Twitter.

#### Reddit

We used the `get_reddit` function from the RedditExtractoR package on multiple occasions to scrape data from the Singapore subreddit. Each time, the output from the get_reddit function was saved as a RData file.

These RData files were subsequently merged together, with duplicate rows removed. An output is saved as a RDS file.

#### Twitter

Twitter was scraped each day from 23 June to 12 July 2020, and the output was saved in a csv file. To cover as much tweets on elections as possible, search terms on the few common election hashtags, and all tweets from Singapore are used:

```
#ge2020 OR #sgvotes OR #sgelections OR #sgelection
```

The individual csv files were loaded and then merged together, with the outputs saved as RDS files.

### Tidy & Transform

Datasets are loaded from the RDS files.

#### Reddit

First, we extracted out only the columns we required. We renamed the `comm_date` column to `date` while converting it to date data type since it was incorrectly read as character data type.

The Reddit data contains threads on all manner of subjects. Using regular expressions, we extracted out rows that include comments on election-related information only:

```
election|^GE[[:blank:]]|GE2020|voting|polling|nomination|manifesto
```

All comments were converted to lowercase.

Next, we created variables called `period_2015` and `period_2020` that splits the election process into three distinct phases for the 2015 and 2020 General Elections, respectively.

```
reddits <- reddits %>% 
  mutate(period_2015 = case_when(
    between(date, as_date("2015-08-25"), as_date("2015-09-01")) ~ "first phase",
    between(date, as_date("2015-09-02"), as_date("2015-09-11")) ~ "second phase",
    between(date, as_date("2015-09-12"), as_date("2015-09-13")) ~ "third phase"), 
         period_2020 = case_when(
           between(date, as_date("2020-06-23"), as_date("2020-06-30")) ~ "first phase",
           between(date, as_date("2020-07-1"), as_date("2020-07-10")) ~ "second phase",
           between(date, as_date("2020-07-11"), as_date("2020-07-12")) ~ "third phase"))
```

Using regular expressions from eye-balling the comment data, assign comments containing words related to the incumbent party and opposition parties to their respective parties.

```
reddits <- reddits %>% 
  mutate(pol_party = case_when(
    str_detect(comment, paste0("lhl|lee hsien loong|goh chok tong|pap|ccs|tcj|",
                               "murali|lky|sun xueling|ib|incumbent|ruling party|",
                               "loong|kate spade|tpl|george yeo|lim hwee hwa|",
                               "jo teo|joteo|hsk|yacob|ivan|masagos|halimah|mr lee")) ~ "incumbent",
    str_detect(comment, paste0("nicole|jamus|wp|ltk|chiam|cst|pritam|kj|opp|nsp|",
                               "chee|people's voice|pv|csj|sylvia|jeanette|",
                               "lim tean|tambyah|sdp|chia|psp|workers party|aljunied")) ~ "opposition"))
```

Next, we unnest the comments into tokens (individual words), with each row consisting of a single token. Stop words, which are common words not helpful for analysis, are then removed. Rows the lacked a value for the `period_2015`, `period_2020`, and `pol_party` variables are removed

In the final step, we attached the afinn, nrc, and bing sentiment dictionaries to the data frame. Rows without any values from the three lexicons are dropped.

#### Twitter

The Twitter data consists of two data frames: (a) Singapore election tweets; and (b) Singapore tweets. Election-related posts are filtered from Singapore tweets. Both the Singapore election tweets and Singapore tweets are merged together and duplicates removed. Using regular expression, terms that are not related to Singapore elections are removed. After which, variables that are needed for analysis are selected.

Similar to what was done for the Reddit data, we created the period variable to split the election process to three phases. Since there is a limitation to what we can scrape from Twitter, only 2020 tweets are available.

```
tweets <- tweets %>% 
  mutate(date = as_date(created_at)) %>% # Convert datetime to date
  filter(date >= as_date("2020-06-23") & date <= as_date("2020-07-12")) %>% 
  mutate(period_2020 = case_when(
    between(date, as_date("2020-06-23"), as_date("2020-06-30")) ~ "first phase",
    between(date, as_date("2020-07-01"), as_date("2020-07-10")) ~ "second phase",
    between(date, as_date("2020-07-11"), as_date("2020-07-12")) ~ "third phase"))
```

Tweets are further categorised to incumbent and opposition parties using regular expressions. More expressions are added for the 10 opposition parties to balance the number of tweets related to each group, and to cover as much opposition tweets as possible. `screen_name` variable from the Twitter data is also used to group the tweets.

```
# Group posts into incumbent-related and opposition-related based on text contents
tweets <- tweets %>%
  mutate(pol_party = case_when(
    str_detect(text, regex(paste0("lhl|lee hsien loong|goh chok tong|pap|ccs|tcj|",
                                  "murali|lky|sun xueling|ib|incumbent|ruling party|",
                                  "loong|kate spade|tpl|george yeo|lim hwee hwa|",
                                  "jo teo|joteo|hsk|yacob|ivan|masagos|halimah|mr lee"),
                           ignore_case = TRUE)) ~ "incumbent",
    str_detect(text, regex(paste0("nicole|jamus|wp|ltk|chiam|cst|pritam|kj|opp|nsp|",
                                  "chee|people's voice|pv|csj|sylvia|jeanette|",
                                  "lim tean|tambyah|sdp|chia|psp|workers party|aljunied|",
                                  "sdp|singapore democratic party|psp|progress singapore party|",
                                  "wp|workers party|workers' party|ppp|people's power party|",
                                  "pv|peoples voice|people's voice|spp|singapore people's party|",
                                  "rp|reform party|rdu|red dot united|hammer|hypebeast|",
                                  "RedDotUnited|ProgressSgParty|thereformparty|workersparty|",
                                  "raeesah|cheng bok|tcb|jeyaretnam"),
                                  ignore_case = TRUE)) ~ "opposition"))

# Group posts into incumbent-related and opposition-related based on screen name
tweets <- tweets %>% 
  mutate(pol_party = if_else(
    screen_name %in% c("PAPSingapore", "VivianBala"), 
    "incumbent", if_else(
      screen_name %in% c("thereformparty", "wpsg", "ProgressSgParty", "KenJeyaretnam", "TanChengBock", "SPP_SG"),
      "opposition", pol_party)))
```

We unnest the texts from the individual tweets into tokens, just like Reddit comments. Stop words are removed thereafter. Common Twitter characters that are not useful in analysis (e.g. hashtags, mentions, hyperlinks) are excluded as well.

Rows that do not have values for `pol_party` are removed. `period` was checked to have values for every row. We generated sentiments using `afinn`, `nrc` and `bing` dictionaries to the data table.

#### Merging Reddit & Twitter

First, we removed the `status_id` and `created_at` columns from the `tweets_sentiments` data frame. We then renamed the `period` column to `period_2020` to align it with the reddit data frame. This is followed by creating a blank column called `period_2015` and re-arranging the colums to match the `reddit` data frame.

Next, the `platform` variable was created to indicate which platform (Reddit or Twitter) a given observation belongs to. This was done separately for each dataset.

Finally, both Reddit and Twitter data were merged together, and the final `df` data will be used for exploratory and analysis.

### Explore and Visualise

#### Chatter plots

Chatter plots were used to display the most frequently occurring words against their emotional valence.<sup>[2](#footnote2)</sup>

##### Reddit

Only 2020 data are selected for Reddit in order to be comparable with Twitter.

![Reddit Chatter Plot](/image/reddit-chatter-plot.png)

Interestingly, the top 50 words from Reddit are all positive.

##### Twitter

![Twitter Chatter Plot](/image/twitter-chatter-plot.png)

Twitter has top 50 words that have more balanced sentiments between positive and negative compared to Reddit.

Both have the most word used in the posts - “winning”. The context of this most used word is likely on the hope in winning the constituency the candidates that are contesting in. Moving later into the election process, the winning candidates would be happy, and the democracy in Singapore has made progress with the official appointment of the Leader of the Opposition.<sup>[3](#footnote3)</sup> Both words “happy” and “progress” are the other top few positive words in Twitter.

In contrast, the top few most negative words in Twitter are “bad”, “lost” and “disappointed”. These words are likely due to candidates that are hopeful to win but did not in the end. This may also be related to another GRC lost by the incumbent.<sup>[4](#footnote4)</sup>

#### Comparison between Three Sentiment Dictionaries

##### Reddit

![Comparison between Three Sentiment Dictionaries on Reddit](/image/three-sentiment-dictionaries-reddit.png)

##### Twitter

![Comparison between Three Sentiment Dictionaries on Twitter](/image/three-sentiment-dictionaries-twitter.png)

Comparing between Reddit and Twitter datasets, Twitter has generally more positive sentiments. This is possibly due to news snippets and links that are posted by news agencies. The contents are tend to be more positive regardless of which party are they on. Reddit has more texts per post, and therefore giving more weightage to negative words.

#### Sentiment plots between incumbent and opposition (AFINN lexicon)

##### Reddit

![Incumbent vs Opposition on Reddit (AFINN)](/image/incumbent-vs-opposition-reddit-afinn.png)

##### Twitter

![Incumbent vs Opposition on Twitter (AFINN)](/image/incumbent-vs-opposition-twitter-afinn.png)

Based on the AFINN sentiment plots, the trend for the opposition is generally similar between Reddit and Twitter with a dip to negative between 4 and 5 July. This is probably due to the police investigation on Workers’ Party candidate Raeesah Khan over alleged online comments on race and religion.<sup>[5](#footnote5)</sup> There is also a dip to negative for the incumbent party in Twitter, due to the withdrawal of PAP candidate Ivan Lim after allegations about his past conduct and behaviour.<sup>[6](#footnote6)</sup>

#### Sentiment plots between incumbent and opposition (NRC lexicon)

In the previous examples, we used the `afinn` lexicon for data visualisation. In this section, we will be using the `NRC` lexicon. The `NRC` lexicon conceptualizes sentiments not along a continuum, but as discrete categories such as sadness, disgust, fear, anger, etc. Due to the presence of multiple categories, we used a Shiny app to provide readers/users with the freedom to visualise each sentiment at their own pace.

The results appear roughly similar across all 10 sentiment categories, with comments made concerning the incumbent party experiencing a large spike near the end of the second phase and throughout the third phase. In contrast, there are two trends associated with the opposition parties. For the negative, sadness, disgust, fear, and anger sentiments, there was a smaller spike at the end of the second phase/start of the third phase relative to the much larger spikes of the positive, anticipation, joy, surprise, and trust sentiments across the same instance.

```
Run Shiny App locally to view the visualizations
```

### Model

#### Assumption Check

Before running a three-way between-groups ANOVA model, one has to ensure that the following assumptions have been met:

* Dependent variable is measured on a continuous scale
* Independent variables are measured on a categorical scale
* Independence of observations
* Absence of significant outliers
* Dependent variable approximately normal for each combination of the independent variables
* Homogeneity of variance for each combination of the independent variables

The first two assumptions have been met. The dependent variable `afinn_value` is measured on a continuous scale. The independent variables of `platform`, `period_2020`, and `pol_party` are categorical variables.

The third assumption, independence of observations, cannot be ascertained. It would not be possible for us to know whether a given Twitter account would have posted under a different username on Reddit or vice versa.

Significant outliers could be identified with the boxplots of `afinn_value` for each combination of independent variables. The results show that there are no outliers in our dataset. This is expected since `afinn_value` is between -5 and 5.

![Boxplot of IVs](/image/boxplot-of-IVs.png)

The `identify_outlier` function from the `rstatix` package could also be used to identify outliers by group. Similar to the boxplots, the results showed that there are no outliers in our dataset.

Next, we used Q-Q plots to identify whether the `afinn_value` variable is normally distributed for each combination of independent variables. As the data points do not fall along the the reference line, the assumption for normality has been violated.

![Q-Q Plots](/image/qq-plots.png)

Levene’s Test for Homogeneity of Variance was conducted on the dataset. The assumption for homogeneity was violated.

![Levene's Test](/image/levenes-test.png)

ANOVA models remain robust even if the assumption of homogeneity of variance is violated. Robustness is dependent on the sample sizes across groups.<sup>[7](#footnote7)</sup> In our case, the sample sizes across groups are not similar to one another. Hence, we have to use a non-parametric equivalent to the 3-way between-groups ANOVA.

#### ANOVA

Functions from the `WRS2` package will be used for the subsequent modelling process as our assumption of homogeneity has been violated. The package offers robust ANOVA for situations such as ours.

The `t3way` function from the `WRS2` package was used as a robust 3-Way Between-Groups ANOVA alternative.<sup>[8](#footnote8)</sup> The results showed that there was a significant three-way interaction between `platform`, `period_2020`, and `pol_party`, p = 0.0010.

There was a significant two-way interaction between `platform` and `period_2020`, p = 0.0270. There was a significant two-way interaction between `platform` and `pol_party`, p = 0.0010. There was no significant interaction between `period_2020` and `pol_party`.

There was a significant main effect of `platform` on `afinn_value`, p = 0.0001. There was a significant main effect of `period_2020` on `afinn_value`, p = 0.0001. There was a significant main effect of `pol_party` on `afinn_value`, p = 0.0010.

```
## Call:
## t3way(formula = afinn_value ~ as.factor(platform) * as.factor(period_2020) * 
##     as.factor(pol_party), data = df)
## 
##                                                                      value
## as.factor(platform)                                             299.455464
## as.factor(period_2020)                                           59.137461
## as.factor(pol_party)                                            140.686817
## as.factor(platform):as.factor(period_2020)                        7.287889
## as.factor(platform):as.factor(pol_party)                         49.477585
## as.factor(period_2020):as.factor(pol_party)                       3.967989
## as.factor(platform):as.factor(period_2020):as.factor(pol_party)  16.799191
##                                                                 p.value
## as.factor(platform)                                              0.0001
## as.factor(period_2020)                                           0.0001
## as.factor(pol_party)                                             0.0010
## as.factor(platform):as.factor(period_2020)                       0.0270
## as.factor(platform):as.factor(pol_party)                         0.0010
## as.factor(period_2020):as.factor(pol_party)                      0.1380
## as.factor(platform):as.factor(period_2020):as.factor(pol_party)  0.0010
```

#### Post-hoc tests

A significant three-way interaction could be followed up with the following post-hoc tests<sup>[9](#footnote9)</sup>:

* Two-way interaction at each level of the third variable
* Main effects of each variable
* Pairwise comparisons of each variable that has more than two levels (e.g. period_2020)

A three-way interaction suggests that one, or more, two-way interactions differ across the levels of a third variable.<sup>[10](#footnote10)</sup> In our case, it could mean that there are differences in the interaction of `period_2020` and `platform` at each level of `pol_party`. We could visualise the three-way interaction as follows. From eye-balling the plot, comments made on Twitter were generally more positive than on Reddit, with comments made specifically on the oppositional parties more positive relative to the incumbent party.

![Three-way Interactions](/image/three-way-interactions.png)

We then subset the data frame by the `pol_party` variable and conduct separate two-way ANOVAs, for each `pol_party`. There was a significant interaction between `period_2020` and `platform`, p = 0.005 for the Incumbent party. Similarly, there was a significant interaction between the same two variables for the Opposition party, p = 0.001. This suggests that at certain phases, sentiment levels differ across platforms. From our above visualisation, it can be observed that the second phase has the lowest sentiment across all four groups (Incumbent party and Reddit platform; Incumbent party and Twitter platform; Opposition party and Reddit platform; Opposition Part and Twitter Platform).

```
split_incumbent <- df %>%
  filter(!is.na(period_2020)) %>%
  filter(pol_party == "incumbent") %>%
  t2way(afinn_value ~ as.factor(period_2020) * as.factor(platform), data = .)
split_incumbent
```

```
## Call:
## t2way(formula = afinn_value ~ as.factor(period_2020) * as.factor(platform), 
##     data = .)
## 
##                                              value p.value
## as.factor(period_2020)                     35.6099   0.001
## as.factor(platform)                        84.7453   0.001
## as.factor(period_2020):as.factor(platform) 10.8771   0.005
```

```
split_opposition <- df %>%
  filter(!is.na(period_2020)) %>%
  filter(pol_party == "opposition") %>%
    t2way(afinn_value ~ as.factor(period_2020) * as.factor(platform), data = .)
split_opposition
```

```
## Call:
## t2way(formula = afinn_value ~ as.factor(period_2020) * as.factor(platform), 
##     data = .)
## 
##                                               value p.value
## as.factor(period_2020)                      31.7457   0.001
## as.factor(platform)                        215.0011   0.001
## as.factor(period_2020):as.factor(platform)  10.8959   0.005
```

Finally, we look at the main effects to determine the effect of individual variables on `afinn_value`. As the `period_2020` variables consists of three levels, pairwise comparisons were made at an α of 0.05/12 as we have 12 groups in all. The results showed that there is a significant difference between all three pairs. The line chart below shows that all 3 phases are marked by negative sentiments, with later phases being more negative than the earlier phases.

```
# t1way(afinn_value ~ platform, data = df)
# t1way(afinn_value ~ period_2020, data = df)
# t1way(afinn_value ~ pol_party, data = df)

lincon(afinn_value ~ period_2020, data = df, alpha = 0.004166667)
```

```
## Call:
## lincon(formula = afinn_value ~ period_2020, data = df, alpha = 0.004166667)
## 
##                               psihat ci.lower ci.upper p.value
## first phase vs. second phase 0.24288  0.14048  0.34528       0
## first phase vs. third phase  0.39335  0.27572  0.51098       0
## second phase vs. third phase 0.15047  0.05472  0.24621       0
```

![Sentiment across the Three Phases](/image/sentiment-across-three-phases.png)

#### Logistic Regression

To investigate `bing` sentiment data, logistic regression was used to take a look at the main effects and if there are interactions that lead to how positive or negative the sentiments are. Categorical variables are converted to binary or integers.

##### Main effects

The main effects are clearly significant with each independent variable (i.e. `platform`, `period` and `pol_party`) contributing to the model. Reddit (`twitter = 0`) is slightly more negative than Twitter (`twitter = 1`), while Opposition has more positive sentiments than Incumbent in either social media platforms. The sentiments generally became gradually negative across the election period.

![Logistic Regression Main Effects](/image/LR-main-effects.png)

![Probability of Positive Bing Sentiment](/image/prob-of-positive-bing.png)

##### Interaction effects

We explore the interactions between two dummy variables: `twitter` and `opposition`. From the output, the interaction effect between the two variables is significant (p = 0.005).

![Logistic Regression Interaction Effects](/image/LR-interaction-effects.png)

##### Model comparison

Main effects and interaction effects models are compared. It shows that Model 2 (Interaction Effects Model) is slightly better than Model 1 (Main Effects Model).

![Logistic Regression Model Comparison 1](/image/LR-model-comparison-1.png)

![Logistic Regression Model Comparison 2](/image/LR-model-comparison-2.png)

##### Probing interactions

The first interaction model is visualised, and simple slopes analysis is performed.

![Logistic Regression Interaction Model](/image/LR-interaction-model.png)

```
## JOHNSON-NEYMAN INTERVAL 
## 
## When twitter is OUTSIDE the interval [-0.65, -0.12], the slope of
## opposition is p < .05.
## 
## Note: The range of observed values of twitter is [0.00, 1.00]
## 
## SIMPLE SLOPES ANALYSIS 
## 
## Slope of opposition when twitter = 0.00 (0): 
## 
##   Est.   S.E.   z val.      p
## ------ ------ -------- ------
##   0.11   0.03     3.54   0.00
## 
## Slope of opposition when twitter = 1.00 (1): 
## 
##   Est.   S.E.   z val.      p
## ------ ------ -------- ------
##   0.43   0.03    15.47   0.00
```

The increase in both independent variables `opposition` and `twitter` (i.e. for opposition on Twitter) were associated with increased value of `bing`. `opposition` (`pol_party = "opposition"`) was predictive of higher levels of `bing` at high `twitter` (`platform = "twitter"`) than at low `twitter` (`platform = "reddit"`).

![Logistic Regression Interaction Plot](/image/LR-interaction-plot.png)

Other two-way interactions and three-way interaction between the three variables are also investigated.

##### Two-way interaction between twitter and period

![Logistic Regression Two-way Interactions across Platform and Period](/image/LR-two-way-interactions-platform-period.png)

```
## JOHNSON-NEYMAN INTERVAL 
## 
## When twitter is INSIDE the interval [-0.23, 0.46], the slope of period is p
## < .05.
## 
## Note: The range of observed values of twitter is [0.00, 1.00]
## 
## SIMPLE SLOPES ANALYSIS 
## 
## Slope of period when twitter = 0.00 (0): 
## 
##    Est.   S.E.   z val.      p
## ------- ------ -------- ------
##   -0.03   0.01    -2.40   0.02
## 
## Slope of period when twitter = 1.00 (1): 
## 
##    Est.   S.E.   z val.      p
## ------- ------ -------- ------
##   -0.02   0.03    -0.87   0.39
```

In contrast with the first two-way interaction model, the `period` variable has a negative interaction. Increases in both independent variables `period` and `twitter` were associated with decreased values of `bing`. The effects of the social media platforms on the relationship of `period` and `bing` was slightly greater at the earlier part of the election process, slightly more for Twitter.

##### Two-way interaction between opposition and period

![Logistic Regression Two-way Interactions across Party and Period](/image/LR-two-way-interactions-party-period.png)

```
## JOHNSON-NEYMAN INTERVAL 
## 
## When opposition is INSIDE the interval [0.04, 1.79], the slope of period is
## p < .05.
## 
## Note: The range of observed values of opposition is [0.00, 1.00]
## 
## SIMPLE SLOPES ANALYSIS 
## 
## Slope of period when opposition = 0.00 (0): 
## 
##    Est.   S.E.   z val.      p
## ------- ------ -------- ------
##   -0.02   0.01    -1.77   0.08
## 
## Slope of period when opposition = 1.00 (1): 
## 
##    Est.   S.E.   z val.      p
## ------- ------ -------- ------
##   -0.07   0.03    -2.33   0.02
```

The same was observed when `twitter` variable (between Reddit and Twitter) is replaced with `opposition` (between incumbent and opposition) the predictor `x` variable. The effects of the political parties on the relationship of `period` and `bing` was slightly greater at the earlier part of the election process.

##### Three-way interaction

![Logistic Regression Three-way Interaction](/image/LR-three-way-interaction.png)

```
## ¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦ While twitter (2nd moderator) = 0.00 (0) ¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦ 
## 
## JOHNSON-NEYMAN INTERVAL 
## 
## When opposition is OUTSIDE the interval [0.15, 0.83], the slope of period
## is p < .05.
## 
## Note: The range of observed values of opposition is [0.00, 1.00]
## 
## SIMPLE SLOPES ANALYSIS 
## 
## Slope of period when opposition = 0.00 (0): 
## 
##    Est.   S.E.   z val.      p
## ------- ------ -------- ------
##   -0.05   0.01    -3.42   0.00
## 
## Slope of period when opposition = 1.00 (1): 
## 
##   Est.   S.E.   z val.      p
## ------ ------ -------- ------
##   0.09   0.04     2.19   0.03
## 
## ¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦ While twitter (2nd moderator) = 1.00 (1) ¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦¦ 
## 
## JOHNSON-NEYMAN INTERVAL 
## 
## When opposition is OUTSIDE the interval [0.00, 0.49], the slope of period
## is p < .05.
## 
## Note: The range of observed values of opposition is [0.00, 1.00]
## 
## SIMPLE SLOPES ANALYSIS 
## 
## Slope of period when opposition = 0.00 (0): 
## 
##   Est.   S.E.   z val.      p
## ------ ------ -------- ------
##   0.07   0.03     1.98   0.05
## 
## Slope of period when opposition = 1.00 (1): 
## 
##    Est.   S.E.   z val.      p
## ------- ------ -------- ------
##   -0.18   0.04    -4.18   0.00
```

In this three-way interaction model, across the election period, the incumbent has decreased values of `bing` while the opposition has increased values of `bing` on Reddit. However, Twitter has an opposite outcome for either of the political parties, i.e. incumbent has increased values of `bing` and opposition has decreased values of `bing` across the election period. The difference of `bing` values between incumbent and opposition is higher on Twitter.

#### 2015 vs 2020 Elections

To gain a well-rounded understanding of Singapore Elections, we cannot rely on data from one election alone. We need data from multiple elections to compare and contrast against one another. We have managed to obtain some Reddit data from the 2015 Singapore General Election. The dataset was reshaped and the trends in sentiment for both the 2015 and 2020 Singapore General Elections were plotted below.

![2015 vs 2020 Elections](/image/2015-vs-2020-elections.png)

A One-way Between-Groups ANOVA was conducted to analyse the effect of `year` on overall sentiments across both elections. There was a significant difference in sentiments between the two elections, with the 2015 election more negative than the 2020 election.

![Sentiment across both Elections](/image/sentiment-across-years.png)

### Interpretation of the Results

From our data science project, we found the following:

1. Generally, the election-related comments made on Reddit are more negatively-toned than Twitter. The negativity worsened at the second phase.

2. Second phase of the election process is marked by increasingly negative sentiments compared to first and third phases. In terms of discrete categories of emotions, towards the end of the second phase and throughout the third phase is where discussions are more heated up, with many more instances of various sentiments showing up in the data. During this particular period, the opposition parties experienced an increase in the number of positively-toned sentiments (e.g. anticipation, joy, trust, etc.) relative to the negatively-toned sentiments (e.g. sadness, disgust, fear, etc.) of the same period.

3. Comments made concerning the incumbent party was more negatively-toned than the opposition parties.

4. On the whole, Reddit users were more negative about the 2015 Singapore General Election than the 2020 Election. This could be attributed to Minister Mentor Lee Kuan Yew’s passing in 2015. Singapore was in a state of mourning and this could have contributed to the negative sentiments on Reddit. Following the majority win by the incumbent party, our plot reflects a change in positive sentiments for the incumbent party at the third phase. Perhaps Singaporeans were relieved that the status quo was maintained when they were at the most vulnerable.

### Implications

Analyses of the Reddit and Twitter data corroborates our thesis. There has been increasing dissatisfaction with the ruling of the incumbent party by Singaporeans. The findings of our project has lend credence to this view, providing evidence that Singaporeans have taken to social media platforms such as Reddit and Twitter to voice their dissatisfaction with the incumbent party. Of note is the negative sentiments for the Opposition parties as well. This contributes some sort of ambivalence or noise into our results. A cursory review of social media platforms would attribute this to the “Internet Brigade (IB)” who are pro-incumbent party and would openly express negative sentiments on opposition parties. Further analysis in this area is needed to tease out the difference.

We have also learned that social media platforms differ in the extent of sentiments expressed. Twitter, as a platform, is more positively-toned compared to Reddit. One plausible suggestion for this outcome is the presence of news pertaining to the elections on twitter. News adopt a neutral tone, which could contribute to the lift in negative sentiments to a less-negative tone.

### Limitations & Future Directions

There are some limitations in our data science project. First, we do not possess sufficient data for the 2015 Singapore General Election. There is a limitation to scrapping historical data from Twitter and we do not have any Twitter data from the 2015 Singapore General Election. While we do have some Reddit data for the 2015 Singapore General Election, this is insufficient as we explicitly filtered for election-related comments. We would have missed out on keywords that were tangentially related to the 2015 Elections. Even then, the 2015 General Election was an atypical election. At that point, the passing of Minister Mentor Lee Kuan Yew could have influenced the results positively for the incumbent party.

Second, the regular expressions used to differentiate comments concerning the incumbent party and the opposition parties are not intelligent enough to differentiate comments that include both parties in a single sentence. As a result, mentions of both parties are assigned to the incumbent party due to how our code is structured. Further research into this area could add additional nuance to our results.

Future research could consider including the Eat-Drink-Man-Woman (EDMW) subforum from the HardwareZone (HWZ) forum. This subforum is another avenue for Singaporeans to express their political views during the election period. Hence, comparisons could then be made across three platforms (i.e. Reddit, Twitter and HWZ EDMW subforum).

### Contribution Statement

Combined:

* Election process timeline chart
* Regular expressions to use to extract election posts
* Cleaned up and simplified script chunks to tidyverse way

David’s contributions:

* Scraping and cleaning the Singapore subreddit data
* Problem Statement and Research Questions
* Researched and implemented Chatter plots to visualise word frequency data with emotional valence of terms used
* Shiny app
* ANOVA models
* Interpretation of the Results and Implications
* Limitations and Future Directions

Fabian’s contributions:

* Scraping and cleaning the Twitter data
* Explanation of the Election Process
* Scraping historical election results data to visualise percentage of votes and voters turnout
* Comparison across three sentiment dictionaries
* Incumbent vs Opposition AFINN Sentiment Plots
* Logistic Regression models
* Text formatting and language

### References

<a name="footnote1">1</a>: [Election Department Singapore](https://www.eld.gov.sg/)

<a name="footnote2">2</a>: [RIP wordclouds, long live CHATTERPLOTS](https://towardsdatascience.com/rip-wordclouds-long-live-chatterplots-e76a76896098)

<a name="footnote3">3</a>: [GE2020: WP chief Pritam Singh says he looks forward to serving as Leader of the Opposition](https://www.channelnewsasia.com/news/singapore/ge2020-wp-chief-pritam-singh-says-he-looks-forward-to-serving-as-12924456)

<a name="footnote4">4</a>: [PAP lost the youth vote in Sengkang GRC](https://mothership.sg/2020/07/pap-lost-sengkang-grc-why/)

<a name="footnote5">5</a>: [Police investigating WP candidate Raeesah Khan over alleged online comments on race, religion](https://www.channelnewsasia.com/news/singapore/raeesah-khan-workers-party-police-reports-race-ge2020-sengkang-12903248)

<a name="footnote6">6</a>: [Singapore GE 2020: PAP new face Ivan Lim withdraws from election following allegations about his past behaviour](https://www.straitstimes.com/singapore/singapore-ge-2020-pap-new-face-ivan-lim-withdraws-as-a-candidate)

<a name="footnote7">7</a>: [The Problem with Using Tests for Statistical Assumptions](https://www.theanalysisfactor.com/the-problem-with-tests-for-statistical-assumptions/)

<a name="footnote8">8</a>: [WRS2: A Collection of Robust Statistical Methods](https://cran.r-project.org/web/packages/WRS2/index.html)

<a name="footnote9">9</a>: [ANOVA in R](https://www.datanovia.com/en/lessons/anova-in-r/)

<a name="footnote10">10</a>: [FAQ How Can I Understand A Three-way Interaction In ANOVA?](https://stats.idre.ucla.edu/other/mult-pkg/faq/general/faqhow-can-i-understand-a-three-way-interaction-in-anova/)