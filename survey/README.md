## Overview

## Data Processing

### Clean Variable Names
Rename question 2 and 3 columns so that they contain the feature/yoke in the suffix 
of each column name

The raw output of the survey data required some processing to be analyized. For example for questions Q2 and Q3, the raw columns that represent the ranking results are not labeled to reflect which features they represent. Instead of having the column name be for example "Q2_1" which represents the first ranking option of the question with rows that represent the reported ranking from respondents, I renamed the column name to include the feature name for easier analysis.

The Qualtrics output of the survey results reports the question name and response text for each column of the data as the first row. For example the value for Q2_1 is "When purchasing a flight simulator yoke, rank the following features and capabilities from most important (1) to least important (12). Drag and drop to rank the yokes. - Xbox Compatibility". The code below extracts the feature text that is after the "-" in the first row for each column and pastes it as a suffix for the column name. This procedure is performed for both Q2 and Q3.  

- Think about creating columns_to_rename programatically

```R 
# Question 2 -------------------------------------------------------------------
# : When purchasing a flight simulator yoke, rank the following features and 
# : capabilities from most important (1)  to least important (12).
#  -----------------------------------------------------------------------------

# Create temporary variable
tmp <- survey_rawdata

# Specify the columns to rename
columns_to_rename <- c("Q2_1", "Q2_2", "Q2_3", "Q2_4", "Q2_5", "Q2_6", "Q2_7", "Q2_8", "Q2_9", "Q2_10", "Q2_11", "Q2_12")

# Extract parts after "-" from the first row for the specified columns
suffixes <- sapply(tmp[1, columns_to_rename], function(x) sub(".*-", "", x))

# Get current column names
current_names <- colnames(tmp)

# Create new names for the specified columns
new_names <- paste(columns_to_rename, suffixes, sep = "_")

# Rename the specified columns
colnames(tmp)[colnames(tmp) %in% columns_to_rename] <- new_names

# clean names
tmp2 <- tmp %>% clean_names() 

# Remove temporary variables
rm(tmp, columns_to_rename, suffixes, current_names, new_names) 

```

```R
# Question 3 -------------------------------------------------------------------
# : Imagine you are looking to purchase a flight simulator yoke and are 
# : considering Yoke A, Yoke B, and Yoke C. The features of the three yokes are 
# : listed in the tables below. Please rank #the following yokes from most 
# : desirable (1) to least desirable (3), assuming that each yoke costs $400. 
# : Drag and drop to rank the yokes.
#  -----------------------------------------------------------------------------

# create temporary variable
tmp <- tmp2

# Specify the columns to rename
columns_to_rename <- c("q3_1", "q3_2", "q3_3")

# Extract parts after "-" from the first row for the specified columns
suffixes <- sapply(tmp[1, columns_to_rename], function(x) sub(".*-", "", x))

# Get current column names
current_names <- colnames(tmp)

# Create new names for the specified columns
new_names <- paste(columns_to_rename, suffixes, sep = "_")

# Rename the specified columns
colnames(tmp)[colnames(tmp) %in% columns_to_rename] <- new_names

# create temporary variables
tmp3 <- tmp

# remove temporary variables
rm(tmp, tmp2, columns_to_rename, suffixes, current_names, new_names) # Remove temporary variables

```
### Apply Survey Filters 

In order to create our analysis sample, we decided to filter our data along the folloing points: 

We intended to remove responses from:
 1. Survey Preview Responses - remove all response relating to testing of the survey.
 2. Respondents who do not own a flight simulation yoke - 
 3. Respondents who only have HOTAS style controllers - Some respondents took the survey while only owning a HOTAS style yoke which was not of interest and is not within the scope of competion we were interested in.
 4. Remove respondents who quit survey after question 1
 5. Remove respondents with inconsistent location responses (Falsely claimed to live in the SF Bay Area)

```R
# think about adding analysis sample column to data to then use as a filter and compare pre-post processed. 
```

#### Check Filtering 

```R


```


## Analysis

## Setup 

### Libraries
