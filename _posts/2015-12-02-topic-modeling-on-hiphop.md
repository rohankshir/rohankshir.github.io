---
layout: post
title: Topic modeling on Hip Hop Music
---

Music has definitely changed over the years. New artists rise to prominence. Beats are produced by more advanced technology. And lyrical content has transformed. As an engineer who focuses in Natural Language Processing, I wanted to make an attempt to understand how the lyrics have changed over the years from a bird's eye view. Though word choice and line syntax are very interesting, I am most interested in the topics of the most popular rap music and analyzing it as a reflection of the popular culture, values, and desires. The reason I chose rap music is because of it's alleged focus on the lyrics. 


### Topic Modeling
How do we analyze a collection of song lyrics for their topics? [Topic models](https://en.wikipedia.org/wiki/Topic_model) is a type of statistical model that discovers abstract topics that occur in a collection of documents. There's been a lot of talk about Blei and Jordan's unsupervised method of topic modeling, [Latent Dirichlet Allocation](https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation) Pretty much it comes up with a mixture of topics, groups of words, that each document in your corpus is composed of. It's unsupervised generative model that has been used in many applications.


### The Billboards
While there is an enormous amount of production going on in Hip Hop, I will only be analyzing the most popular music of each year. The rise in popularity could be due to a complex nonlinear function of society, but no doubt the popularity causes it to be played more times, and thus have more people listen to it. It's the rise of popularity along with the continuity that I'll be analyzing at first. What I'm analyzing is analagous to the following question. 

##### If an alien society were to analyze us through our popular rap music, what would they find?

If we can view the lyrics as a narrow but not insignificant snapshot of society, there's a reason to take this analysis seriously. With that I present to you my work and my findings. In a hurry? Skip to the [results](#results). So how do we get these songs automatically?

#### Song Acquisition

The [Billboard charts](http://www.billboard.com/charts) have been around for as long as I can remember. After some Googling, I happened upon Github user [guoguo12's](https://github.com/guoguo12) python [API](https://github.com/guoguo12/billboard-charts) for accessing these charts programmatically, and decided to check it out.

`pip install billboard.py`

After playing around with it for a bit in interactive python, I realized that it didn't always fetch the charts for a given day, but if called the API for the next 6 days, I'd end up getting a hit. This signaled to me, that Billboards organized their data on a weekly basis, not a daily basis.

I wrote some scripts that do a little bit of date manipulation in concordance with guoguo12's billboard API to get the most popular songs for any given year, parametrized by week or month. Feel free to steal it and use it on your own :)

{% highlight python %}
import billboard
from datetime import date
from datetime import timedelta

def get_dates_by_month(year):
    ret = []
    d = date(year = year, month = 1, day = 1)
    ret.append(d)
    while d.month < 12:
        d = d.replace(month = d.month + 1)
        ret.append(d)
    return ret


def get_dates_by_week(year):
    ret = []
    d = date(year = year, month = 1, day = 1)
    delta = timedelta(days = 7)
    while d.year == year:
        ret.append(d)
        d = d + delta
    return ret

def get_chart_entries(playlist, date):
    chart = billboard.ChartData(playlist, str(date))
    delta = timedelta(days = 1)
    total_delta = timedelta(days = 0)
    while len(chart.entries) == 0:
        total_delta += delta
        chart = billboard.ChartData(playlist, str(date + total_delta))
    return (chart, total_delta)


def get_charts(playlist, dates):
    ret = []
    delta = timedelta(days = 0)
    for d in dates:
        chart, delta = get_chart_entries(playlist, d + delta)
        ret.append(chart)
    return ret

def get_n_most_frequent_entries(charts, n):
    d = {}
    for chart in charts:
        for song in chart.entries:
            if song.title not in d:
                d[song.title] = 1
            else:
                d[song.title] += 1

    l = [(k,v) for k,v in d.items()]
    l.sort(key=lambda x: x[1])
    l.reverse()
    
    return [title for title,freq in l[:n]]
        
charts = get_charts('r-b-hip-hop-songs', get_dates_by_week(2010))

top_songs = get_n_most_frequent_entries(charts, 30)

for song in top_songs:
    print song

{% endhighlight %}

By the end I could get the *n* most popular songs of any given year, along with the artist information. Now that we could get the song titles, the next step was to 

#### Lyric Acquisition
I scoured the webs for a database for rap music lyrics only to find what seems to be a fizzled-out project, [Hip Hop Word Count](http://www.fastcompany.com/3007753/hip-hop-word-count-living-breathing-database-every-word-every-rap-song-ever). Not sure what happened to project, but I spent a lot of time Googling only to be left empty handed. Frustrating.

So what else? There's [Genius](https://en.wikipedia.org/wiki/Genius_(website)), an online knowledge base of music with lyrics and annotations. It had an initial focus on rap music so it was perfect for my needs. 

## <a name="results"></a> Results

These are the results of my analysis.