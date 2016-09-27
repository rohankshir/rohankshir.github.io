---
layout: post
title: Getting comfortable with the Unix pipe
---

## Files ain't going away anytime soon, son. 

If you're a developer, engineer, technical person who's expected to know something about coding, you're gonna be working with files. You might as well learn how to get glean some information from it without busting open excel, lest you want to be mistaken for a business person. 

## Pipe it like it's hot

The [Unix pipe](http://www.westwind.com/reference/os-x/commandline/pipes.html) is the weapon I use to get a quick picture of what's going on in a file, whether it's a large data file [if you're into *Big Data*, you can leave] that I need to read in, or it's some output that my system has created. The weapon is really plumbing between different unix commands that read from stdin and output to stdout. Unfortunately you can't hurt anyone with this pipe. 

## Examples are good, I hear

Download this [impatient_data.csv](/public/impatient_data.csv) that I got from [data.gov](https://www.data.gov/health/) if you want to follow along. We're gonna do some cool stuff with it. This is basically data that describes each patient's diagnosis, along with the hospital discharge data -- most importantly, the location and the **money**.

<iframe src="//giphy.com/embed/KJg6Znn4V1Jcs" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/KJg6Znn4V1Jcs"></a></p>

Here are the columns of the csv file, brought to you by `head`:

{% highlight bash %}
bash-3.2$ head -n 1 impatient_data.csv
DRG Definition,Provider Id,Provider Name,Provider Street Address,Provider City,Provider State,Provider Zip Code,Hospital Referral Region Description, Total Discharges , Average Covered Charges , Average Total Payments ,Average Medicare Payments

{% endhighlight %}


Let's see how many lines are in the file. 


{% highlight bash %}

bash-3.2$ wc -l impatient_data.csv
  163066 impatient_data.csv
{% endhighlight %}

Ha! 163,000 lines! Good luck doing anything in Excel. 

Let's say I want to know how many of these charges there are in Jersey, my home state, the best state. And then I want to know what are the unique diagnoses DRG Definition) in Jersey. 

{% highlight bash %}
bash-3.2$ grep ',NJ,' impatient_data.csv  | wc -l
    4826
	
bash-3.2$ grep ',NJ,' impatient_data.csv  | cut -d',' -f 1 | sort | uniq
"280 - ACUTE MYOCARDIAL INFARCTION
"281 - ACUTE MYOCARDIAL INFARCTION
"282 - ACUTE MYOCARDIAL INFARCTION
"286 - CIRCULATORY DISORDERS EXCEPT AMI
"287 - CIRCULATORY DISORDERS EXCEPT AMI
"391 - ESOPHAGITIS
"392 - ESOPHAGITIS
"563 - FX
....
97 - ALCOHOL/DRUG ABUSE OR DEPENDENCE W/O REHABILITATION THERAPY W/O MCC
917 - POISONING & TOXIC EFFECTS OF DRUGS W MCC
918 - POISONING & TOXIC EFFECTS OF DRUGS W/O MCC
948 - SIGNS & SYMPTOMS W/O MCC
bash-3.2$
{% endhighlight %}


## Whoa ho ho there! Lemme break that down 

`grep ',NJ,' impatient_data.csv`

I used `grep` to filter for the charges in New Jersey. In fact, I padded 'NJ' with commas in my command to filter out any extraneous 'NJ's that showed up in other columns. There's about 5000 data points from the Dirty Jersey.

`cut -d',' -f 1 `

I used `cut` to cut out a single column or field from the output of the `grep` command. Specifically, the first field of the output, which is the diagnosis description. 

` sort | uniq `

Well, I wanted a list of all the unique charges that happened so I'm gonna use `uniq` obviously. But first you have to sort.

So together that becomes:

`grep ',NJ,' impatient_data.csv  | cut -d',' -f 1 | sort | uniq`

This gives you all the unique Diagnosis-related groups (DRG). 

### Pro tip: use `less` to verify the output of your piped commands

What if we want to figure out the top 10 most frequently occuring Diagnoses in Joisey? Well, that's easy. 

{% highlight bash %}
$ grep ',NJ,' impatient_data.csv | cut -d',' -f 1 | cut -d'-' -f 2 | sort | uniq -c  | sort -nr | head -n 10
 166  ACUTE MYOCARDIAL INFARCTION
 119  MISC DISORDERS OF NUTRITION
 116  ESOPHAGITIS
  93  CIRCULATORY DISORDERS EXCEPT AMI
  64  SIMPLE PNEUMONIA & PLEURISY W CC
  64  SEPTICEMIA OR SEVERE SEPSIS W/O MV 96+ HOURS W MCC
  64  HEART FAILURE & SHOCK W MCC
  63  SYNCOPE & COLLAPSE
  63  KIDNEY & URINARY TRACT INFECTIONS W/O MCC
  63  HEART FAILURE & SHOCK W/O CC/MCC
{% endhighlight %}

Interesting... Acute myocardial infarctions happen a LOT. I've never heard of them, but a quick Google search shows that they're in fact Heart Attacks. What's more surprising are all these nutrition disorders. Hmph. I should really eat my vegetables. Let's move on to a extremely personal question of mine..

## Should I retire in Florida or not?

As a 25 year old man, this question begs to be answered. Let's find the average Total costs of hospital charges in New York and Florida. 

{% highlight bash %}
+Rohans-MacBook-Pro:public rohan$ grep ',NY,' impatient_data.csv  | cut -d',' -f 11 | cut -c 2-| awk '{ sum += $1; n++ } END { if (n > 0) print sum / n; }'
+13567.2
+Rohans-MacBook-Pro:public rohan$ grep ',FL,' impatient_data.csv  | cut -d',' -f 11 | cut -c 2-| awk '{ sum += $1; n++ } END { if (n > 0) print sum / n; }'
+11716.5
{% endhighlight %}


Well, I save an average of $1800 on Heart Failure related conditions, but then again, I'll be in Florida. That's a tough one. 

I hope you learned as much about Unix piping as I did from this Health data set. Thanks Obama.
<iframe src="//giphy.com/embed/MeWF1j3gPBL44" width="480" height="356" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/thanks-obama-gif-infomercial-fail-MeWF1j3gPBL44"></a></p>
