

```python
#Dependencies
import tweepy
import json
import matplotlib.pyplot as plt
import seaborn
import pandas as pd
import numpy as np
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()# Target User Accounts
```


```python
#API Keys
consumer_secret = "b64YDRVqWVuyLXwLjzrjeUpTbtm1zXanc1yKg0ZTR6OUTOLIQN"
consumer_key = "qWuvZrFXqi20Uds5sNVjt86su"
access_token = "17548644-XE85M1wzAN7SWitsHEA3KVD7whnPDy9gi1tpA0pUb"
access_token_secret = "lTd5jVCOTOV4PCufOdevcY8JSoWeVjTzT8xzxh9Y1IW3W"
```


```python
# Setup Tweepy API Authentication
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth) 
```


```python
# Target User Accounts
target_user = ("@BBCWorld", "@CBSNews", "@CNN","@FoxNews","@NYTimes")
```


```python
#Creation of dataframe and empty dictionary for scores
user_scores_dict = {}
tweet_data_df = pd.DataFrame(columns = ['User','Tweet Data','Date','Compound Score',
                                        'Positive Score','Neutral Score','Negative Score', 'Tweets Ago'])


x = 100        #Counts tweets
for user in target_user:

    # Variables for holding sentiments
    compound_list = []
    positive_list = []
    negative_list = []
    neutral_list = []
    x = 100
    
    # Loop through 100 tweets
    for status in tweepy.Cursor(api.user_timeline, id=user).items(100):

        # Get all tweets from home feed
        
        process_status = status
        tweet = json.dumps(process_status._json, indent=3)
        tweet = json.loads(tweet)
        text = tweet['text']
     
        # Run Vader Analysis on each tweet
        compound = analyzer.polarity_scores(text)["compound"]
        pos = analyzer.polarity_scores(text)["pos"]
        neu = analyzer.polarity_scores(text)["neu"]
        neg = analyzer.polarity_scores(text)["neg"]
        
        #add to dataframe
        tweet_data_df = tweet_data_df.append(({'User':tweet['user']['name'],'Tweet Data':tweet['text'],
                                                                'Date':tweet['created_at'],'Compound Score': compound,
                                                                'Positive Score': pos, 'Neutral Score':neu, 'Negative Score':neg,
                                              'Tweets Ago': x}), ignore_index = True) 
        
        x -= 1   #decrease tweet counter
        
```


```python
#Saves Tweet Data to a CSV File

tweet_data_df.to_csv('Tweet_Data.csv')
```


```python
#creates the scatter plot for each news site, pulls by index in the dataframe
bbc_plot = plt.scatter(tweet_data_df[0:99]['Tweets Ago'], tweet_data_df[0:99]['Compound Score'], c = 'lightblue')
cbs_plot = plt.scatter(tweet_data_df[100:199]['Tweets Ago'], tweet_data_df[100:199]['Compound Score'], c = 'forestgreen' )
cnn_plot = plt.scatter(tweet_data_df[200:299]['Tweets Ago'], tweet_data_df[200:299]['Compound Score'], c = 'red')
fox_plot = plt.scatter(tweet_data_df[300:399]['Tweets Ago'], tweet_data_df[300:399]['Compound Score'], c = 'mediumblue')
nyt_plot = plt.scatter(tweet_data_df[400:499]['Tweets Ago'], tweet_data_df[400:499]['Compound Score'], c = 'yellow')

#styling
plt.style.use('seaborn-dark')
plt.grid()
plt.xlim(100,0)
plt.xlabel('Tweets Ago')
plt.ylabel('Tweet Polarity')
plt.title('Sentiment Analysis of Media Tweets (12/17/17)')

#makes a legend
plt.legend(['BBC','CBS','CNN','Fox','New York Times'], title = 'Media Sources',bbox_to_anchor=(1.05, 1))

#Saves figure
plt.savefig('sentiment_analysis.png')
plt.show()
```


![png](output_6_0.png)



```python
#calculates average compound score for each news organization
bbc_avg = tweet_data_df[0:99]['Compound Score'].mean()
cbs_avg = tweet_data_df[100:199]['Compound Score'].mean()
cnn_avg = tweet_data_df[200:299]['Compound Score'].mean()
fox_avg = tweet_data_df[300:399]['Compound Score'].mean()
nyt_avg = tweet_data_df[400:499]['Compound Score'].mean()

#creates list of averages and list of news names
avg_list = [bbc_avg, cbs_avg, cnn_avg, fox_avg, nyt_avg]
news_list = ['BBC','CBS','CNN','Fox','New York Times']

#Creates bar graph
plt.bar(news_list, avg_list, color = ['lightblue','forestgreen','red','mediumblue','yellow'], width = 1.0,
        align = 'center', fill = True)

#Styling
plt.ylabel('Tweet Polarity')
plt.title('Sentiment Analysis of Media Tweets (12/17/17)')

#Saves figure
plt.savefig('sentiment_analysis_avg.png')

plt.show()
```


![png](output_7_0.png)

