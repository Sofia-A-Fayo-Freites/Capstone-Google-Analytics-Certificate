Google Analytics Course Capstone
================
Sofía A. Fayó Freites
2024-11-13

## ***Analysis of consumer trends in Spain through 2024 using the Google Trends Dataset***

This is a document generated using R Markdown, and R code for data
processing and exploration.

Required installed packages:

``` r
library(tidyverse)
library(readr)
library(ggplot2)
library(jsonlite)
```

### Data processing: First tables

The first part of the analysis delves into broad consumer trends across
the nation, utilizing data extracted from the initial SQL BigQuery
query.

1.  Read the CSV file with readr.

``` r
dataset <- read_csv("~/Data Science/Google/Google Data Analytics Certificate/Module 8/capstone/Google Trends_First Table SQL Query.csv")
```

2.  Convert the column “month” to string with the name of the month and
    then back again to date.

``` r
trends_df <- as.data.frame(dataset)
trends_df$month <- month.name[trends_df$month]
trends_df <- trends_df %>%
  mutate(month = factor(month, levels = month.name))
glimpse(trends_df)
View(trends_df)
```

3.  Filter by the most searched term in Google by month.

``` r
top_searched_by_month <- trends_df %>%
  group_by(month) %>%
  filter(total_score == max(total_score)) %>%
  arrange(month)
glimpse(top_searched_by_month)
View(top_searched_by_month)
```

4.  Filter the top 5 most searched queries in Google.

``` r
top_5_queries <- trends_df %>%
  group_by(term) %>%
  summarize(total_score = sum(total_score)) %>%
  arrange(desc(total_score)) %>%
  slice_head(n = 5)
glimpse(top_5_queries)
View(top_5_queries)
```

5.  Create a dataframe with the top 5 Google queries per month in 2024.

``` r
filtered_df <- trends_df %>%
  filter(term %in% top_5_queries$term)

top_5_queries_month <-  filtered_df %>%
  group_by(month, term) %>%
  summarize(total_score = sum(total_score))
glimpse(top_5_queries_month)
View(top_5_queries_month)
```

### Data processing: Second table

To gain a deeper understanding of consumer behavior, the second part of
the analysis examines data on a monthly and weekly basis. The necessary
data for this analysis was sourced from the second SQL BigQuery query.

1.  Read the CSV file with readr.

``` r
dataset_2 <- read_csv("~/Data Science/Google/Google Data Analytics Certificate/Module 8/capstone/Google Trends_Second Table SQL Query.csv")
```

2.  Convert the dataset to dataframe, rename columns and months.

``` r
trends_df_2 <- as.data.frame(dataset_2)
trends_df_2 <- rename(trends_df_2, term=f0_)
trends_df_2$month <- month.name[trends_df_2$month]
trends_df_2 <- trends_df_2 %>%
  mutate(month = factor(month, levels = month.name))
glimpse(trends_df_2)
View(trends_df_2)
```

3.  Create the filter for the top 3 searches per month.

``` r
top_3_queries_in_each_month <- trends_df_2 %>%
  group_by(month, term) %>%
  summarize(total_score = sum(total_score)) %>%
  arrange(month, desc(total_score)) %>%
  group_by(month) %>%
  slice_head(n = 3)
View(top_3_queries_in_each_month)
```

4.  Filter the original data frame to include only the top 3 terms for
    each month.

``` r
filtered_df_2 <- trends_df_2 %>%
  inner_join(top_3_queries_in_each_month, by = c("month", "term"))
View(filtered_df_2)
```

5.  Group by week, month, and term, and calculate total score per week
    for each term.

``` r
top_3_queries_in_each_month <- filtered_df_2 %>%
  group_by(week, month, term) %>%
  summarize(total_score = sum(total_score.x))
View(top_3_queries_in_each_month)
```

### Export the tables in JSON format

``` r
write_json(top_searched_by_month, "top_searched_by_month.json")
write_json(top_5_queries, "top_5_queries.json")
write_json(top_5_queries_month, "top_5_queries_month.json")
write_json(top_3_queries_in_each_month, "top_3_queries_in_each_month.json")
```
