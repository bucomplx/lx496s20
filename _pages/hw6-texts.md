---
layout: single
title: Texts
permalink: "/hw6-texts/"
---

<div class="title-table clear-table center-table" markdown="1">

| :--------    | :--:        | -:           |
| CAS LX 394 / | NLP/CL      | Homework 6   |
| GRS LX 694   | Spring 2019 | Due FRI 4/5 |

</div>


### NOTE THE DUE DATE ###

Because it took me all the way until Friday to get this posted,
it is not super necessary to have it done by class time on Monday.
I've set the due date to Friday next week.  Though as usual I
don't really think it'll necessarily take all that long.

That's not to say that I won't assign something new on Monday, there
may well be some overlap between assignments.


### Characterizing and searching texts ###

A lot of this homework is based on things from chapter 3, things
we hadn't gotten to until just recently.  So part of this is just
intended to help build confidence in being able to work on your own
things.

We didn't quite get to the things I'd intended to, so this isn't covering
a huge amount of new ground, but still not bad just for the exercise. 

Let's start by getting a corpus.

### Right Ho, Jeeves ###

So that we're working on the same thing, let us look at
P.G. Wodehouse's *Right Ho, Jeeves.*

So the first step is to go to
[Project Gutenberg](https://www.gutenberg.org)
and find it.  This should be straightfoward, but
I'll let you do it rather than just providing the link.
Builds character.

What you want is the URL (web address) of the
"Plain-text UTF-8" version.  Probably the simplest way to do
that is just to navigate to it on the Project Gutenberg site and
then copy the URL from the address bar.

We'll be comparing both *My Man Jeeves* and *Right Ho, Jeeves*.
I'll provide versions of a few of these commands for *My Man Jeeves*,
and then you can do the same things but for *Right Ho, Jeeves*.
Mostly I will try to prefix the names for things that pertain
to *My Man Jeeves* with `mmj_` and the names for things that pertain
to *Right Ho, Jeeves* with`rhj_`.

I put what I got for *My Man Jeeves* into `mmj_url`.
Your first task is to find `rhj_url`.

```python
>>> mmj_url = 'http://www.gutenberg.org/cache/epub/8164/pg8164.txt'
```

<!-- it will in fact be
http://www.gutenberg.org/cache/epub/10554/pg10554.txt
If you managed to read this in the source for this web page
then, fine, you win hacker points anyway.
-->

Now, you follow the procedure from class (and from the book)
to bring it in from the internet.  The code below will fetch
*My Man Jeeves* and place its text in `mmj_raw`.
Do that, and then (your part) also fetch
*Right Ho, Jeeves* so that `rhj_raw` holds this other text.

```python
from urllib import request
mmj_url = "http://www.gutenberg.org/cache/epub/8164/pg8164.txt"
mmj_response = request.urlopen(mmj_url)
mmj_raw = mmj_response.read().decode('utf8')
```

Now, we need to chop out just the text.
That is, we want to get rid of the stuff that Project Gutenberg
added, which leads up to the actual text, and then the stuff
at the end, after the text.
Figure out where the text starts and end by just looking
through the file in the web browser.

Below is how
I would do it for *My Man Jeeves*, which leaves just the text
in `mmj_trimmed`.
Do that, and then do it
for *Right Ho, Jeeves*, so that you have just the text of
*Right Ho, Jeeves* in `rhj_trimmed`.

```python
mmj_start = mmj_raw.find("MY MAN JEEVES")
mmj_end = mmj_raw.rfind("End of the Project")
mmj_trimmed = mmj_raw[mmj_start: mmj_end]
```

And then tokenize the books into words.
Below is how I'll tokenize *My Man Jeeves* into `mmj_tokens`.
Do that, and then tokenize *Right Ho, Jeeves* into `rhj_tokens`.

```python
import nltk
mmj_tokens = nltk.word_tokenize(mmj_trimmed)
```

If we turn these into texts, we can look at the collocations.
Do the following for *My Man Jeeves* and then do the same
for *Right Ho, Jeeves*.  As expected, we're mostly getting names.

```python
mmj_text = nltk.text.Text(mmj_tokens)
mmj_text.collocations()
```

### Part of speech tags ###

Let's take a look at the character of the writing in these books.
They feel kind of flowery, adjective-filled.
Let's see if that sense is justified.
We have not worked with part of speech tagging much yet, but let's do it here anyway.  

NLTK has a built in tagger (which assigns words to syntactic categories).
We know it's not going to be perfect.
It may not even be all that good, but it'll do to a first approximation.
It's very easy to use.  Just feed it tokens, and it will return the tokens with tags.

```python
mmj_tagged = nltk.pos_tag(mmj_tokens)
```

As before, do the tagging above for *My Man Jeeves* and then do the tagging for *Right Ho, Jeeves*
as well.

Take a look at what we've got:

```python
mmj_tagged[1600:1663]
rhj_tagged[1575:1650]
```

You should be seeing a description of books about birds, and a visitor with a fish-like face.

Also, notice that what we're looking at are lists of pairs.  The first member of each pair
is the word, and the second member of each pair is the category that NLTK assigned.
Observe that it has assigned `NN` to singular nouns, `NNS` to plural nouns, and `NNP` to
proper nouns.  And that it has assigned `JJ` to adjectives.

We're going to compute the ratio of adjectives to nouns.

So, the goal is to look at the text, count the nouns
(we'll include singulars and plurals, but not proper nouns), count the adjectives,
and then compute the ratio, so we can compare this to texts we'll analyze later.

A Conditional Frequency Distribution would suit our purposes nicely,
except that what a CFD is calculated on is a list of pairs, with the condition as the first
element of each pair, and the data point as the second element.
Our list of tagged words is actually in the other order, it has pairs of (word, category).
We need to turn that into a list of pairs like (category, word).

```python
mmj_catwords = [(x, y) for (y, x) in mmj_tagged]
mmj_cfd = nltk.ConditionalFreqDist(mmj_catwords)
```

Now we can ask: How many adjectives are there?

```python
len(mmj_cfd['JJ'])
```

How many nouns?

```python
len(mmj_cfd['NN']) + len(mmj_cfd['NNS'])
```

Define a function
`jn_ratio(tagged)`
that will compute the ratio of adjectives to nouns
(that is, divide the number of adjectives by the number of nouns).

What you should get when you are done is:

```python
>>> jn_ratio(mmj_tagged)
0.3697478991596639
>>> jn_ratio(rhj_tagged)
0.3761185682326622
```

Those are pretty close.

**UPDATE** *Note*: Your numbers might be slightly different.
If they are, it is probably because when I created my list,
I used `lower()` on the words.  I know (from trying it)
that this changes the results slightly.  What I'm not certain about is
why.  It does not seem like conceptually it should matter.
I will ponder this a bit.  But if your numbers are close,
that's good enough.

### Not throwing away our shot ###

Let's move to a different text.  Partly, we're going to be doing another
bit of authorship attribution.

Go back to Project Gutenberg and find the Federalist Papers.
Get the URL for the plain text UTF-8 version, and bring it in
like we did before, so that `fed_raw` has the text.

**UPDATE** *Note*: If your URL does not include the number 1404, then you probably
found an older version.  Pick the one in the Gutenberg Archive that is
most downloaded, most recent, has 1404 in its URL.

Take a look at it in the web browser.  There are 85 different subparts,
written by different people.
Each one starts with "FEDERALIST No.".
Let's break this up into individual papers.

```python
fed_papers = fed_raw.split('FEDERALIST No. ')
end_index = fed_papers[-1].find('End of the Project')
fed_papers[-1] = fed_papers[-1][:end_index]
fed_papers = fed_papers[1:]
```

**Task**. Explain briefly what each line above is doing and why.

Look at each paper.  We want to find the authors.
Generally there are a couple of lines and then the author or authors
in all caps.

If we want to find the author(s), we can read down from the top of a
paper until we reach a word in all caps.  This will be the author's
surname.  In a couple of cases there are two listed authors, and they
show up like `HAMILTON, with MADISON`.

So, our plan to find the authors of a paper is:
Scan down line by line from the top,
split the line on `', with '`, and then see if the first element is an
all-caps word.  Once we've found the author(s) stop looking.

The first step is that we need to find the lines.
Let's take a look at the first 100 characters of the first paper.

```python
fed_papers[0][:100]
```

We can see that each line ends with `\r\n`.
Those are the codes for "carriage return" and "linefeed".
So we can split on that.

```python
lines = fed_papers[0].split("\r\n")
```

Now, let's define a function that will find the authors of a paper,
following the plan outlined above.

```python
def find_authors(paper):
    for line in paper.split('\r\n'):
        authors = line.split(', with ')
        if len(authors) > 0 and authors[0].isupper():
            return ' '.join(authors)
```

This is what you should see if it is working:

```python
>>> find_authors(fed_papers[0])
'HAMILTON'
>>> find_authors(fed_papers[1])
'JAY'
>>> find_authors(fed_papers[17])
'MADISON HAMILTON'
```


**UPDATE** *Note*: 
I kind of skipped a step here.
What you want to do first is create `authors`, a list of the authors.
Do that by creating a list (which should be 85 long)
that has the author of each paper.  What I mean is that you should
wind up with

```python
>>> authors[0]
'HAMILTON'
>>> authors[1]
'JAY'
>>> authors[17]
'MADISON HAMILTON'
```

The way you probably want to create `authors` is by using a list
comprehension that loops over `fed_papers` and for each `p` in
`fed_papers` adds `find_authors(p)` to the list.

After that, the next task is basically just turning that list into
a set so that you have a list of the unique authors.  But you do
need the *list* `authors` in order to do the `FreqDist()` part
a couple of steps below,
since what it is doing is counting how many articles each author
has.

**Task**.  Define a set of the authors called `author_set`.
When you've got it, you should be able to do this:

```python
>>> type(author_set)
set
>>> print(author_set)
{'HAMILTON', 'JAY', 'MADISON', 'MADISON HAMILTON'}
```

(It is of course possible that you will see the names in
a different order.)

Now, let's see how many each author wrote.  We have a list
in `authors`.

```python
fd = nltk.FreqDist(authors)
fd.most_common()
```

According to a well-known musical,
John Jay got sick after writing five.
James Madison wrote twenty-nine.
Hamilton wrote the other fifty-one.

**Task**. So, who do we need to consider the author of the
co-authored ones, in order for those numbers in the musical to be
accurate?

Is that reasonable?
Let's take a look at a couple of stats for the papers written by
each author, and then we can compare the stats for the co-authored
ones to the stats of the individual authors.  That might allow us
to make an educated guess as to who was primarily responsible for
the writing in those.

We'll do the same thing we did above for the Jeeves books, 
looking at the ratio of adjectives to nouns.  This means that
we need to tokenize and tag the papers.

**Task**.  Use a list comprehension to
make a list of tokenized papers (using `nltk.word_tokenize()` like
above), and call it `paper_tokens`.
When you have done that, you should get these results:

```python
>>> len(paper_tokens)
85
>>> paper_tokens[0][:10]
['1',
 'General',
 'Introduction',
 'For',
 'the',
 'Independent',
 'Journal',
 '.',
 'Saturday',
 ',']
``` 

**Task**. Then, create tagged versions of each paper
(built the same was as before, using `nltk.pos_tag()` and using
the `paper_tokens` list we just built).  Call this `paper_tagged`.
When you have done that, you should get these results:

```python
>>> len(paper_tagged)
85
>>> paper_tagged[0][:10]
[('1', 'CD'),
 ('General', 'NNP'),
 ('Introduction', 'NNP'),
 ('For', 'IN'),
 ('the', 'DT'),
 ('Independent', 'NNP'),
 ('Journal', 'NNP'),
 ('.', '.'),
 ('Saturday', 'NNP'),
 (',', ',')]
```

Now, you have two parallel lists, one is `authors` with the authors
for each paper, and the other is `paper_tagged` with the tagged
words for each paper.

Since we already defined `jn_ratio()` a while back, which computes
the adjective to noun ratio for a tagged corpus, we can see what that
is for the first and second Federalist Papers.

```python
>>> jn_ratio(paper_tagged[0])
0.3765182186234818
>>> jn_ratio(paper_tagged[1])
0.4711111111111111
```

Let's get a second measure as well, we can pick lexical diversity.
We've already done this a couple of times before.

**Task**. Define a function `lexdiv()` that will take a list of
words and compute the ratio of unique words to total words.
When you are done you should be able to do this:

```python
>>> lexdiv(paper_tokens[0])
0.37569367369589346
>>> lexdiv(paper_tokens[1])
0.3603411513859275
``` 

Now, let's go through and gather together our evidence.
We're going to create a list by author of all of the
adjective-noun ratios and lexical diversity scores of each
paper, and then we can look at the ranges and average for
each author and see where the jointly authored papers fall.

Because we have a couple of parallel lists, the best way
to approach this is not to run through all of the elements
in one of the lists, but rather run through the positions
numerically.  Here is the function we will use.

```python
def collect_measures():
    author_stats = {a:[[], []] for a in authors}
    for n in range(len(papers)):
        author = authors[n]
        div = lexdiv(paper_tokens[n])
        jn = jn_ratio(paper_tagged[n])
        author_stats[author][0].append(div)
        author_stats[author][1].append(jn)
    return author_stats
```

**Task**. Consider what the function does, and explain what
is happening in the three lines that begin with `author_stats`.

If you successfully explained what is going on in the function,
then you'll see what this is doing:

```python
author_stats = collect_measures()
div_total = sum(author_stats['JAY'][0])
div_n = len(author_stats['JAY'][0])
div_max = max(author_stats['JAY'][0])
div_min = min(author_stats['JAY'][0])
div_avg = div_total / div_n
jn_total = sum(author_stats['JAY'][1])
jn_n = len(author_stats['JAY'][1])
jn_max = max(author_stats['JAY'][1])
jn_min = min(author_stats['JAY'][1])
jn_avg = jn_total / jn_n
```

And after doing the above, you should be able to get the following:

```python
>>> print('Jay: average lex. div.: {:5.3f}'.format(div_avg))
Jay: average lex. div.: 0.343
>>> print('Jay: average JN ratio.: {:5.3f}'.format(jn_avg))
Jay: average JN ratio.: 0.435
>>> print('Jay: lex. div. range: {:5.3f}-{:5.3f}'.format(div_min, div_max))
Jay: lex. div. range: 0.295-0.382
```

**Task**. Given what we have done above to find averages and
ranges for lexical diversity and ratio of adjectives to nouns,
write something that will print a little table of the
ranges and averages for the lexical diversity and adjective-noun
ratio for each author.

**Task**. Compare the "MADISON HAMILTON" papers to the "MADISON"
and "HAMILTON" papers.  If you assume that the averages are pretty
representative, even given the small number of jointly authored
papers, who seems likely to have been mostly responsible for the
writing in the jointly authored papers? 

It's probably worth noting here that the authorship in the
corpus is the result of some prior scholarship, since these were
initially published anonymously.  There are better arguments to
be made.  So, this is mostly just a bit of fun, not really any
serious conclusion.