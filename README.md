# 2020 Singapore General Election Sentiments across Two Social Media Platforms

## David Tan & Fabian Kong
## 02 August 2020

## Introduction

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