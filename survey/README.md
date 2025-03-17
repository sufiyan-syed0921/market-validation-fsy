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

**1. Survey Preview Responses**
<br>
Remove all preview responses relating to testing of the survey. 

**2. Respondents who do not own a flight simulation yoke** 
<br>
Some responses indicated that they did not own a yoke controller at all and these responses were removed.
 
**3. Respondents who only have HOTAS style controllers**
<br>
Some responses indicated they owned controlles in a HOTAS (Joystick) format. Opinions of those who soley own these type of controllers 
were not of interest and outside the scope of this survey.
 
**4. Remove respondents who quit survey after question 1** 
<br> 
Some respondents quit the survey after responding to question 1 and these responses were removed. 

**5. Remove respondents with inconsistent location responses (Falsely claimed to live in the SF Bay Area)**
<br> 
 We included a question (Q12) to see if any respondents lived in the SF bay area to see if they would be open to participating in a in- 
 person study. We noticed that there was a suspiciously high number of responses who claimed they were located in the bay area given this 
 survey was diseminated on a reddit forum with global reach. To address this concern of fraudulent responses, we used the location 
 coordinates from the survey metadata to implement a location filter to remove responses who "lied" about their location. See the code 
 below for details on the method of how this was done. 
 
```R
# Prepare for filtering based on location (filter 5)---------------------------------------
# We remove all responses who claimed to live in the SF bay area but coordinates 
# reported by Qualtrics were outside a 100 mile radius of San Ramon, CA.  

## Function to calculate haversine distance
haversine_distance <- function(lat1, lon1, lat2, lon2) {
  R <- 3959  # Earth radius in miles
  dLat <- (lat2 - lat1) * pi / 180
  dLon <- (lon2 - lon1) * pi / 180
  lat1 <- lat1 * pi / 180
  lat2 <- lat2 * pi / 180
  a <- sin(dLat/2) * sin(dLat/2) + sin(dLon/2) * sin(dLon/2) * cos(lat1) * cos(lat2)
  c <- 2 * atan2(sqrt(a), sqrt(1 - a))
  d <- R * c
  return(d)
}

## San Ramon, CA coordinates
san_ramon_lat <- 37.7799
san_ramon_lon <- -121.978

```
<br>

In order to identify which respondents that took the survey only owned HOTAS style yokes, I needed to manually inspect the unique text entry responses for the Q1 "Other" response option. After inspecting these responses and identifying which controllers were HOTAS style, I included the names of the controllers in an object called "exc_filter" to use later as a filter.   

```R

# Prepare to filter to include only owners of Yoke style controllers -----------

# Find unique list of text entry HOTAS style controller names that respondents provided
unique(survey_rawdata$Q1_4_TEXT) # Run and filter manually to populate "sticks" object

# Gather responses with flight sticks
sticks <- c("Thrustmaster Warthog", 
           "Thrustmaster TCA Sidestick", 
           "Thrustmaster TCA Airbus", 
           "Thrustmaster Hotas Warthog", 
           "Sidewinder Force Feedback 2", 
           "Logitech x52 hotas", 
           "Logitech extreme 3d pro (I’m cheap)", 
           "Have HOTAS")

# Assign names to filter object
exc_filter <- paste(sticks, collapse = "|")

```
<br> 
My approach to filtering the data was to create dummy variables for each of the 5 filters and a binary analysis sample variable (ansamp) that defines which rows are part of the final sample. 

```R

# Create dummy variables for filters -------------------------------------------

tmp4 <- tmp3 %>%
  # Remove header rows
  slice(-(1:2)) %>%  
  # Create dummies
  mutate(
    ## Track pre-filtered rows
    initial_sample = 1,  
    ## If survey previews
    survey_preview = ifelse(status == "Survey Preview", 1, 0),  
    ## If no yoke ownership
    no_yoke = ifelse(grepl("I do not own a flight simulation yoke", q1, ignore.case = TRUE), 1, 0),  
    ## If only own HOTAS yokes
    hotas = ifelse(q1 == "Other" & grepl(exc_filter, q1_4_text, ignore.case = TRUE), 1, 0)  
  ) %>%
  rowwise() %>%
  mutate(
    # If quit after Q1
    quit_aq1 = ifelse(all(is.na(c_across(q2_1_xbox_compatibility:q15))), 1, 0)  
  ) %>%
  ungroup() %>%
  mutate(
    location_latitude2 = as.numeric(location_latitude), # make coordinates numeric
    location_longitude2 = as.numeric(location_longitude),
    # Calculate distance from San Ramon
    distance_to_sr = haversine_distance(location_latitude2, location_longitude2, 
                                        san_ramon_lat, san_ramon_lon),
    # If within 100 miles of San Ramon
    radius100 = ifelse(distance_to_sr <= 100, TRUE, FALSE),  
    # Claimed Bay Area but outside 100 miles
    inbay_ovr100 = ifelse(radius100 == FALSE & q12 == "Yes", TRUE, FALSE)  
  )

```
<br>

Here I cretate the ansamp variable tha defines the analysis sample and creating the filtered processed data file. 
```R
# Filter data ------------------------------------------------------------------

# Apply the filters to get the final analysis sample
tmp5 <- tmp4 %>%
  mutate(ansamp = ifelse( # change this mutate and have this define the ansamp == 1
    survey_preview == 0 & # (1) Remove survey previews 
    no_yoke == 0 & # (2) Remove respondents who don't have a yoke
    hotas == 0 &  # (3) Remove respondents who only have HOTAS controllers
    quit_aq1 == 0 & # (4) Remove respondents who quit survey after Q1
    inbay_ovr100 == FALSE, 1, 0)) # (5) Remove respondents with inconsistent location responses

# Create final data for analysis  
data <- tmp5 %>% 
  select(-survey_preview, -no_yoke, -hotas, -quit_aq1) %>% # remove filter variables
  filter(ansamp == 1) # filter to include analysis sample only

```

#### Check Filtering 

I run these checks to see how many rows were removed and inspect the removed rows. 
```{r}
# Check Number of Rows Removed ------------------------------------------------------------------

## OLD 
paste0("Old N: ", nrow(tmp4))

## NEW
paste0("New N: ", nrow(data))

## Spot Check Removed Rows 
removed_rows <- tmp5 %>% filter(ansamp == 0)
```
<br> 
This table helps understand why rows were filtered out by displaying how many rows were filtered for each combination of the 5 filter variables. 

```R
# Check Filters ----------------------------------------------------------------------------------

## Table all filters
tmp5 %>% select(survey_preview,
                no_yoke,
                hotas,
                quit_aq1,
                inbay_ovr100, 
                ansamp) %>% 
  group_by_all() %>% 
  summarise(N = n(), .groups = "drop") %>% 
  flextable() %>% 
  autofit()

```
<br>
These checks inspect the post-processed data to make sure the filters were applied correctly. 

```R
# (1) Survey previews and (2) Yoke Ownership
data %>% select(status, q1) %>% 
  group_by_all() %>% 
  summarise(N = n(), .groups = "drop") %>% 
  flextable() %>% # create table
  autofit()

```

```R
# (3) No hotas ownership 
table(data$q1_4_text)

```

```R
# (4) Respondents Quitting After Q1
data %>% filter(if_all(q2_1_xbox_compatibility:q15, is.na))

# (5) Inconsistent location responses
locfilter_check <- data %>% 
  select(q12, ends_with("e2"), ends_with("100"), distance_to_sr, ansamp) %>% 
  filter(q12 == "Yes")

```

## Analysis



## Setup 

### Libraries
