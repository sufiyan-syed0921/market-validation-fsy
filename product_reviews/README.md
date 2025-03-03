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



### Survey Questionaire

To supplement the review analysis, in survey to over 300 flight simulation enthisiasts and owners of our competitor yokes we asked the following question: 

"Do you find the smoothness and feel of the pitch axis an issue in your current yoke? (select all that apply)" 

with possible answers of: 

- Yes, this is an issue for the Turtle Beach: Velocity One
- Yes, this is an issue for the Honeycomb Alpha
- Yes, this is an issue for the Thrustmaster TCA Boeing
- Yes, this is an issue for [Q1 Text Entry]
- Pitch Axis smoothness is not an issue for any of these yokes

At the very beggining of the survey, respondents were asked if they own any of the yokes present in the responses above and if not any of the 3, to write in the one they own as a text entry response. Using the Qualtrics question parameters, respondents were only shown responses options for this question to the yoke's they own. If respondents indicated that they owned more than one yoke, they were presented multiple response options as a select all that apply question that reflects the yoke that they own.

The code below demostrates how I proceeded to process and generate frequencies for this question. This was replicated for each of the three main yokes and an aggregate statistic for respondents who did not own any of the 3 main yokes.

``` R
q9_tb_data <- data %>%
  filter(grepl("Turtle Beach Velocity One", q1, ignore.case = TRUE),
         (grepl("Turtle Beach: Velocity One", q9, ignore.case = TRUE) | 
          grepl("not", q9, ignore.case = TRUE)))

# Create a frequency table for the column 'q9' in the filtered data
table(q9_tb_data$q9)

```

To prep for the frequencies, I subsetted datasets for each of these 4 yokes that filters on three parameters. In the example above, we first select responses for Q1 that matches to the Turtle Beach yoke name to ensuring our total sample contains only those who own the yoke. The second argument in the filter function selects responses of Q9 that contain the Turtle Beach Yoke name or responses of Q9 that contain "not" (which selects the "Pitch Axis smoothness is not an issue for any of these yokes" response level). In result these filters subset the data to incude only 1. Those who own the Turtle Beach Yoke, 2. Those who either responded that the Turtle Beach yoke had pitch smoothness issues and those who did not. Tabling this subseted dataset can then be used to get the frequency of respondents who had perceived issues with pitch smoothness for the Turtle Beach yoke.


## Analysis

### Survey Question Frequencies

```R
q9_tb_propy <- q9_tb_data %>%
  mutate(q9 = ifelse(grepl(" not ", q9, ignore.case = TRUE), "no", "yes")) %>%
  summarise(yes_p = mean(q9 == "yes"), total_n = n(), yes_n = sum(q9 == "yes"))

q9_tb_propy
```
Using the subseted data from the previous section for Q9, I use the code above to generate the frequencies. Furthermore I create another column named q9 to recode the raw variable to a yes/no binary variable and then use summarize to generate the proportion of those who found issues in pitch smoothness in the yoke (yes), and the denominator and numerator for that calculation (the total sample size and count of "yes" responses". 


### Qualitative Coding & Frequencies



## Setup 

### Libraries
