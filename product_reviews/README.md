# Product Review Data Processing & Analysis

The primary objective of this segment of the research is to gain insights toward RQ1: 

*Understand whether users of flight simulation yokes perceive feel, smoothness, and precision issues in current market offerings*.

Our primary method to acheive this goal involved a qualitative analysis of product reviews on online retail websites selling the yokes. Additionally in a survey we administered after this analysis, we included a question to ask directly users of these yokes if they have any issues with the features described in RQ1 (See full survey work here: LINK). 

Read each section below for more details on the data collection and analysis workflow: 

## Data Management

### Webscraping Program 

To start the data collection process, I designed a program using a python notebook to scrape raw html code for any product review on the page of the inputed URL. In our case the majority of the reviews I scraped originated from Amazon, which limits the amount of reviews per page to 10. For this reason, this program needed to be ran on each page of reviews available on the given retailer website. 

The program extracts details such as the reviewer name, star rating, review title, review date, purchase verification, and the review text for further analysis.

#### Dependencies

This program uses the *urllib* package to interface and connect with the inputed url (*Request(), urlopen()*) while using the *bs4* package to parse the html content (*BeautifulSoup()*) and extract the elements we are scraping. Furthermore *Pandas* is used to format and manipulate the product review data into a dataframe that can be saved. Lastly *os* and *datetime* are used for managing directories and metadata for the program. 

#### Prepare function to extract reviews 

```
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
```
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
```
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
### Review Cleaning 

## Analysis

### Qualitative Coding 

### Survey Question Frequencies

## Setup 

### Libraries
