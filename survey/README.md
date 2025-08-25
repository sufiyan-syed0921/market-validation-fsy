# Survey Data Processing & Analysis

The primary objective of this segment is twofold:

1. To process the survey data in order to establish the analysis sample.
2. To analyze three survey questions that align with our three research questions.

Details on the methodology for each question are provided in the corresponding section for each research question and its associated survey question. The following sections outline the data processing steps and the analysis conducted for each question.

## Data Processing

### Clean Variable Names
The raw survey data required some processing before it could be analyzed. For example, in questions Q2 and Q3, the columns representing feature rankings were not labeled with the corresponding feature names. Instead, the dataset used generic labels such as "Q2_1", where the column represented the first feature option and the rows contained respondents’ reported rankings. To improve clarity and facilitate analysis, I renamed these columns to include the associated feature names.

The Qualtrics survey output includes the question text and metadata in the first and second rows of the dataset. For example, the first-row value for Q2_1 is:

>When purchasing a flight simulator yoke, rank the following features and capabilities from most important (1) to least important (12). Drag and drop to rank the yokes. - Xbox Compatibility.

To make the data easier to work with, the code extracts the feature name (the text appearing after the "–") from the first row of each question column and appends it as a suffix to the column name. This procedure is applied to both the Q2 and Q3 output columns.

```R 
# Question 2 -------------------------------------------------------------------
# : When purchasing a flight simulator yoke, rank the following features and 
# : capabilities from most important (1)  to least important (12). Drag and drop to rank the yokes.
#  -----------------------------------------------------------------------------

# Create temporary variable
tmp <- survey_rawdata

# Specify the columns to rename
columns_to_rename <- survey_rawdata %>% select(starts_with("Q2_")) %>% names()

# Extract parts after "-" from the first row for the specified columns
suffixes <- sapply(tmp[1, columns_to_rename], function(x) sub(".*-", "", x))

# Get current column names
current_names <- colnames(tmp)

# Create new names for the specified columns
new_names <- paste(columns_to_rename, suffixes, sep = "_")

# Rename the specified columns
colnames(tmp)[colnames(tmp) %in% columns_to_rename] <- new_names

# Clean names
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

# Create temporary variable
tmp <- tmp2

# Specify the columns to rename
columns_to_rename <- tmp %>% select(starts_with("q3_")) %>% names()

# Extract parts after "-" from the first row for the specified columns
suffixes <- sapply(tmp[1, columns_to_rename], function(x) sub(".*-", "", x))

# Get current column names
current_names <- colnames(tmp)

# Create new names for the specified columns
new_names <- paste(columns_to_rename, suffixes, sep = "_")

# Rename the specified columns
colnames(tmp)[colnames(tmp) %in% columns_to_rename] <- new_names

# Create temporary variables
tmp3 <- tmp

# Remove temporary variables
rm(tmp, tmp2, columns_to_rename, suffixes, current_names, new_names) # Remove temporary variables

```
### Apply Survey Filters 

To create the analysis sample, the data was filtered along the folloing points.

Responses were removed from:

**1. Survey Preview Responses**
<br>
Remove all preview responses relating to testing of the survey. 

**2. Respondents who do not own a flight simulation yoke** 
<br>
Some responses indicated that they did not own a yoke controller, and these responses were removed.
 
**3. Respondents who only have HOTAS style controllers**
<br>
Some responses indicated they owned controllers in a HOTAS (Joystick) format. Opinions of those who soley own these type of controllers 
were outside the scope of this survey.
 
**4. Remove respondents who quit survey after question 1** 
<br> 
Some respondents quit the survey after responding to question 1 and these responses were removed.

**5. Remove respondents with inconsistent location responses (Falsely claimed to live in the SF Bay Area)**
<br> 
To recruit potential participants for an in-person study, we asked respondents (Q12) if they lived in the San Francisco Bay Area. However, we observed a suspiciously high number of “yes” responses, despite the survey being disseminated globally via Reddit. To address this concern, we cross-checked responses with location metadata and removed cases where the coordinates fell outside the Bay Area. See code below for details of this filtering process.

The location filter flagged responses where the reported coordinates were more than 100 miles outside the Bay Area, despite the respondent indicating they lived there. The code below prepares for this filtering by creating a distance function, which is applied later in the program.

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

To identify respondents who only owned HOTAS-style controllers, we manually reviewed the unique text entry responses provided under the Q1 “Other” option (*Do you own any of the following flight simulation yokes? (select all that apply)*).

After identifying which controllers were HOTAS-style and determining the set of respondents who owned these exclusively, I compiled the controller names into an object called exc_filter, which was later used as a filter.

```R

# Prepare to filter to include only owners of Yoke style controllers (filter 3) -----------

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

The approach to filtering the data included creating dummy variables for each of the five filters, along with a binary analysis sample variable (ansamp) that indicates which rows are included in the final sample.

```R

# Create dummy variables for filters -------------------------------------------

tmp4 <- tmp3 %>%
  # Remove header rows
  slice(-(1:2)) %>%  
  # Create dummies
  mutate(
    ## Track pre-filtered rows
    initial_sample = 1,  
    ## If survey previews (filter 1)
    survey_preview = ifelse(status == "Survey Preview", 1, 0),  
    ## If no yoke ownership (filter 2)
    no_yoke = ifelse(grepl("I do not own a flight simulation yoke", q1, ignore.case = TRUE), 1, 0),  
    ## If only own HOTAS yokes (filter 3)
    hotas = ifelse(q1 == "Other" & grepl(exc_filter, q1_4_text, ignore.case = TRUE), 1, 0)  
  ) %>%
  rowwise() %>%
  mutate(
    # If quit after Q1 (filter 4)
    quit_aq1 = ifelse(all(is.na(c_across(q2_1_xbox_compatibility:q15))), 1, 0)  
  ) %>%
  ungroup() %>%
  mutate(
    location_latitude2 = as.numeric(location_latitude), # Make coordinates numeric
    location_longitude2 = as.numeric(location_longitude),
    # Calculate distance from San Ramon
    distance_to_sr = haversine_distance(location_latitude2, location_longitude2, 
                                        san_ramon_lat, san_ramon_lon),
    # If within 100 miles of San Ramon
    radius100 = ifelse(distance_to_sr <= 100, TRUE, FALSE),  
    # Claimed Bay Area but outside 100 miles (filter 5)
    inbay_ovr100 = ifelse(radius100 == FALSE & q12 == "Yes", TRUE, FALSE)  
  )

```
<br>

Next, the ansamp variable that defines the analysis sample is created used to form the filtered processed data file. 
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

These checks are run to see how many rows were removed and to inspect the removed rows. 
```{r}
# Check Number of Rows Removed ------------------------------------------------------------------

## OLD 
paste0("Old N: ", nrow(tmp4))

## NEW
paste0("New N: ", nrow(data))

## Spot check removed rows 
removed_rows <- tmp5 %>% filter(ansamp == 0)
```
<br> 

This table provides insight into why rows were filtered out by showing the number of responses removed for each combination of the five filter variables.
```R
# Check filters ----------------------------------------------------------------------------------

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
# (4) Respondents quitting after Q1
data %>% filter(if_all(q2_1_xbox_compatibility:q15, is.na))

# (5) Inconsistent location responses
locfilter_check <- data %>% 
  select(q12, ends_with("e2"), ends_with("100"), distance_to_sr, ansamp) %>% 
  filter(q12 == "Yes")

```

## Analysis

### RQ1: General Yoke Pitch Smoothness

To supplement the **product review analysis/LINK**, we fielded a survey to over 300 flight simulation enthusiasts and owners of competitor yokes. To address RQ1, respondents were asked (Q9):

>Do you find the smoothness and feel of the pitch axis an issue in your current yoke? (select all that apply)

with possible answers of:

- Yes, this is an issue for the Turtle Beach: Velocity One <br> 
- Yes, this is an issue for the Honeycomb Alpha <br>
- Yes, this is an issue for the Thrustmaster TCA Boeing <br>
- Yes, this is an issue for [Q1 Text Entry] <br>
- Pitch Axis smoothness is not an issue for any of these yokes <br>

[add pic of survet q]

At the beginning of the survey, respondents were asked whether they owned any of the three yokes listed above, or if not, to specify their yoke in a text entry field. Using Qualtrics display logic, respondents only saw response options corresponding to the yokes they owned. If a respondent indicated ownership of multiple yokes, the question was presented in a “select all that apply” format so that feedback could be captured for each relevant yoke.

The code below illustrates how responses were processed and frequencies generated for this question. This procedure was replicated separately for each of the three main yokes, as well as for an aggregated category of respondents who did not own any of the three primary competitors.

To prepare the data for frequency calculations, subsets were created for each of the four yoke categories (the three primary competitors and the “other yokes” group). Each subset was filtered using three parameters. In the example below for the Turtle Beach yoke, the filtering steps are as follows:

1. **Ownership filter** – Select only respondents who indicated in Q1 that they own the Turtle Beach yoke. This ensures the analysis sample is restricted to relevant owners.
2. **Response filter** – From Q9, select responses that either (a) mention the Turtle Beach yoke by name, or (b) include “not” (corresponding to the option “Pitch axis smoothness is not an issue for any of these yokes”).
3. **Combined subset** – The resulting dataset includes only:
 - Respondents who own the Turtle Beach yoke
 - Their responses indicating whether or not they experienced pitch smoothness issues.

Once filtered, the subset can be tabulated to calculate the frequency of respondents reporting issues with the pitch axis for the Turtle Beach yoke. This same process was repeated for the Honeycomb and Thrustmaster yokes, as well as for the aggregated “other yokes” category.


```R
# Process data
q9_tb_data <- data %>%
  filter(grepl("Turtle Beach Velocity One", q1, ignore.case = TRUE),
         (grepl("Turtle Beach: Velocity One", q9, ignore.case = TRUE) | 
          grepl("not", q9, ignore.case = TRUE)))

# Create a frequency table for the column 'q9' in the filtered data
table(q9_tb_data$q9)

```


Using the subsetted data from the previous section, I generated frequencies with the code below. First, a new column (q9) was created to recode the raw responses into a binary yes/no variable, indicating whether respondents reported pitch smoothness issues for the given yoke. Next, `summarize()` was used to calculate:

- The proportion of respondents who reported issues (yes_p),
- The numerator (the count of “yes” responses), and
- The denominator (the total number of respondents in the subset).

The yes_p column from this summarized dataset is then used in the next section to graph the findings across yokes.


```R
# Summarize data
q9_tb_propy <- q9_tb_data %>%
  mutate(q9 = ifelse(grepl(" not ", q9, ignore.case = TRUE), "no", "yes")) %>%
  summarise(yes_p = mean(q9 == "yes"), total_n = n(), yes_n = sum(q9 == "yes"))

q9_tb_propy

```


**Graphing** 

The process for creating these donut charts is very similar to the charts created for the product review findings outlined here **LINK**. 

```R
# Prepare data: Proportions of responses for 4 sets of responses
ggdonut_sq_data <- data.frame(
  Category = rep(c("turtle beach", "honeycomb", "thrustmaster", "other"), each = 2),  # 4 categories of responses
  Response = rep(c("Yes", "No"), times = 4),  # Only the "Yes" responses
  Proportion = c(q9_tb_propy$yes_p, (1-q9_tb_propy$yes_p),   # Proportions for turtle beach
                 q9_hc_propy$yes_p, (1-q9_hc_propy$yes_p),   # Proportions for honeycomb
                 q9_tmb_propy$yes_p, (1-q9_tmb_propy$yes_p),  # Proportions for thrustmaster
                 q9_othr_propy$yes_p, (1-q9_othr_propy$yes_p)) # Proportions for other
)

```
```R

# Function to create a single-level donut chart
create_donut_chart <- function(df, label) {
  ggplot(df, aes(x = 2, y = Proportion, fill = Response)) +
    geom_bar(stat = "identity", width = 0.8, color = "white") +  # Single donut
    coord_polar(theta = "y") +
    theme_void() +  # Remove axis and background
    xlim(0.5, 2.5) +  # Adjust the limit to create the hole
    geom_text(aes(label = ifelse(Response == "Yes", scales::percent(Proportion), "")),
              position = position_stack(vjust = 0.5), color = "black") +  # Percentage labels in black
    annotate("text", x = 0.5, y = 0, label = label, size = 4, color = "black") +  # Center label in black
    theme(legend.position = "none")  # Remove the legend
}

```

Next, after running my function for each of the 3 cateogories, I  used arrangeGrob to position the plots together and other GridExtra functions to customize aesthetics. 

```R

# Prepare graphs for the three categories
donut1 <- create_donut_chart(filter(ggdonut_pr_data, Category == "turtle beach"), "Turtle Beach")
donut2 <- create_donut_chart(filter(ggdonut_pr_data, Category == "honeycomb"), "Honeycomb")
donut3 <- create_donut_chart(filter(ggdonut_pr_data, Category == "thrustmaster"), "Thrustmaster")

# Arrange the three charts horizontally
gd_pr <- arrangeGrob(donut1, donut2, donut3, ncol = 3)

# Add a main title with adjusted positioning
grid.newpage()  # Clear the current page
grid.draw(gd_s)  # Draw the final grob
grid.text("Survey Question:\nDo you find the smoothness and feel of the pitch axis an issue in your current yoke?\n", 
          x = 0.5, y = 0.85, gp = gpar(fontsize = 11, fontface = "bold"))  # Adjusted title position

grid.draw(gd_s)  # Draw the grob

ggsave("donut_chart_s.svg", plot = gd_s, width = 12, height = 4, bg = "white")

```

### RQ2: Average Feature Rank

To address our second research question (RQ2): “Determine the value users place on the feel, smoothness, and precision of a flight yoke relative to other features”, we asked respondents to complete a ranking task pertaining to common flight simulation yoke features (Q2):

>When purchasing a flight simulator yoke, rank the following features and capabilities from most important (1) to least important (2). Drag and drop to rank the yokes.

For analysis, I subset the dataset to include only the columns corresponding to this ranking question and converted responses to numeric values. This allowed me to calculate the mean ranking for each feature, which serves as the basis for comparing their relative importance in the next step.

```R

# Display the modified data frame
avg_rank_data <- data %>%
  select(starts_with("q2"), response_id) %>% 
  mutate_at(vars(starts_with("q2")), as.numeric) # Convert to numeric

```

To generate the mean rankings, I applied the `summarize_all()` function across all columns to calculate the average score for each feature. The results were then pivoted into long format to make the output easier to interpret and ordered using `arrange()`.

```R
avg_rank_calc <- avg_rank_data %>% 
  select(-response_id) %>%
  summarise_all(mean, na.rm = TRUE) %>%  # Calculate the mean rank for each item
  gather(key = "Item", value = "Average_Rank") %>%  # Convert to long format for easier interpretation
  arrange(Average_Rank)  # Sort by average rank

avg_rank_calc # Mean ranking results

```

**Graphing** 

Before visualizing the results, I processed the data to match the desired aesthetics of the graphs. This included cleaning the values in the Item column using regular expressions and wrapping text to ensure labels fit neatly along the y-axis. I also rounded the values in the Average_Rank column and ordered the rows from lowest to highest average rank to improve readability in the final graphs.

```R

avg_rank_calc_gg <- avg_rank_calc %>% 
  mutate(Item = Item %>%
         sub("q2_[0-9]+_", "", .) %>%  # Remove the dynamic prefix
         gsub("_", " ", .) %>%         # Replace underscores with spaces
         toTitleCase(), # Capitalize the first letter of each word
         Average_Rank = round(Average_Rank, 2), # Round average rank value
         Item = ifelse( # Wrap text
           Item == "Smoothness Feel and Precision of the Control Axes", "Smoothness, Feel, and\nPrecision of Control Axes", Item),
         Item = ifelse(
           Item == "Included Throttle Control", "Included\nThrottle Control", Item),
         Item = ifelse(
           Item == "Included Rudder Brake Control", "Included Rudder\nBrake Control", Item),
         Item = ifelse(
           Item == "Adequate Resistance Force", "Adequate Resistance\nForce", Item),
         Item = ifelse(
           Item == "Play", "Plug-In-Play", Item)
         )  %>% 
  arrange(desc(Average_Rank)) # Order from lowest to largest average rank             

```

The preceeding dataset is then inputted into a ggplot function that creates a horizontal bar chart displaying average ranks of features in ascending order. 

```R

gg_afr <- avg_rank_calc_gg %>% 
  ggplot(aes(x = Average_Rank, y = Item)) +
  
  # Bars
  geom_segment(aes(x=0, xend=Average_Rank, y=Item, yend = Item), lwd=8, color = "#5EC6CC") +
  
  geom_rect(aes(xmin = Average_Rank, xmax = Average_Rank, 
                ymin = as.numeric(Item) - 0.25, ymax = as.numeric(Item) + 0.25)) + 
  
  # Annotation
  geom_text(aes(x= Average_Rank, y=Item, label = paste0("  ", Average_Rank)),
            hjust = 0, size=9/.pt, fontface = "plain", check_overlap = FALSE) +  

  # Axis Scales
  scale_y_discrete(
    breaks = unique(avg_rank_calc_gg$Item),
    labels = unique(avg_rank_calc_gg$Item),
    expand = c(0.05, 0)
  ) +
  
  scale_x_continuous(
    limits = c(-0.5, 12),
    expand = c(0,0)
  ) + 
  
  # Titles and theme
  ylab("Feature") +
  xlab("Average Rank") +

  theme_bw() + 
  theme(
    # Title
    plot.title = element_text(size=11, color = "#000000", face = "bold", margin = margin(0,0,15,0)),
    
    # Text
    axis.text.x = element_text(size = 9, color = "#000000", face = "bold", margin=margin(0,0,15,0)),
    axis.text.y = element_text(size = 9, color = "#000000", face = "plain", hjust = 0, margin = margin(t=0,r=10,b=0,l=0)),
    axis.title.x = element_text(size = 9, color = "#000000", face = "plain", margin = margin(t=-5,r=0,b=0,l=0)),
    axis.title.y = element_text(size = 9 , color = "#000000", face = "bold", angle = 0, margin = margin(r=-33, b=0, l=0), vjust = 1.02),
    
    # Gridlines
    panel.grid = element_blank(),
    plot.margin = unit(c(.2,.2,.2,.2),"in")) 

# Preview 
gg_afr

# Save
ggsave("bar_chart_afr.svg", plot = gg_afr, width = 8, height = 8)

```
[insert pic of graph]

### RQ3: Yoke Comparison Analysis
To address our third research question, RQ3: *“Assess the comparative desirability of the current product against its primary competitors”*, we designed a survey question modeled after a conjoint-style analysis.

Respondents were asked to imagine purchasing a flight simulation yoke priced under $400 and to rank four options from most to least desirable. Each option included a list of its features. Two of the options were designed to closely mirror the feature sets of leading competitor yokes (with branding removed to reduce bias), while the third represented our proposed product.

Specifically, Yoke A resembled the Turtle Beach model, Yoke B resembled the Honeycomb model, and Yoke C reflected our proposed product. We chose not to include the Thrustmaster yoke in this analysis, based on the assumption that it was less directly comparable to the proposed product.

This ranking exercise was conjoint-inspired in that it asked respondents to make tradeoffs between realistic product bundles, but it stopped short of a full conjoint design that decomposes the value of individual features. For our purposes, however, this streamlined approach was well-suited: our primary interest was not in estimating utilities for each feature (See RQ2), but rather in testing whether the combination of features in our proposed product could stand competitively against existing alternatives. By forcing respondents to directly compare bundled product options, the task produced evidence to evaluate relative desirability in this specific market context.

Full question text:
>Imagine you are looking to purchase a flight simulator yoke and are considering Yoke A, Yoke B, and Yoke C. The features of the three yokes are listed in the table below. Please rank the following yokes from (1) most desirable to (3) least desirable, assuming each yoke costs $400. Drag and drop to rank the yokes.

[add pic of survet q]

Similar to the analysis of the feature ranking question, I completed the same workflow for this question

``` R
avg_cjrank_data <- data %>%
  select(starts_with("q3"), response_id) %>% 
  mutate_at(vars(starts_with("q3")), as.numeric) # Convert to numeric

avg_cjrank_calc <- avg_cjrank_data %>% 
  select(-response_id) %>%
  summarise_all(mean, na.rm = TRUE) %>%  # Calculate the mean rank for each item
  gather(key = "Item", value = "Average_Rank") %>%  # Convert to long format for easier interpretation
  arrange(Average_Rank)  # Sort by average rank

avg_cjrank_calc # Mean Ranking Results

```

**Graphing**

Similar to the graphing of average feature ranks, the "Item" column was processsed and average rank values were rounded to improve the aesthetics of the bar chart. 

``` R
avg_cjrank_calc_gg <- avg_cjrank_calc %>% 
  mutate(Item = Item %>%
         sub(".*?(Y)", "\\1", .) %>%  # Remove everything prior to the first "Y"
         gsub("_", " ", .) %>%         # Replace underscores with spaces
         str_to_title(), # Capitalize the first letter of each word
         Average_Rank = round(Average_Rank, 2), # Round
         Item = ifelse(
           Item == "Yoke A", "Yoke A\nVelocity One", Item),
         Item = ifelse(
           Item == "Yoke B", "Yoke B\nHoneycomb XPC", Item),
         Item = ifelse(
           Item == "Yoke C", "Yoke C\nTest Yoke", Item))


```

This dataframe is then inputted into a ggplot function to create a horizontal bar chart displaying the average ranks of yokes in ascending order. 

```R

gg_cmr <- avg_cjrank_calc_gg %>% 
  ggplot(aes(x = Average_Rank, y = Item)) +
  
  # Bars
  geom_segment(aes(x=0, xend=Average_Rank, y=Item, yend = Item), lwd=8, color = "#5EC6CC") +
  
  geom_rect(aes(xmin = Average_Rank, xmax = Average_Rank, 
                ymin = Item, ymax = Item)) + 
  
  # Annotation
  geom_text(aes(x= Average_Rank, y=Item, label = paste0("  ", Average_Rank)),
            hjust = 0, size=9/.pt, fontface = "plain", check_overlap = FALSE) + 

  # Axis scaling
  scale_y_discrete(
    breaks = unique(avg_cjrank_calc_gg$Item),
    labels = unique(avg_cjrank_calc_gg$Item),
    expand = c(0.3, 0)
  ) +
  
  scale_x_continuous(
    limits = c(-0.1, 3),
    expand = c(0,0)
  ) + 
  
  # Titles and theme
  ylab("Yoke") +
  xlab("Average Rank") +

  theme_bw() +  # Use a clean theme
  theme(
    # Title
    plot.title = element_text(size=11, color = "#000000", face = "bold", margin = margin(0,0,20,0)),
    
    # Text
    axis.text.x = element_text(size = 9, color = "#000000", face = "bold", margin=margin(0,0,0,0)),
    axis.text.y = element_text(size = 9, color = "#000000", face = "plain", hjust = 0.5, margin = margin(t=,r=10,b=0,l=0)),
    axis.title.x = element_text(size = 9, color = "#000000", face = "plain", margin = margin(t=0,r=0,b=0,l=0)),
    axis.title.y = element_text(size = 9 , color = "#000000", face = "bold", angle = 0, margin = margin(r=-45, b=0, l=0), vjust = 1.04),

    # Gridlines
    panel.grid = element_blank(),
    plot.margin = unit(c(.2,.5,.2,.5),"in")) 

gg_cmr

ggsave("bar_chart_cmr.svg", plot = gg_cmr, width = 8, height = 4)

```

[insert pic of graph]



