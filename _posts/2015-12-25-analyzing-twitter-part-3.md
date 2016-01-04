---
layout: post
title: Analyzing Twitter Part 3
---


In the [last post]({% post_url 2015-10-30-analyzing-twitter-part-2 %}), we looked at one way to analyze a collection of documents, `tf-idf`. This weighting technique is extremely common in Information Retrieval applications, and it helpful in favoring discriminatory traits of a document over nondisciminatory ones such as 'Obama' vs. 'the'.

One issue encountered while performing `tf-idf` weighting on tweets is the short, constrained nature of tweets. This creates an upper limit on the Term Frequency, reducing the importance of that portion of the weighting scheme. Another issue is the gargantuan vocabulary size of Twitter users. New words are generated all the time, hashtags and handle references are thrown about, making the representation of Tweets in the form of `tf-idf` vectors problematic.

How do we gather more useful and interesting facets of [Netspeak](https://us.sagepub.com/en-us/nam/computer-mediated-communication/book225026) from our large Tweet corpous? 


## Bigrams

Viewing words in isolation in a document can be a simple and effective, but often falls victim to the problems outlined above. Computational linguists ameliorate these issues by observing at sequences of words. Bigrams are two words in sequence that occur in a document. More generally, an [`n-gram`](https://en.wikipedia.org/wiki/N-gram) is a sequence of *n* words in text. `n-gram` models are used in all types of language applications, such as word prediction, speech recognition, spelling correction, language identification, etc.

How do we count all the bigrams in a collection of tweets? We can aggregate all the tweets into a single sequence of words, while separating each tweet with a separator word, such as "STOP".

{% highlight python %}
# Use stop to separate tweets in the list of all the tweets
separator_word = "STOP"
tweet_words = [separator_word]

for tweet_tokens in tweets:
    tweet_words += tweet_tokens
    tweet_words.append(separator_word)
{% endhighlight %}


Subsequently, the goal is to count and score these bigrams to gain insights. The `ntlk` python package provides several association measures. We will explore two different schemes today.

### Raw Frequency

This association measure simply scores the bigrams of the corpus by their raw frequency. The most frequently occuring bigrams will be scored highest. Below are a list of highest scored bigrams according to this measure, with a code snippet for solidarity. A few examples like "thank you" and "good morning" show commonly used phrases that are commonly embedded in discourse. Other popular bigrams give the feeling of the emotional stream of conscious, with many references to the subject, such as "I feel", "I want", "I need", and "you have". Finally, we can see commonly used [collocations](https://en.wikipedia.org/wiki/Collocation), such as "ready for", "such as", "at least", "to go".

The raw frequency scoring technique gleans some general, conversational facets of the English language. Many of these can be instrumental in guaging sentiment, constituents, and discourse. 

<br><br>  

<div id="vis4"></div>
<script type="text/javascript">

function parse(spec,div_id) {
  vg.parse.spec(spec, function(chart) { chart({el:div_id}).update(); });
}
spec_uri = "/public/raw_frequency_wordcloud_spec.json";
parse(spec_uri, "#vis4");

</script>
<br>

{% highlight python %}
bigram_finder = BigramCollocationFinder.from_words(tweet_words)

print "Loaded bigram collocation finder."

# Filter out ngrams with separator
def is_separator_bigram(w1,w2):
    return  w1  == separator_word or w2 == separator_word
bigram_finder.apply_ngram_filter(is_separator_bigram)

print "Filtered separator bigrams."

# How many of the top scored bigrams by each association measure are
# we interested in
n = 100

bigrams_raw = bigram_finder.nbest(BigramAssocMeasures.raw_freq, n)

{% endhighlight %}


### Chi Squared

The Chi-squared test is an independence test to see if two words occuring together are more likely than random chance. The formula for Chi-Squared is as follows:
\begin{equation}
chisq_{x,y} = N * \phi^2 
\end{equation}

Where $\phi$ is essentially a normalized sum of squared deviations between observed and theoretical frequencies. The theoretical frequencies, or the expected value, is derived from the base probabilities of each word occuring in the text, while the observed come from the counts of the corresponding bigrams. Though this isn't hard to compute, it's conveniently packaged into nltk's Bigram Association Measure module. It's one line.

{% highlight python %}
bigrams_chisq = bigram_finder.nbest(BigramAssocMeasures.chi_sq, n)
{% endhighlight %}


Below is the resulting word cloud of the top bigrams scored by the chi-squared measure. This much more aptly captures the multifaceted composition of Twitter. Memes bubble up in the form of "pimp ship" and "grief porn", most of which are coined in the past few years to describe recent phenomena. There's hashtag marketing schemes like "#mediencrew #fotografie", and "#eu09 vergenza". Finally, there's more general collocations like "nice lady" and "official site". 

<br><br>  

<div id="vis5"></div>
<script type="text/javascript">

function parse(spec,div_id) {
  vg.parse.spec(spec, function(chart) { chart({el:div_id}).update(); });
}
spec_uri = "/public/chisq_bigram_wordcloud_spec.json";
parse(spec_uri, "#vis5");

</script>
<br>


As seen in the examples above, bigrams give a more nuanced perspective on language compared to tf-idf, but requires more computational power, and usually more data. We also see the difference in weighting formula, raw frequency vs. chi-squared, paints a completely different picture of our data. A combination of tf-idf, chi-squared, raw frequency, and other measures can help form a diverse, but vectorized representation of documents to build complex models for higher-level applications.

In my next post, I'll be delving into topic modeling on music lyrics. In particular, we'll explore hip-hop music and Latent Dirichlet Allocation.

