# Market Validation Report: Flight Simulation Yoke Controller

## Background

This repository aims to summarize research results from an exploratory analysis on the outcome of introducing an additional product offering in the flight simulation yoke controller market.The proposed product has one main advantage over current market offerings, which we aim to test: an increased perceived feel, smoothness, and precision of yoke movement at a price point under $400. Our main product competitors we outlined for this analysis is the Turtle Beach: VelocityOne, Honeycomb XPC, and the Thrustmaster TCA Boeing. Given this, we set out to answer 3 key questions in an attempt to validate a market for this product:

##### *RQ 1: Understand whether users of flight simulation yokes perceive feel, smoothness, and precision issues in current market offerings.​*
##### *RQ 2: Determine the value users place on the feel, smoothness, and precision of a flight yoke relative to other features.​*
##### *RQ 3: Assess the comparative desirability of the current product against its primary competitors.​*

## Methods & Data Processing
Our methodological approach to answering these questions included data sources in customer product reviews and survey data

### Product Reviews 
Web-scraped 306 customer reviews (3 stars or under) from Amazon, Walmart, Best Buy and other popular retailers​

Sample breakdown for each yoke:​
- Turtle Beach Velocity One (n=174) ​
- Honeycomb Aeronautical XPC (n=82)​
- Thrustmaster TCA Boeing (n=50)

#### Code
For more detailed information on data processing, please review this code file and readme:
[Product Review Data Processing & Analysis](https://github.com/sufiyan-syed0921/market-validation-fsy/tree/main/product_reviews)

### Survey
Surveyed approximately 300 yoke users on the r/flightsim subreddit using Qualtrics​. Users who took the survey were entered into a raffle for a $100 Steam gift card​. 

Responses were filtered out of the final analysis for 2 key reasons: 
- **Yoke Ownership:**
Users who do not own a flight yoke, or those who quit the survey after indicating yoke ownership, were removed.

- **Inconconsitent Location Responses:**
Respondents were asked if they are local to the SF Bay Area. If so, respondents were further asked if they would be interested in participating in an in-person user study. We received a significant number of responses claiming to be local to the SF Bay Area while having IP addresses far outside the SF Bay Area which raised concerns of untruthful responses. These responses were removed from the sample.​

After these filters, the post-processed sample size for the survey data was 157. 

#### Code
For more detailed information on data processing, please review this code file and readme:
[Survey Data Processing & Analysis](https://github.com/sufiyan-syed0921/market-validation-fsy/tree/main/survey)


## Results 

### RQ 1: Understand whether users of flight simulation yokes perceive feel, smoothness, and precision issues in current market offerings.​

<div align="center">
  <h4>Amazon Reviews</h4>
</div>

![Donut Chart AR](donut_chart_ar.svg)
  
<div align="center">
  <h4>Survey Question</h4>
</div>

![Donut Chart S](donut_chart_s.svg)

### RQ 2: Determine the value users place on the feel, smoothness, and precision of a flight yoke relative to other features.​

- Feature Ranking

![Bar Chart AFR](bar_chart_afr.svg)

### RQ 3: Assess the comparative desirability of the current product against its primary competitors.​

- Product Ranking

![Bar Chart CMR](bar_chart_cmr.svg)

### Limitations

## Conclusions
