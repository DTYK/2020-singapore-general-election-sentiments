## 2020 Singapore General Election Sentiments across Two Social Media Platforms

### David Tan & Fabian Kong
### 02 August 2020

### Introduction

The Parliament in Singapore may be dissolved before the expiry of its five-year term by the President on the advice of the Prime Minister. The general election must be held within three months of the Parliament’s dissolution. There are two types of constituency or electoral division: single member constituency (SMC) and group representation constituency (GRC). SMC, as the name suggests, has only one member of parliament (MP), while GRC has three to six MPs. The GRC scheme was introduced in the 1988 general election to ensure that minority groups are represented in Parliament 1.

Below are the percentage of votes between incumbent and opposition, and of voters turnout for parliamentary general elections between 1988 (when GRC scheme is introduced) and 2020. Data for 2015 and earlier are scraped from Data.gov.sg using CKAN API while data for 2020 is scraped from Wikipedia (which is sourced from Elections Department Singapore and Channel News Asia) using rvest package.

```
source("Tan_Kong-ge_results.R")
ge_plot
```

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

### Problem Statement

In our project, we will be analysing the sentiments of the 2020 Singapore General Election across Reddit and Twitter:

1. Will there be differences in sentiments across social media platforms?

2. Will there be differences in sentiments across the election period?

3. Are there any differences in the sentiment of the words used to describe the incumbent party versus the opposition party?

4. How does the 2020 Singapore General Election compare to previous elections? Are there any differences in sentiments across the election process? The 2020 Singapore General Election will be compared to the 2015 Singapore General Election.

For our data science project, we activated the following packages, using the tidyverse approach.

```
# If a package is installed, it will be loaded. If not, missing package(s) will
# be installed from CRAN and then loaded.

# Specify packages
packages <- c("RedditExtractoR", "rtweet", "tidyverse", "knitr", "tidytext", "lubridate", "ggrepel", "car", "ggpubr", "jtools", "huxtable", "haven", "broom", "modelr", "interactions", "WRS2", "rstatix", "shiny", "shinycssloaders")

# Load or install then load all
package.check <- lapply(
  packages,
  FUN = function(x) {
    if(!require(x, character.only = TRUE)) {
      install.packages(x, dependencies = TRUE)
      library(x, character.only = TRUE)
    }
  }
)
```

### Import

There are two sources for our data: (a) Reddit; and (b) Twitter.

#### Reddit

We used the `get_reddit` function from the RedditExtractoR package on multiple occasions to scrape data from the Singapore subreddit. Each time, the output from the get_reddit function was saved as a RData file.

```
#sg_reddit <- get_reddit(subreddit = "singapore", page_threshold = 500, wait_time = 5)
```

These RData files were subsequently merged together, with duplicate rows removed. An output is saved as a RDS file.

```
#load("data/sg_reddit.RData")
#load("data/sg_reddit2.RData")
#load("data/sg_reddit3.RData")
#load("data/sg_reddit5.RData")
#load("data/reddit.RData")

#reddits <- bind_rows(sg3, sg4, sg5, sg6, df) %>% 
#    distinct()

#saveRDS(reddits, "data/reddits.rds")
```

#### Twitter

Twitter was scraped each day from 23 June to 12 July 2020 using the following lines of code, and the output was saved in a csv file. To cover as much tweets on elections as possible, search terms on the few common election hashtags, and all tweets from Singapore are used.

```
# Load Twitter API credentials
#source("script/twitter_api.R")

# Daily between 2020-06-23 to 2020-07-12
#today_dat = Sys.Date()
#yday_dat = Sys.Date() - 1

#df_sgelections <- search_tweets(q = "#ge2020 OR #sgvotes OR #sgelections OR #sgelection",
#                                n = 500000,
#                                lang = "en",
#                                since = yday_dat,
#                                until = today_dat,
#                                include_rts = F,
#                                retryonratelimit = T)

#save_as_csv(df_sgelections, paste("data/sgelections_", yday_dat, ".csv", sep = ""),
#            prepend_ids = TRUE, na = "", fileEncoding = "UTF-8")

#df_sgtweets <- search_tweets(n = 500000,
#                             lang = "en",
#                             since = yday_dat,
#                             until = today_dat,
#                             include_rts = F,
#                             geocode = lookup_coords("singapore"),
#                             retryonratelimit = T)
```

The individual csv files were loaded and then merged together, with the outputs saved as RDS files.

```
#sgelections0623 <- read_csv("data/sgelections_2020-06-23.csv")
#sgelections0624 <- read_csv("data/sgelections_2020-06-24.csv")
#sgelections0625 <- read_csv("data/sgelections_2020-06-25.csv")
#sgelections0626 <- read_csv("data/sgelections_2020-06-26.csv")
#sgelections0627 <- read_csv("data/sgelections_2020-06-27.csv")
#sgelections0628 <- read_csv("data/sgelections_2020-06-28.csv")
#sgelections0629 <- read_csv("data/sgelections_2020-06-29.csv")
#sgelections0630 <- read_csv("data/sgelections_2020-06-30.csv")
#sgelections0701 <- read_csv("data/sgelections_2020-07-01.csv")
#sgelections0702 <- read_csv("data/sgelections_2020-07-02.csv")
#sgelections0703 <- read_csv("data/sgelections_2020-07-03.csv")
#sgelections0704 <- read_csv("data/sgelections_2020-07-04.csv")
#sgelections0705 <- read_csv("data/sgelections_2020-07-05.csv")
#sgelections0706 <- read_csv("data/sgelections_2020-07-06.csv")
#sgelections0707 <- read_csv("data/sgelections_2020-07-07.csv")
#sgelections0708 <- read_csv("data/sgelections_2020-07-08.csv")
#sgelections0709 <- read_csv("data/sgelections_2020-07-09.csv")
#sgelections0710 <- read_csv("data/sgelections_2020-07-10.csv")
#sgelections0711 <- read_csv("data/sgelections_2020-07-11.csv")
#sgelections0712 <- read_csv("data/sgelections_2020-07-12.csv")

#sgelections <- bind_rows(
#    sgelections0623, sgelections0624, sgelections0625, sgelections0626, sgelections0627,
#    sgelections0628, sgelections0629, sgelections0630, sgelections0701, sgelections0702,
#    sgelections0703, sgelections0704, sgelections0705, sgelections0706, sgelections0707,
#    sgelections0708, sgelections0709, sgelections0710, sgelections0711, sgelections0712)

#saveRDS(sgelections, "data/sgelections.rds")

# Merge Singapore datasets
#sgtweets0623 <- read_csv("data/sgtweets_2020-06-23.csv")
#sgtweets0624 <- read_csv("data/sgtweets_2020-06-24.csv")
#sgtweets0625 <- read_csv("data/sgtweets_2020-06-25.csv")
#sgtweets0626 <- read_csv("data/sgtweets_2020-06-26.csv")
#sgtweets0627 <- read_csv("data/sgtweets_2020-06-27.csv")
#sgtweets0628 <- read_csv("data/sgtweets_2020-06-28.csv")
#sgtweets0629 <- read_csv("data/sgtweets_2020-06-29.csv")
#sgtweets0630 <- read_csv("data/sgtweets_2020-06-30.csv")
#sgtweets0701 <- read_csv("data/sgtweets_2020-07-01.csv")
#sgtweets0702 <- read_csv("data/sgtweets_2020-07-02.csv")
#sgtweets0703 <- read_csv("data/sgtweets_2020-07-03.csv")
#sgtweets0704 <- read_csv("data/sgtweets_2020-07-04.csv")
#sgtweets0705 <- read_csv("data/sgtweets_2020-07-05.csv")
#sgtweets0706 <- read_csv("data/sgtweets_2020-07-06.csv")
#sgtweets0707 <- read_csv("data/sgtweets_2020-07-07.csv")
#sgtweets0708 <- read_csv("data/sgtweets_2020-07-08.csv")
#sgtweets0709 <- read_csv("data/sgtweets_2020-07-09.csv")
#sgtweets0710 <- read_csv("data/sgtweets_2020-07-10.csv")
#sgtweets0711 <- read_csv("data/sgtweets_2020-07-11.csv")
#sgtweets0712 <- read_csv("data/sgtweets_2020-07-12.csv")

#sgtweets <- bind_rows(
#    sgtweets0623, sgtweets0624, sgtweets0625, sgtweets0626, sgtweets0627,
#    sgtweets0628, sgtweets0629, sgtweets0630, sgtweets0701, sgtweets0702,
#    sgtweets0703, sgtweets0704, sgtweets0705, sgtweets0706, sgtweets0707,
#    sgtweets0708, sgtweets0709, sgtweets0710, sgtweets0711, sgtweets0712)

#saveRDS(sgtweets, "data/sgtweets.rds")
```

### Tidy & Transform

Datasets are loaded from RDS files.

```
# Reddit
reddits <- readRDS("Tan_Kong-reddits.rds")

# Twitter
sgelections <- readRDS("Tan_Kong-sgelections.rds")
sgtweets <- readRDS("Tan_Kong-sgtweets.rds")
```

#### Reddit

First, we extracted out only the columns we required. We renamed the `comm_date` column to `date` while converting it to date data type since it was incorrectly read as character data type.

```
reddits <- reddits %>% 
  mutate(date = dmy(comm_date)) %>% 
  select(date, comment)
```

The following shows the Reddit raw dataset:

```
glimpse(reddits)
```

```
## Rows: 169,031
## Columns: 2
## $ date    <date> 2020-06-21, 2020-06-21, 2020-06-21, 2020-06-21, 2020-06-21...
## $ comment <chr> "Govt gonna get more than their $600 per person back from t...
```

The Reddit data contains threads on all manner of subjects. Using regular expressions, we extracted out rows that include comments on election-related information only. All comments were converted to lowercase.

```
reddits <- reddits %>% 
  filter_at(vars(comment), 
            any_vars(str_detect(comment, "election|^GE[[:blank:]]|GE2020|voting|polling|nomination|manifesto"))) %>% 
  mutate(comment = str_to_lower(comment))
```

Next, we created variables called period_2015 and period_2020 that splits the election process into three distinct phases for the 2015 and 2020 General Elections, respectively.

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

```
reddit_tc <- reddits %>% 
  unnest_tokens(output = word, input = comment) %>% 
  anti_join(stop_words) %>%
  filter(!is.na(period_2015) | !is.na(period_2020)) %>% 
  filter(!is.na(pol_party))
```

In the final step, we attached the afinn, nrc, and bing sentiment dictionaries to the data frame. Rows without any values from the three lexicons are dropped.

```
reddit_sentiments <- reddit_tc %>% 
  left_join(get_sentiments("afinn"), by = "word") %>% 
  left_join(get_sentiments("nrc"), by = "word") %>% 
  left_join(get_sentiments("bing"), by = "word") %>% 
  rename(afinn_value = value, nrc_sentiment = sentiment.x, bing_sentiment = sentiment.y) %>% 
  drop_na(afinn_value:bing_sentiment) %>% 
  arrange(date)
```

#### Twitter

The Twitter data consists of two data frames: (a) Singapore election tweets; and (b) Singapore tweets. Election-related posts are filtered from Singapore tweets. Both the Singapore election tweets and Singapore tweets are merged together and duplicates removed. Using regular expression, terms that are not related to Singapore elections are removed. After which, variables that are needed for analysis are selected.

```
# Filter election-related posts from Singapore tweets
sgtweets <- bind_rows(
  sgtweets %>% filter(str_detect(text, "election|^GE[[:blank:]]|GE2020|voting|polling|nomination|manifesto")),
  sgtweets %>% filter(str_detect(text, "ge2020|sgvotes|sgelections|sgelection")),
  sgtweets %>% filter(str_detect(hashtags, "election|^GE[[:blank:]]|GE2020|voting|polling|nomination|manifesto")),
  sgtweets %>% filter(str_detect(hashtags, "ge2020|sgvotes|sgelections|sgelection")))

# Merge filtered Singapore and election-tagged tweets
tweets <- bind_rows(sgtweets, sgelections) %>% 
  distinct()

# Exclude tweets not related to Singapore and elections
tweets <- tweets %>% 
  filter(!str_detect(text, regex(paste0("malaysia|mahathir|muhyiddin|",
                                        "trump|biden|pence|sanders|republican|",
                                        "us election|clinton|congress|",
                                        "china|poland|french|tokyo|ardern|",
                                        "modi|delhi|boris|brexit|ireland|",
                                        "kang daniel|kangdaniel|bts_twt|",
                                        "btsvotingorg|henggarae|monstax|exo"),
                                 ignore_case = TRUE))) %>% 
  select(status_id, created_at, screen_name, text, name, location)
```