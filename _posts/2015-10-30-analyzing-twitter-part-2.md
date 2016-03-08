---
layout: post
title: Analyzing Twitter Part 2
---


In the [last post]({% post_url 2015-06-30-analyzing-twitter-part-1 %}), we motivated why Twitter is interesting and got started on acquiring a corpus of tweets. In this post, we'll be talking about getting acquainted with the data. Instead of looking at our data set as such a data set, we'll slice it in a couple of ways to become familiar with it, and understand what we're working with.

## tf-idf

`tf-idf` stands for term frequency-inverse document frequency. The definition, supplied by [Wikipedia](https://en.wikipedia.org/wiki/Tf%E2%80%93idf), is a measure of how important the word is to a document in a collection of documents. Words with a high term frequency like 'the' and 'of' appear very frequently, and are offset by the inverse document frequency, 

\begin{equation}
tfidf_{t,d} = tf_{t,d} * idf_{t,d}
\end{equation}

The most widely used measure of term frequency is the raw frequency of the term in the document. This is simplest, but other weight mechanisms include binary and log normalization. We will use raw frequency for now.

\begin{equation}
tf_{t,d} = f_{t,d}
\end{equation}

Inverse document frequency measures how important the word is in the entire set of documents. Similar to term frequency, there are several [weighting schemes](https://en.wikipedia.org/wiki/Tf%E2%80%93idf#Inverse_document_frequency_2) for inverse document frequency. Again, we will use the most commonly used one.

\begin{equation}
	idf_{t,d} = log \frac{N}{n_t}
\end{equation}

$N$ is the number of documents and $n_t$ is the total number of documents that contain the term. When you multiply $tf$ with $idf$, you offset the strength of the term in the document by the popularity of it in the set of documents. Pretty straightforward, right?

Let's run it on our Twitter corpus and see what we find. `skikit-learn` has a package, `sklearn.feature_extraction.text.TfidfVectorizer` that can help us compute `tf-idf` scores for a large set of tweets. In fact, it can be done in 4 lines.

{% highlight python %}
vectorizer = TfidfVectorizer(min_df=1)
X = vectorizer.fit_transform(tweets)
idf = vectorizer.idf_
terms_score  = zip(vectorizer.get_feature_names(), idf)
{% endhighlight %}


We end up with a list of tuples that have the terms and their respective scores in `terms_score`. Only computed on a couple thousand tweets, we have over 25 thousand term scores. To sift through the mess, I filtered out words that were non-english and randomly selected a few from each tf-idf bin. The larger the font-size, the higher the `tf-idf` score.

<br><br>  

<div id="vis3"></div>
<script type="text/javascript">

function parse(spec,div_id) {
  vg.parse.spec(spec, function(chart) { chart({el:div_id}).update(); });
}
spec_uri = "/public/tfidf_spec.json";
parse(spec_uri, "#vis3");

</script>


<br>

After a quick glance, you can see that the smaller words tend to be less informative, short in length, and morphologically similar. Words like *the*, *this* are determiners and pretty much used ubiquitously. The words *rainy* and *wondering* have larger font and are more interesting and meaningful. These words could possibly be used for more interesting models, like topic models or sentiment analysis.

At the same time, it's much harder to gauge word salience when the length of a tweet is constrained to 140 characters. This limit doesn't allow for much repetion of words, thereby drastically reducing the variation on the term frequency portion of `tf-idf`. Another limitation of tf-idf is the lack of describing relations between words. In the word cloud above, you can see that the words *was* and *like* are very small, and thus have low scores. What you can't see, without implicitly understanding English and some deduction, that the two words are used often in conjunction: *was like*. 



For the next post, we'll explore bigrams and different association measures. Cheers!



