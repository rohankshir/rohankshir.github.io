---
layout: post
title: Analyzing Twitter Part 1
---

## Why Twitter?

There is something fascinating about Twitter. It quivers and shakes. It sings and shouts. It wakes up and sleeps. Each tweet is like a breath of the shared consciousness. If the stock market is a way to track what people think of the market, Twitter is a way to track what people think is important. The restricted character limit encourages brevity and hence creativity. At the same time, the social engine churns out a steady stream of insipid, mundane chatter. I believe both forms of content, and the spectrum in between contain valuable insights.

For those of us interested in natural language processing (NLP), there's nothing else like it. A living book of the social consciousness, that grows in richness as each year goes by. There are innumerous questions of the uses of twitter data. A few are, can we use Twitter data to help predict elections? Can we determine important events by using a combination of statistical and NLP techniques to sift through information? Can we build language models that understand the latest slang? Are hashtags no longer a novelty, but a tool used to bind associative concepts repeatedly? Can associated hashtag sets produce some baseline representation of the world? The list goes on and on.

I've decided to write a multiple part post on understanding Twitter data, while hopefully garnering a few key insights along the way. I will post code in the simplest form necessary. 

## Data Acquisition

There are a number of twitter corpora with sentiment analysis available. Sanders Analytics has built up a [corpora](http://www.sananalytics.com/lab/twitter-sentiment/), thoughthe code that uses the Twitter API to retrieve the tweets is out of date. Stanford also has created another Twitter Sentiment corpus, cleverly built by searching for emoticons such as :( and :) to create a distantly labeled set of tweets, more can be found out [here](http://help.sentiment140.com/for-students). I decided to use the Stanford one, as it would prepare me to deal with data that isn't 'perfect', while allowing to build new and interesting corpora for a variety of applications. This data spans between April to June in the year 2009, which is old, but still interesting from a data analysis perspective. It also has 1.6 million rows (which is quite a lot of data), which has a balanced number of positive and negative samples. I've written a small script to extract only the useful parts of the corpus, with the ability to specify how many lines of data you want.


{% highlight python %}

# Utility: Sentiment Corpus Reader
# Purpose: A utility to read sentiment corpora in a csv format
# Data file format has 6 fields:
# 0 - the polarity of the tweet (0 = negative, 2 = neutral, 4 = positive)
# 1 - the id of the tweet (2087)
# 2 - the date of the tweet (Sat May 16 23:58:44 UTC 2009)
# 3 - the query (lyx). If there is no query, then this value is NO_QUERY.
# 4 - the user that tweeted (robotickilldozr)
# 5 - the text of the tweet (Lyx is cool)


import csv

# Given a file that adheres to the data format (above) return a list
# of tuples with the following information (sentiment score, tweet
# text)
def read_data(filename, numrows):
    training_data = []
    count = 0
    with open (filename, 'rb') as csvfile:
        corpus_reader = csv.reader(csvfile)
        for line in corpus_reader:
            if count == numrows:
                break;
            count += 1
            training_data.append(line[5]), int(line[0])))

    return training_data

{% endhighlight %}



