# Product Review Data Processing & Analysis

The primary objective of this segment of the research is to gain insights toward RQ1: 

*Understand whether users of flight simulation yokes perceive feel, smoothness, and precision issues in current market offerings*.

Our primary method to acheive this goal involved a qualitative analysis of product reviews on online retail websites selling the yokes. Additionally in a survey we administered after this analysis, we included a question to ask directly users of these yokes if they have any issues with the features described in RQ1 (See full survey work here: LINK). 

Read each section below for more details on the data collection and analysis workflow: 

## Data Collection & Management

### Webscraping Program (LINK to file)

To start the data collection process, I designed a program using a python notebook to scrape raw html code for any product review on the page of the inputed URL. In our case the majority of the reviews I scraped originated from Amazon, which limits the amount of reviews per page to 10. For this reason, this program needed to be ran on each page of reviews available on the given retailer website. 

The program extracts details such as the reviewer name, star rating, review title, review date, purchase verification, and the review text for further analysis.

#### Dependencies

This program uses the *urllib* package to interface and connect with the inputed url (*Request(), urlopen()*) while using the *bs4* package to parse the html content (*BeautifulSoup()*) and extract the elements we are scraping. Furthermore *Pandas* is used to format and manipulate the product review data into a dataframe that can be saved. Lastly *os* and *datetime* are used for managing directories and metadata for the program. 

#### Prepare function to extract reviews 

```Python
# add headers
headers = {"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36"}

# define function to get webpage results into Beautifulsoup object 
def get_page(url, headers):
    try:
        encoded_headers = {key: value.encode('utf-8') for key, value in headers.items()}
        req = Request(url, headers=encoded_headers)
        page = urlopen(req)
        soup = BeautifulSoup(page, "html.parser")
        return soup
    except Exception as e:
        print(f"Error occurred: {e}")
        import traceback
        traceback.print_exc()

```
Using your inputed user agent into the "headers" object the defined get_page() function uses this user agent and the url to make the HTTP request to parse and return the raw html code in the soup object. 

The code below runs the get_page() function on the inputed url to retrieve the reviews. The loop aded to the function adds a check to see if the html was successfully parsed.

#### Gather HTML from URL

``` Python
# Loop until header1 is not None
while True:
    # Get html
    soup = get_page("https://www.amazon.com/Thrustmaster-Yoke-PACK-Boeing-Xbox-x/dp/B09DPYHM55/ref=sr_1_5?crid=29KZ42OY0C859&dib=eyJ2IjoiMSJ9.gQXGQTXbcMi54OeLHcBcCdy8bUzAiP_khKF2HF5sDWteRLmjwsCoEm-T8ES3RJDk_pwvrXj5zel4nKbkyOA8Kc_d9eOe7P4LdnBf0ENU7pdjnW7xTK1Qdyl53s_jPhNO8xJh635sqWj92SiLQhRkqB6v3zgWbPwxbaXYM6A05uYAoi3s7d1u3lfAfrqrdOXxUbA2DWaManb3dv3M-e02zwsWrCouonZwu86t9YX1RCs.cKc6vAHkdCKax4iiYjtUPIMLhnIJikbZP9z_UlLYEaM&dib_tag=se&keywords=thrustmaster&qid=1733025366&sprefix=thrustmas%2Caps%2C203&sr=8-5&th=1", headers=headers)

    # Get h1
    header1 = soup.find('h1')

    # Check if header1 is not None
    if header1 is not None:
        break  # Exit the loop if header1 is not None

```

Now we scrape the key elements from the raw html into a list. We start by locating all 'div' elements with he attribute data-hook="review" which correlates to the html code for each review and save the bs4 output. Then, by using html elements, attributes, and classes as filters to the html code to retrieve the key information as the loop iterates over each element in our reviews_content output. Lastly we append the filtered review information to the emtepy reviews_list object we created at the start. 

#### Extract Review Data 

``` Python
# Create an empty list to store dictionaries for each review
reviews_list = []

review_content = soup.find_all('div', {'data-hook': 'review'})

# Extract Contents
for review in review_content:
    reviewer_name = review.find('span', {'class': 'a-profile-name'}).get_text().strip()
    star_rating_tag = review.find('i', {'data-hook': ['review-star-rating', 'cmps-review-star-rating']})
    star_rating = star_rating_tag.get_text().strip() if star_rating_tag else "No Rating"
    review_title_tag = review.find('a', {'data-hook': 'review-title'})

    # If the review title is not found within an <a> tag, try finding it within a <span> tag
    if review_title_tag is None:
        review_title_tag = review.find('span', {'data-hook': 'review-title'})

    # Extract the text of the review title from the tag, if found
    if review_title_tag is not None:
        review_title = review_title_tag.get_text().strip()
    else:
        review_title = "Title not found"
    review_date = review.find('span', {'data-hook': 'review-date'}).get_text().strip()
    purchase_verification_tag = review.find('span', {'data-hook': 'avp-badge'})
    purchase_verification = purchase_verification_tag.get_text().strip() if purchase_verification_tag else ""
    review_text_spans = review.find_all('span', {'data-hook': 'review-body'})
    review_text = ' '.join([span.get_text().strip() for span in review_text_spans])

    # Append review as a dictionary to the list
    reviews_list.append({'Reviewer': reviewer_name,
                         'Star Rating': star_rating,
                         'Review Title': review_title,
                         'Review Date': review_date,
                         'Purchase Verification': purchase_verification,
                         'Review Text': review_text})
```
### Review Proccessing (LINK) 

Moving on to this next program that manages the review data we just scraped. This program performs the following tasks: 
 1. Counts and displays the number of rows in each CSV file, as well as the total number of rows across all files.
 2. Concatenates multiple CSV files into a single DataFrame.
 3. Cleans the data by removing duplicates and unnecessary columns.
 4. Translates non-English review titles and review texts to English using the Google Translate API.
 5. Saves the cleaned and consolidated data into an Excel file for further analysis.

#### Dependencies

This program uses *os* to retrieve and manage directories and *pandas* to save and manipulate information in dataframes. I use *numpy* and *string* for futher manipulation and management of the review data. For reviews in languages other than english, I use the *googletrans* package for detecting and translating review text and title information. *traceback* is used for debugging the *translate_to_english* function that I define in the program. Lastly *os* and *datetime* are used to retrieving and managing directories of inputs/outputs and program metadata.  

### Inspect Row Counts for Product Review Data Files

Given the webscraping program did not include pagination functionality, the program needed to be ran every page available of product reviews. This left us with multiple csv files that needed to be concetenated and cleaned to create the analysis file. Given we are working with multiple files, I started off with a check to count the rows of each .csv file while also counting the rows of the concatenated file to ensure no duplicate or missing rows. 

This code below assumes each individual .csv file of review data starts with the same name and is in your current working directory. 

``` Python
# Note that this program assumes csv files of review data are present in your current working directory. Alter the code
# under "Directory containing the CSV files" to specify the directory you want the program to look for your files

## Define Function to count rows
def count_rows(csv_file):
    df = pd.read_csv(csv_file)
    return len(df)

## Directory containing the CSV files
csv_directory = os.getcwd()

## List all CSV files in the directory
csv_files = [os.path.join(csv_directory, file) for file in os.listdir(csv_directory) if file.startswith('honeycomb') and file.endswith('.csv')]

## Dictionary to store counts for each file
file_counts = {}

## Variable to store the total number of rows
total_rows = 0

## Loop through CSV files and count rows
for file in csv_files:
    rows_count = count_rows(file)
    file_name = os.path.basename(file)
    file_counts[file_name] = rows_count
    total_rows += rows_count
    print(f"File: {file_name}, Rows: {rows_count}")

## Display the total number of rows
print(f"\nTotal number of rows across all files: {total_rows}")

## Clean enviornment
del csv_directory, file_counts, total_rows, file, file_name, rows_count 

```

### Translate any non-english reviews to english  

One characteristic of this webscraped data is that we collected multiple reviews that were in languages other than english. Given this I created a function to translate content outside of english using the Google Translate API (googletrans package). The function completes this by (1) detecting any (non-punctuation) text exists and (2) running all other text into an english translator. This function is then ran on all review title and text content.


``` Python
## Define Translation Function
def translate_to_english(text, src_language='auto'):
    try:
        if pd.isnull(text):
            return text
        elif all(char in string.punctuation or char.isspace() for char in text):
            print("Skipping punctuation-only text:", text)
            return text
        else: 
            translator = Translator()
            print("Translating:", text)
            translated_text = translator.translate(text, src=src_language, dest='en')
            print("Translating to English:", translated_text.text)
            return translated_text.text
    except Exception as e:
        print(f"An error occurred during translation: {e}")
        print(f"Original text: {text}")
        traceback.print_exc()  # Print the traceback for debugging
        return text

## Test Function
text_to_translate = "Bonjour tout le monde"  # French text
translated_text = translate_to_english(text_to_translate)
print(translated_text)  # Output: Hello everyone

del text_to_translate, translated_text

```

## Analysis

### Product Reviews

Our goal with analyzing the qualitative data was to obatain the percentage of reviews (3 stars or lower) that mention issues of yoke smoothness or related issues for each of the 3 yoke models. After coding each of the reviews to determine if they mention smoothness issues, I uploaded a spreadsheet that contains this coding data in a column to table. For more information on the qualitative analysis process see:   

```R
# Turtle beach
addmargins(table(turtlebeach_reviewdata$yokesmooth_class_result2))

# save proportion for later graphing
tb_review_prop <- round(mean(turtlebeach_reviewdata$yokesmooth_class_result2 == "Yes"), 2)
tb_review_prop 

```

```R
# Honey comb
addmargins(table(honeycomb_reviewdata$yksmooth_class_result_10))

# save proportion for later graphing
hc_review_prop <- round(mean(honeycomb_reviewdata$yksmooth_class_result_10 == "Yes"), 2)
hc_review_prop

```

```R
# Thrust master
addmargins(table(thrustmaster_reviewdata$yksmooth_class_result_r))

# save proportion for later graphing
tmb_review_prop <- round(mean(thrustmaster_reviewdata$yksmooth_class_result_r == "Yes"), 2)
tmb_review_prop 
```

**Graphing** 

After calcualting the frequencies, I used these objects to to graph them in a series of donut charts for the 3 competitor yokes. First I began by preparing a dataset that I will input into my graphing function. The dataset included 3 columns: the yoke name or "category", the response in either "Yes" or "No" for the smoothness question, and the proportions for the yes and no responses.  

``` R
# Prepare data: Proportions of responses for 3 sets of responses
ggpie_rdata <- data.frame(
  Category = rep(c("turtle beach", "honeycomb", "thrustmaster"), each = 2),  # 3 categories of responses
  Response = rep(c("Yes", "No"), times = 3),  # Single question, 2 possible answers (Yes/No)
  Proportion = c(tb_review_prop, (1-tb_review_prop),   # Proportions for turtle beach
                 hc_review_prop, (1-hc_review_prop),   # Proportions for honeycomb
                 tmb_review_prop, (1-tmb_review_prop))  # Proportions for thrustmaster
)

```

Since we were going to be creating multiple charts in this program, I craeted a function to save some of the formating details and specifics to run on all the charts. 

```R
# Function to create a single-level donut chart with middle label
create_donut_chart <- function(df, label) {
  ggplot(df, aes(x = 2, y = Proportion, fill = Response)) +
    geom_bar(stat = "identity", width = 0.8, color = "white") +  # Single donut
    coord_polar(theta = "y") +
    theme_void() +  # Remove axis and background
    xlim(0.5, 2.5) +  # Adjust the limit to create the hole
    geom_text(aes(label = ifelse(Response == "Yes", scales::percent(Proportion), "")),
              position = position_stack(vjust = 0.5), color = "black") +  # Percentage labels in black
    annotate("text", x = .5, y = 0, label = label, size = 4, color = "black") +  # Center label in black
    theme(legend.position = "none")  # Remove the legend
}

```

Next, after running my function for each of the 3 yokes, I  used arrangeGrob to position the 3 plots together and other GridExtra functions to customize aesthetics. 

``` R 

# Prepare graphs for the three categories
donut1 <- create_donut_chart(filter(ggpie_rdata, Category == "turtle beach"), "Turtle Beach")
donut2 <- create_donut_chart(filter(ggpie_rdata, Category == "honeycomb"), "Honeycomb")
donut3 <- create_donut_chart(filter(ggpie_rdata, Category == "thrustmaster"), "Thrustmaster")

# Arrange the three charts horizontally
g_ar <- arrangeGrob(donut1, donut2, donut3, ncol = 3)

# Add a main title with adjusted positioning
grid.newpage()  # Clear the current page
grid.rect(gp = gpar(fill = "white", col = NA))  # Set background color behind all charts
grid.text("Percentage of reviews (3 stars or lower) that mention yoke smoothness\nand related problems as an issue", x = 0.5, y = 0.85, gp = gpar(fontsize = 11, fontface = "bold"))  # Adjusted title position

grid.draw(g_ar)  # Draw the grob

ggsave("donut_chart_ar.svg", plot = g_ar, width = 12, height = 4, bg = "white")

```


### Qualitative Analysis

After scraping and collecting the product review data, the next step was to analyze the data qualitatively. Our intention was to see if among the reviews, any common complaints of the products aligned with our intended positioning and key value points of our product that would enter the market. 

Before we webscraped began to analyze the reviews, we purchased and tested our competitor yokes to generate exploratory hypotheses of what common complaints of the competitor products would exist that align with our key value points of our prototype. We coded each of the reviews if they contained each of these topics and calculated the percentage of reviews that each topic was observed. Furthermore we were able to test our hypotheses by assesing the proportion of positvely coded reviews for each topic. 

Given this context of trying to asses product-market fit, we choose to only analyze reviews with 3 stars or under. We made this assumption because we assumed if reviews 3 stars or under mentioned the topics of interest, it would be more meaningful relative to influencing purchasing descisions than reviews with 4 or 5 stars. We assumed that reviews with 4 or 5 stars, the negative sentiment caused by the topic would not be significant enough to sway a purchasing decision. Furthermore instead of these reviews possibly inflating the occurance of reviews expressing negative sentiment along our target topics, we did not include these in our analysis sample. 

The key pain points we inspected each review to contain included:


#### Yoke Smoothness Issues:

The primary improved value point of our product entering the market was an improved smoothness and quality of the pitch action. 

Thus, for each of our competitor yokes, we inspected every review to see if it contained negative sentiment and customer dissatisfaction with the quality and feel of the controller's yoke, especially concerning its pitch axis. Specifically reviews were flagged if they included complaints regarding the smoothness, stickiness, jerkiness, or any issues related to being stuck or unresponsive with the yoke's pitch axis. Additionally reviews mentioning specifc issues regarding the pitch auto-centering or "center-dent" and elevator axis negatively affecting the yokes performance. The results are shown below and described in the graphs above. 

Honeycomb Yoke: 31% (25/82) of reviews (3 stars or lower) mention the yoke smoothness and related problems as an issue.

Thrustmaster Boeing: 10% (5/50) of reviews (3 stars or lower) mention the yoke smoothness and related problems as an issue.

Turtle Beach Velocity One: 30% (52/174) of reviews (3 stars or lower) mention the yoke smoothness and related problems as an issue.


#### Realism: 

One assumption we held for our product was that due to the construction and improved mechanics, it would offer a more realistic flight simulation experience relative to our competitor yokes. Thus, we analyzed each review for the competiotor yokes to see if they contained dissatisfaction with realistic esperience provided by the yoke and if the yoke's construction fails to effectively stimulate a real-life flight experience. For the 3 competitors here is what we found:

Velocity One: 6% (10/174) of reviews (3 stars or lower) mention realism as an issue.

Thrustmaster Boeing: 0% (0/50) of reviews (3 stars or lower) did not find explicit mentions of a poor realistic experience for this yoke.

Honeycomb Alpha: 13% (11/82) of reviews (3 stars or lower) mention realism as an issue. 


#### Pitch axis is too short

Particularly with the Velocity One, we wanted to explore explicitly if customers thought the travel in the pitch axis was too short. We hypothesized that short pitch travel was a facet of poor realism which is detailed further down this list. Ultimately we found no reviews in our analysis with this specific complaint. 


#### Looks or feels like a toy & issues with plastic construction

Another hypothesis we had for the Velocity One yoke was that customwers could be displeased with its plastic construction and orientation as more of a gaming console controler vs a virtual flight simulation controller. Reviews with complaints of its appearance, the physical feel of the yoke and comments that characterize the yoke as "Toyish" and dissatisfactions regarding its plastic construction. Ultimatley we found that 16% (27/174) of Velocity One reviews (3 stars or lower) mention the yoke plastic, toy-like construction and related build quality as an issue. 


#### Grinding and Noisy: 

Specific to the Honeycomb Alpha, real life testing of the product found it's pitch axis to be particularly noisy and sometimes exhibiting a grinding feeling. We wanted to test if this was a common complaint amongst consumers of this yoke. We analyzed reviews of the Honeycomb Alpha to see any reviews mentioned a grinding feeling or noise when moving the yoke laterally. Our analysis found that 4% (3/82) of reviews (3 stars or lower) mentioned the yokes' pitch axis grinding and being noisy.


#### Pitch is too strong (Travel is too heavy):

In addition to the grinding and noisiness of the pitch action in the Honeycomb yoke, we found the weighting to be relatively heavy compared to the other yoke options. We wondered if this too was a common complaint of users so we analyzed each review to see if it conained negative sentiment towards the resistance or difficulty when moving the pitch axis up and down. We found that 22% (18/82) of reviews (3 stars or lower) mention the pitch is too strong as an issue.


#### Pitch Stickiness

Specific to the Thrustmaster Boeing, our time testing the product revealed a sometimes sticky feeling when moving the pitch axis laterally. We tested this claim for and analyzed each review for negative sentiment revolving around a high degree of friction or "stickineess" in moving the yoke laterally. We found that 6% (3/50) of reviews (3 stars or lower) mention stickiness of the yoke as an issue. 

