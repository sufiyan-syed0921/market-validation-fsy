## Overview

## Data Processing

### Clean Variable Names
Rename question 2 and 3 columns so that they contain the feature/yoke in the suffix 
of each column name

The raw output of the survey data required some processing to be analyized. For example for questions Q2 and Q3, the columns created that represent the ranking results are not labeled to 

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

```R

```

#### Check Filtering 

```R


```


## Analysis

## Setup 

### Libraries
