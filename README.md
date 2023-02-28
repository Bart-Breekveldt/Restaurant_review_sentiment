# Restaurant_review_sentiment

This project is executed for the course Critical Data Mining of Media Culture (Utrecht University) in which natural language processing techniques are adressed. The project is executed with Christian Acosta, Malka Aktepe and Jeroen Dekker. The main goal: "How can sentiment analysis be used to predict the star rating of restaurant reviews?" We did this with the VADER-sentiment analysis and determine, while it is trained in a social media context can also be used in the restaurant review setting. The city selected is London, England. This is due to (1) English nlp-analyses are generally considered to be the most sophisticated, and untranslated reviews works best and (2) London is a highly multi-cultural city with lots of cuisines, inc

**1. webscraping of restaurant reviews from Google Maps and Tripadvisor**

For retrieving restaurant reviews at first Google Maps is used as a data source. Retrieving reviews from the internet is executed by webscraping, which is facilitated by the Google Places API. A Puppeteer scraper hosted on the Apify platform is used to perform the retrieval of reviews and relevant data. Puppeteer is a browser automation library that allows you to control a browser using JavaScript. The Google Places API returns information about places using HTTP requests. A Puppeteer scraper hosted on the Apify platform is used to perform the retrieval of reviews and relevant data. Resulting in 29.000 reviews for London. Reviews by restaurant are limited to 250 to prevent overrepresentation of a few highly reviewed restaurants.

To avoid bias towards one review-sorting alorithm and the reviews it got was to use a second source: Tripadvisor. We used a self-built algorithm to obtain control over the kind of information scraped, and determine if Tripadvisor and Google Maps reviews do show the same pattern in the type of reviews scraped. Restaurants in Tripadvisor are sorted by best to worst. To obtain a balanced sample, one restaurant is scraped from each page of 30 restaurants. For London This resulted in 46.000 reviews. The amount of reviews by restaurant is limited to 300. The scraper itself is hosted by a controlled Chrome driver from Python witht the package "Selenium", but supported by "BeautifulSoup" and "requests". Selenium can execute several interactive tasks with an website, like a human would, to then obtain specific pieces of information. Each second restaurant from the 30 list is taken, because the first and last are often adds. Only English languager reviews are scraped. The notebook shows an example from Liverpool, England. Cuisines are grouped geographically, except for bars/pubs, categories may be slightly adjusted to different areas. Scraped information includes price range, title, review, rating, cuisine and restaurant.

**2. Exploratory Data Analysis (EDA)**
The Exploratory Data Analysis checks the completeness of the data, the distribution of stars, in which five-star ratings are overrepresented and reviews over time. Due to this imbalance the dataset is balanced, which gave better results than weighting the individual results to the rating they were in.

**3. Return distinctive tokens by rating**
Based on VADER sentiment this notebook gives the most distinctive terms at each star rating, this is useful to recognize the real value of certain words in usage. This can be done using term frequency inverse document frequency which includes relative occurrence (idf) of terms combined with their raw occurrence (tf). Only words that occur in the VADER lexicon are included. These 7000-word lexicon is based on a representative set of human interpetation, with their mean being the actual word sentiment on a scalem -4 to +4. These are normalized to -1 to +1. The tf-idf didn't work as great as in other cases, due to overrepresentation of tf over idf. Some words like "good" occurred a lot, which then got returned as the top result of 3, 4 and 5 star reviews. In the spirit of the tf-idf the function of relative occurrence is created, this gives the occurrence of a token used in a star review relative to the average over all stars multiplied by token count to prevent highly infrequent words to be on top. The relative scores get subtracted by 1, which lead to scores around 0, instead of around 1, which improves the power of more distinct tokens by star over token count. WordClouds can be seen in the notebooks. This analysis is also done for bigrams, which may add some context, but in which the count by bigram is much lower than the count by token reducing its predictive confidence. VADER scores of tokens occurring in the lexicon by bigram are averaged. At last the average score of the top 20 tokens is reviewed by rating, to determine the average rating differences between stars. This can be improved by actually weighting all tokens in a final score.

Results are
-0.26 (1-star), -0.1205 (2-star), 0.141 (3-star), 0.5015 (4-star) and 0.6275 (5-star) indicating a higher use of positive sentiment tokens over negative sentiment tokens. For the bigrams these are -0.3230 (1-star), -0.115 (2-star), 0.276 (3-star), 0.4865 (4-star) and 0.5135 (5-star). Notable changes from tokens to bigram scores are 3-star (upwards) and 5-star (downwards).

The bigrams actually add some human-interpetable context. The top bigrams can be interpretated as star-descriptions ("the worst" - 1-star, "was ok" - 2-star, "was good" 3-star, "very good" - 4-star and "the best" - 5-star). But the VADER analyis lacks some focus in restaurant review specific token valuation i.e. "highly recommended" is valued at 0.27 (scale -1 to +1) VADER-score, while "was nice" stands at 0.42.

**4. Review sentiment model**
Predicting stars from a VADER-sentiment seemed difficult. Our assumption is that people's feelings may be contextualized well, but they find it hard to rate their average feelings to a certain star score. This is shown in the pairplot and the star distributions regarding the four forms of sentiment returned by VADER: compound, positive, negative and neutral sentiment. This led us to use the VADER-sentiment scores as predictor (Y) variable and the review texts as predictive (X). We determined the best predictive VADER-sentiment variable to be positive sentiment, which means the restaurant seniment score is most dependent on the amount of positivity in the restaurant review, thus a neutral review can be interpreted as negative. To improve the predictive power terms under 25 review occurrences and text stemming is undertaken. Both measures densify the vectorized predictor array and dampens the potential hinder of curse of dimensionality which is a result of sparse matrices. In addition different models are compared: polynomial regression, linear regression and support vector machine and optima are found for used ngrams (1), text_size (0.1) and tf-idf minimum review occurrences (25). With more data these optima can change i.e. reducement in optimal text size and eventual use of bigrams. To create stability in the outcomes of a random train text split, ten iterations are averaged for the R2 and proportional RSME metrics used. The Polynomial regression has the best R2 of 0.833 with a proportional RSME (RSME porportional to the variable's mean) of 0.3319. At last the stemmed text is translated back to the most common original tokens related to the newly stemmed text and coupled with the VADER score of that word to determine the individual influence of a certain token. Influence if of average kind, not additive. But is shows the words which are the most influential on positive sentiment and which area the least. 

