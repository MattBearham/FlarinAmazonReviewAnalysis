# Flarin - Comparative Amazon Review Analysis

For this project, the consumer health company Infirst Healthcare wanted to extract insights from amazon reviews for their product Flarin, using natural language processing methods. In order to provide additional insights, these were compared with similar products on the market.

## Preprocessing

### Scraping Reviews

The reviews are retrieved from Amazon using the python package **BeautifulSoup**. Using a pre-formulated cookie, the first page of reviews is retrieved using the **requests** package and the page URL. Using the HTML pointers, we find the total number of reviews for the product: \
\
`num_reviews = soup.findAll("div",{'data-hook':'cr-filter-info-review-rating-count'})`\
\
Once the total number of reviews is retrieved and extracted, we can calculate the number of pages of reviews we need to access by dividing the total number by 10 and rounding down. From this, considering Amazon's URL generation, we can create create a list of URLs which contain all
of the reviews for the product. For each page, text and dates of reviews are located in the text using the HTML pointers in the getReviews() function: \
\
`reviews = soup.findAll("span", {'data-hook':"review-body})`\
`dates = soup.findAll("span", {'data-hook':"review-date"})`\
\
This gives us a lovely list of each review, with the date it was written.

### Data Cleaning

Review text is cleaned by removing special characters and striping blank spaces. Dates are transformed into a datetime format. \
\
The review text is then tokenized (using the word_tokenize() function from the **nltk** package) in order to prepare it for the NLP models. Individual words are then processed by removing stopwords (`stopwords = nltk.corpus.stopwords.words("english")`) and processing abbreviations (e.g. n't -> not).

## Analysis

### Word Frequency Analysis

First of all, as a preliminary analysis we create frequency distributions for the tokenized reviews using the FreqDist() function from **nltk**. This is later displayed as a bar chart.\
\
Following this, the **nltk** BigramCollocationFinder and TrigramCollocationFinder objects are used to find the top 5 most common pairs and triplets of words respectively, giving more of an insight into the phrases and context of what is being stated in the review (e.g. the phrase 'good pain relief').\
\
All three graphs are presented as subplots on the figure below. The colours are calculated iteratively as a gradient between the darkest and lightest tone before plotting.

![Word Frequency Analysis](/Flarin/flarin_word_frequencies.png)

### Sentiment Analysis

The sentiment of each review was calculated using the SentimentIntensityAnalyzer model from **nltk.sentiment**. Starting with an empty dictionary, the polarity scores were calculated for each review and assigned a category according to the table below:

|Classification |Value Range   |
|-------------- |-------------:|
|Strong Negative|<-0.65        |
|Negative       |-0.65 - <-0.35|
|Slight Negative|-0.35 - <-0.10|
|Neutral        |-0.10 - <0.10 |
|Slight Positive|0.10 - <0.35  |
|Positive       |0.35 - <0.65  |
|Strong Positive|>=0.65        |

Overall sentiment is plotted as a bar chart below under 'Overall Sentiment':

![Overall Sentiment](/Flarin/flarin_sentiment_analysis.png)

Following this, the sentiment is plotted over time to show any changes in public sentiment. The age of a review is calculated according to the formula `(d1.year - d2.year) * 12 + d1.month - d2.month`. Reviews are then stored in a dictionary `time_grouped_sentiments` using their age in months as an index.
From this, the average sentiment of the reviews for each month is calculated. Since the sample size from month to month is highly variable and sometimes small, confidence intervals were calculated to convey this - this was done using **scipy.stats**'s t.interval() function.\
\
The average sentiment is then plotted along with error bars to product the following figure:

![Sentiment Analysis over Time](/Flarin/flarin_sentiment_over_time.png)

To supplement this plot per the request of the client, a plot of the number of reviews per month over the previous 12 months was also included:

![Reviews per Month](/Flarin/flarin_reviews_12_month.png)

The client suggested that a combination of the two sentiment displays would be useful. To produce this plot, reviews were grouped into buckets spanning 3 months. Within each 3-month period, a less specific sentiment categorization plot was plotted according to the table below:

|Classification |Value Range   |
|-------------- |-------------:|
|Very Negative  |<-0.65        |
|Negative       |-0.65 - <-0.35|
|Neutral        |-0.10 - <0.10 |
|Positive       |0.35 - <0.65  |
|Very Positive  |>=0.65        |

This plot more effectively shows the change in distribution of sentiment over time:

![Grouped Review Sentiment Over Time](/Flarin/flarin_sentiment_distribution.png)

### Word Cloud

Since this project was for the client's marketing department, an easily-digestible, visual display would convey information well. Using the **wordcloud** package, a wordcloud was generated of the most common words in reviews. While this shows similar information to the frequency analysis, this way
of displaying data was very well recieved by the client. Colours were generated using the random_color_func() function which is passed into the WordCloud() function to align the figure with the Flarin brand identity.

![Word Cloud](/Flarin/flarin_wordcloud.png)

### Stomach Mentions

Finally, since this product is a formulation of ibuprofen, the client was specifically interested in mentions of the GI side effects; stomach protection is also a large part of the brand's identity. As a late ad-hoc request, a simple plot of mentions of the word 'stomach' over time was included:

![Stomach Mentions](/Flarin/flarin_stomach_mentions.png)

## Comparative Analysis

The code is set up in such a way that by simply changing the URL in the request, this report can be produced for any product on Amazon. This allowed for comparison of Flarin's reviews with those of other similar competitors, such as Nurofen, Voltarol, and FlexiSeq. These plots were produced in separate
files, and the figures presented alongside the Flarin ones in the slide deck [here](AmazonReviewsSummary.pptx).
