---
layout: post
title: NLP on Hip Hop Music Part 1
---

<link rel="stylesheet" href="{{ site.baseurl }}public/css/topic-modeling.css">

# How has Hip Hop changed over the years?

It's a challenging question one that is subject to many viewpoints and analyses. Music has changed so much over the years in so many different ways. New artists continue to spring to prominence. Production technology evolves rapidly, allowing for more iterations and interaction of different genres. The internet allows for more collaboration. A small but interesting part of this change lies in how language is imbibed within Hip Hop music. 


<div id="wrapper" onclick="pauseFade()" >
<a name="Words"></a>
<div id="apDiv4" class="fadeA currentA" >
<div id="vis12"></div>
<center>Unique words from 2000</center>
</div>

<div id="apDiv5" class="fadeA" style="display:none;" >
<div id="vis10"></div>
<center>Words common from 2000 and 2014</center>
</div>

<div id="apDiv6" class="fadeA" style="display:none;">
<div id="vis11" ></div>
<center>Unique words from 2014</center>
</div>

</div>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.0/jquery.min.js"></script>
<script>
var divsA = $('.fadeA');
var shouldFade = true;
function parse(spec,div_id) {
  vg.parse.spec(spec, function(chart) { chart({el:div_id}).update(); });
}
spec_uri1 = "/public/2000minus2014_spec.json";
parse(spec_uri1, "#vis12");
spec_uri2 = "/public/2000and2014_spec.json";
parse(spec_uri2, "#vis10");
spec_uri3 = "/public/2014minus2000_spec.json";
parse(spec_uri3, "#vis11");

function fade2() {
	if (!shouldFade)
	{
		return;
	}
	
    var current = $('.currentA');
    var currentIndex = divsA.index(current),
        nextIndex = currentIndex + 1;
    
    if (nextIndex >= divsA.length) {
        nextIndex = 0;
    }
    
    var next = divsA.eq(nextIndex);

    current.stop().fadeOut(1000, function() {
        $(this).removeClass('currentA');
	    setTimeout(fade2, 3500);
    });

    
    next.stop().fadeIn(1000, function() {
        $(this).addClass('currentA');
    });
}

fade2();

function pauseFade() {
	if (shouldFade) {
		shouldFade = false
	}
	else {
		shouldFade = true;
		fade2();
	}
}


</script>

I wanted to focus on answering a few questions:

* What are the core themes of Hip Hop lyrics in any given year?
* How have these themes evolved over the years?
* On a lower level, which sort of words and syntax weave together Hip Hop music?

Using NLP to process these documents allows us to statistically model these problems and hopefully glean something interesting. One technique that comes to mind is statistical topic modeling. 

## Topic Modeling
Topic modeling algorithms statistically analyze the words of the original texts to discover the themes that run through them. The key idea is to first come up with a model that allows to generate a collection of documents given a distribution of topics, and a distribution of words over each topic. Given the distribution, we can sample topics and words from this topics to build a document up, implying that each document is a mixture of topics.  However, we don't start out with the distributions, we only start out with the documents, songs in our case. We can use the observed documents to reverse the generative process and infer the hidden or latent structure, which are these topic and word distributions. 

Why is this useful? As the amount of texts in our world grows exponentially, humans do not have the capacity to hand annotate all of the key topics of every document. Determining these topics can be very useful for search, discovery, categorization, and other downstream analysis. It gives us a way to dive into a large collection of documents by drilling down by topic.

While in practice it has shown enormous value in categorizing research publications and news, there's been much less work in the music domain. Specifically hip-hop music. Hip Hop music is a huge genre. How can we reduce the scope of our analysis to a manageable size, both for processing time and interpretability?

## The Billboards
While there is an enormous amount of production going on in Hip Hop, I will only be analyzing the most popular music of each year. Popularity is as much a reflection of culture as it is the genre itself. This follows the biggest bang for your buck philosophy.

I'll walk through in detail how I accomplished each step and reveal some source code. Not interested? [Skip to the results](#Results)

# Implementation

Before we can run our NLP analysis, we need to first build a corpus of song lyrics.

## Title and Artist Acquisition

The [Billboard charts](http://www.billboard.com/charts) have been around for as long as I can remember. After some Googling, I happened upon Github user [guoguo12's](https://github.com/guoguo12) python [API](https://github.com/guoguo12/billboard-charts) for accessing these charts programmatically, and decided to check it out.

`pip install billboard.py`

After playing around with it for a bit in interactive python, I realized that it didn't always fetch the charts for a given day, but if called the API for the next 6 days, I'd end up getting a hit. This signaled to me, that Billboards organized their data on a weekly basis, not a daily basis.

I wrote some scripts that do a little bit of date manipulation in concordance with guoguo12's billboard API to get the most popular songs for any given year, parametrized by week or month. Feel free to steal [it](https://github.com/rohankshir/lyrics_scraper/blob/master/billboards.py) and use it on your own :)

{% highlight python %}
args = parser.parse_args()
    
charts = get_charts(args.chart, get_dates_by_month(args.year))
	
top_songs = get_n_most_frequent_entries(charts, args.number)
    
for song in top_songs:
    print song


{% endhighlight %}

By the end I could get the `n` most popular songs of any given year, along with the artist information. Now that we could get the song titles, the next step was to scrape the lyrics for each of these songs. 

##  Lyric Acquisition
I scoured the webs for a database for rap music lyrics only to find what seems to be a fizzled-out project, [Hip Hop Word Count](http://www.fastcompany.com/3007753/hip-hop-word-count-living-breathing-database-every-word-every-rap-song-ever). Not sure what happened to project, but I spent a lot of time Googling only to be left empty handed. Frustrating.

So what else? There's [Genius](https://en.wikipedia.org/wiki/Genius_(website)), an online knowledge base of music with lyrics and annotations. It had an initial focus on rap music so it was perfect for my needs. 

Using the titles and artists from the `billboards` script, we can query the Genius API to get the URL of the source for each song's lyrics. Following that we can scrape the lyrics from the html by using a little bit of `Nokogiri` and `xpath`.

{% highlight ruby %}
songs = Genius::Song.search(keywords) # Returns an array of Song objects
song = get_song(songs, keywords) # get the most likely Song from the results using simstring

if song.nil?
  puts "ERROR: no song found. Either too ambiguous or no results"
  exit
end
puts song.url

doc = Nokogiri::HTML(open(song.url))

file = File.open(filepath, 'w')
doc.xpath("//*[contains(@class, 'lyrics_')]").each do |node|
   file.write( node.text )
end

{% endhighlight %}

You can find the full source [here](https://github.com/rohankshir/lyrics_scraper), that includes a bash script, `create_corpus.sh`, that downloads an entire corpus of song lyrics for any Billboards genre for a range of years on to disk. Note that it depends on `gnu parallel` for efficiency purposes, but you can uncomment the simpler slower while loop if you're sadly unable to install `gnu parallel`.

##  Topic Modeling

This was by far the most challenging part of the work I did. Using probabilistic LDA to infer topic and word distributions didn't work out of the box. One of the challenges was recognizing the multiple layers of assumptions that went into using this. Does the generative process properly describe the lyrics in hip-hop music? While topic modeling algorithms have been shown to work very well on news articles and research papers, music is not as simple.

Hip-hop lives in the land of in metaphors, stories, and repetition. Consider the two 2015 hits "The Hills" by The Weeknd and "Hotline Bling" by Drake. Both of these songs describe old flames of these rappers, both addressing them indirectly through topics such as calling, drugs, and love. The words do not directly reflect the topic of the song. The fundamental assumption of the generative process is a known flaw, but I decided to try it out anyway. What else am I gonna do with my weekends?

Beyond this assumption lie the research constraints and assumptions of LDA: 1) the bag of words assumption, 2) order of documents doesn't matter, and 3) a fixed number of topics. This becomes a matter of trial and error - how many topics do we use? How many songs do we need to analyze? What time range do we bundle songs together for analysis?

From the pre-processing side, there is a lot of cleaning up to do before getting anything that looks remotely useful. Some of these steps include `tf-idf`, stemming, stop word filtering, case normalization. You can find everything in the github source. I used the [`lda`](https://pypi.python.org/pypi/lda) package available via `pip`. `sklearn`, `nltk`, and standard python modules.

The final script takes in the lyrics corpus, a range of years, and runs topic-modeling for each year's song lyrics, finally spitting out the top k topic-word lists in each iteration. These results are then aggregated for the visualizations outlined below. I had finally settled on 5 to 8 topics on a 100 document per year corpus after many experiments. Ideally, this analysis would be done on tens of thousands of songs. Hopefully, someone can take this work and run with it.

I've included a few snippets of the code here. I'll update my github with the full source if anyone is interested - please email me if so.

{% highlight python %}
# topic_modeling.py
def lda_lda(docs, num_topics, num_iters, n_top_words, vectorizer):
    vec = vectorizer(tokenizer=word_tokenize,
                          stop_words='english',
                          preprocessor=document.preprocess_word,
                          lowercase=True,
                     token_pattern='(?u)\b\w\w\w\w+\b')
    pprint.pprint(docs)
    data = vec.fit_transform(docs)
    data *= 100
    data = data.astype(int)

    vocab = vec.get_feature_names()

    model = lda.LDA(n_topics=num_topics, n_iter=num_iters, random_state=1)
    model.fit(data)  # model.fit_transform(X) is also available
    topic_word = model.topic_word_  # model.components_ also works

    ret = []

    print model.nz_	
    for i, topic_dist in enumerate(topic_word):
        topic_words = np.array(vocab)[np.argsort(topic_dist)][:-(n_top_words+1):-1]

        ret.append(u'Topic {}: {}'.format(i, u' '.join(topic_words)).encode('utf-8').strip())

    return ret


# document.py
def preprocess_word(s):
    exclude = set(string.punctuation)
    s = s.decode("ascii","ignore")
    s = ''.join(ch for ch in s.lower() if ch not in exclude)
    s = s.lower()
    try:
        s = stemmer.stem(s)
    except:
        pass
    return clean_word(s)
        
{% endhighlight %}

# Results <a name="Results"></a>

<div id="wrapper">
<div id="apDiv1" class="fade current" style="display:none;">
<div id="vis6"></div>
<center>Topic words 2000 to 2005</center>
</div>

<div id="apDiv2" class="fade" style="display:none;">
<div id="vis7"></div>
<center>Topic words for 2005 to 2009</center>
</div>

<div id="apDiv3" class="fade" style="display:none;">
<div id="vis8" ></div>
<center>Topic Words for 2010 to 2014</center>
</div>

</div>

<script>
var divs = $('.fade');

function parse(spec,div_id) {
  vg.parse.spec(spec, function(chart) { chart({el:div_id}).update(); });
}
spec_uri1 = "/public/hiphop_topics_vega_spec.json";
parse(spec_uri1, "#vis6");
spec_uri2 = "/public/hiphop_2006-2010_vega_spec.json";
parse(spec_uri2, "#vis7");
spec_uri3 = "/public/hiphop_2010-2014_vega_spec.json";
parse(spec_uri3, "#vis8");

function fade() {
    var current = $('.current');
    var currentIndex = divs.index(current),
        nextIndex = currentIndex + 1;
    
    if (nextIndex >= divs.length) {
        nextIndex = 0;
    }
    
    var next = divs.eq(nextIndex);

    current.stop().fadeOut(2000, function() {
        $(this).removeClass('current');
        setTimeout(fade, 2500);
    });

    
    next.stop().fadeIn(2000, function() {
        $(this).addClass('current');
    });
    
}

fade();
</script>

The larger the word, the more important the topic word is. Some of the topics from from 2000 to 2005 include 'dance', 'party', 'love', and 'tonight'. From 2006 to 2009, a few of the topics include 'money', 'bitches/shawties', 'feel', and 'love'. From 2010 to 2015, some of the major topics include 'fuck', 'bitches', 'girls', 'think', and 'love'.

### Love is a common thread throughout hip-hop music and perhaps all music

Not the most surprising find, but still worth noting how love is talked about. In 2001, the words related to the love topic include: `'girl', 'always' 'goodbye' 'baby' 'music' 'suppose' 'thing' hurt' 'emotion' 'girl' 'sometime' baby' 'thought' 'work' 'stroke' 'lifetime' 'freak'`. In 2014, related words include:  `'rock' 'whoa' 'light' 'feel' 'clap' 'pussy' 'potion' 'girl' 'dance' 'hold' 'drunk' 'song' 'party' 'throw' 'wiggle' 'shake'`.

This recontextualizes love from the romantic viewpoint to the carnal hedonistic one. Though the earlier years of hiphop aren't vindicated from wrongdoing altogether, love was still held in a positive virtuous light - and the lyrics focused more on devotion and dedication to love. Now we "love" when that "pussy clap" and that booty "wiggle".

Since this is taken from the top 100 Billboard charts, this is just a reflection of society as much as Hip-hop. I wouldn't be surprised if this trend was seen across genres, not only in lyrics, but music videos as well. 

### The prominence of Fuck

Another thing I've noticed is the rise and sustained growth of Fuck, the latest swiss army knife of an expletive, carving up top hits by the dozen. It can be used a a substitude for sex, kill, assault, insult, a unit of concern, and everything in between. I've had a hard time analyzing it's increased use in every day vernacular because I wasn't able to distinguish whether it was me just growing up or society actually used it more often. This provides some evidence for the latter. 


## Further Research

While topic modeling didn't work out as ideally as initially expected, it yielded some good insights that lend themselves to further exploration. Namely, vernacular, word choice and meaning, and how it changes with [time](#Words). In our next post, we'll dive deeper into that. Hope you enjoyed. Cheers!
