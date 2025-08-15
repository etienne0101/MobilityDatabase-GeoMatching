---
title: "Script"
output: html_document
date: "2025-08-08"
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)

install.packages("dplyr")
install.packages("stringi")
install.packages("readr")
```

## R Markdown

```{r merge}
# Load required packages
library(stringi)
library(readr)
library(dplyr)

# Load data
feeds <- read_csv("data/feeds_v2.csv")
geonames <- read_csv2("data/geonames-all-cities-with-a-population-1000.csv", col_names = TRUE, show_col_types = FALSE)

# Function to normalize names (lowercase, no accents, trim)
normalize_text <- function(x) {
  x <- tolower(stri_trans_general(x, "Latin-ASCII"))
  x <- trimws(x)
  return(x)
}

# Add normalization columns
feeds <- feeds %>%
  mutate(
    city_norm = normalize_text(location.municipality),
    country_code_norm = toupper(trimws(location.country_code))
  )

geonames <- geonames %>%
  mutate(
    city_norm = normalize_text(Name),
    country_code_norm = toupper(trimws(`Country Code`))
  )

# Merge the two datasets
merged <- geonames %>%
  left_join(feeds, by = c("city_norm", "country_code_norm"))

# Save the result
write_csv(merged, "geonames_merged_with_feeds.csv")


# Relevance calculation
# Add a temporary ID
feeds <- feeds %>%
  mutate(feed_id = row_number())

# List IDs that matched
matched_ids <- feeds %>%
  inner_join(geonames, by = c("city_norm", "country_code_norm")) %>%
  distinct(feed_id)

# Unique number of feeds that found a match
matched_count <- nrow(matched_ids)

# Correct calculation of non-match
total_feeds <- nrow(feeds)
unmatched_count <- total_feeds - matched_count

cat("Total rows in feeds              :", total_feeds, "\n")
cat("Rows with UNIQUE match            :", matched_count, "\n")
cat("Rows WITHOUT match                :", unmatched_count, "\n")
cat("Match rate (%)                    :", round(matched_count / total_feeds * 100, 2), "%\n")

# Actual unmatched rows
unmatched <- feeds %>%
  anti_join(matched_ids, by = "feed_id")

# (optional) Save
write_csv(unmatched, "feeds_non_matches.csv")

# Step 1: Create normalized join keys in feeds and geonames
feeds_keys <- feeds %>%
  mutate(city_norm = normalize_text(location.municipality),
         country_code_norm = toupper(trimws(location.country_code))) %>%
  distinct(city_norm, country_code_norm)

# Step 2: Extract unique cities in geonames that match
matched_cities <- geonames %>%
  mutate(city_norm = normalize_text(Name),
         country_code_norm = toupper(trimws(`Country Code`))) %>%
  semi_join(feeds_keys, by = c("city_norm", "country_code_norm")) %>%
  distinct(city_norm, country_code_norm, Population)

# Step 3: Calculate total covered population
total_covered_population <- sum(matched_cities$Population, na.rm = TRUE)

# Display
cat("Number of cities with feed        :", nrow(matched_cities), "\n")
cat("Total covered population          :", total_covered_population, "\n")

```
