Project proposal
================
The Scatterplots

``` r
library(tidyverse)
```

    ## Warning in system("timedatectl", intern = TRUE): running command 'timedatectl'
    ## had status 1

## Dataset

A brief description of your dataset including its provenance,
dimensions, etc. as well as the reason why you chose this dataset.

Make sure to load the data and use inline code for some of this
information.

``` r
parks <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2021/2021-06-22/parks.csv')
```

    ## Rows: 713 Columns: 28

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (5): city, park_pct_city_data, pct_near_park_data, spend_per_resident_d...
    ## dbl (23): year, rank, med_park_size_data, med_park_size_points, park_pct_cit...

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

28 \#\# Questions

The two questions you want to answer.

## Analysis plan

A plan for answering each of the questions including the variables
involved, variables to be created (if any), external data to be merged
in (if any).
