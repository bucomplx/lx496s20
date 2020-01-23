---
layout: single
title: CHILDES
permalink: "/hw5-childes/"
---

<div class="title-table clear-table center-table" markdown="1">

| :--------    | :--:        | -:           |
| CAS LX 394 / | NLP/CL      | Homework 5   |
| GRS LX 694   | Spring 2019 | due Mon 3/4  |

</div>

### Working with CHILDES ###

In class, I started describing a project that I thought would be an
interesting thing to do with CHILDES, having to do with looking at
bilingual corpora to see if we could find evidence that the point
at which children exit the "root infinitive" stage matches in both
languages.  Since one of the primary hypotheses about this phenomenon
is that it is maturational, it should be the case that it should
not be language-dependent but rather individual-dependent.

I still think it's a cool idea for a project, but there are some logistical
problems.  First, there is the kind of predictable issue that it's going
to be hard to do without knowing at least a bit of the languages involved,
so that "non-adult" forms can be identified.  Second, there is the fact
that not all languages seem to show the effect, such that it is
(perhaps in principle, but if not at least observationally mostly)
correlated with not being a null-subject language.  And it also needs
to have identifiable infinitives.  So, the second obstacle is that
not any pairings of languages will work.  Which leads to the third
obstacle: finding appropriate pairs of languages where both languages
are expected to show root infinitives, which cover the relevant age
ranges (like around 1;6 to 3;6) with a sufficient number of transcripts,
in languages we know well enough to analyze.  There aren't many options,
and it turns out that even among those, they're not tagged in a way that
makes the project easy, because the original researchers were looking
at other things.

But, let's maybe see how one might start.  The main thing we're really
confronting here is the problem of insufficiently tagged corpora, and
the relative lack of power of the `CHILDESCorpusReader`.

To begin, we can look at the
[descriptions of the bilingal corpora in CHILDES](http://childes.talkbank.org/access/Biling/)
for a corpus that:

- is bilingual,
- involving two languages that both show root infinitives,
- both of which you know well enough to make sense of the transcripts,
- has children between about 1.5 and 3.5 years old,

From a language standpoint, the ones
with Cantonese, Catalan, Chinese, Italian, Japanese, Portuguese,
and Spanish are out, at least.  I think Farsi and Hungarian too.
German, French, Russian, English, Dutch,
Danish could work.  But, this is not many from which to choose.
Particularly once we limit the age ranges, and then look for corpora
that are large enough.

Probably the most promising one is the GNP corpus, where I am
making the unwarranted assumption that French is one of the
languages "you well enough to make sense of the transcripts."
It will not in fact be required that you learn French to do this
homework.

In most of these corpora (including the GNP corpus),
we have primarily just the words.  Fair enough,
these were transcribed for some particular purpose of the original researchers.
But it means that if we want to do things like look for main clause infinitives, we need
to do some more work than just searching for verbs and infinitives.

Before we get there, though, there are some other things to consider.
Recall that there are some kind of standard searching/analysis utilities
that are distributed alongside CHILDES.  You have access to them in the
browsable database (individual commands like `mlu` and `freq`), and you
can download a version for your local computer as well.  They are collectively called
CLAN (CHild language ANalysis, I think).  And they are pretty well suited to a lot
of things one might want to do with CHILDES.  In the real world, if you are working
with CHILDES, you might want to use those rather than using NLTK and Python,
because, as it turns out, they are more capable than the `CHILDESCorpusReader`.

Still, let's see how we can do in Python.  It is certainly more flexible,
even if it doesn't have everything we want already built in.

### XML vs. CHAT ###

If you look at the description of the particular corpus you want to use, there will
likely be links both into the browsable database and to a file you can download.
However, the file you download there is almost certain to be in CHAT format
(files ending in `.cha`).  NLTK does not know how to handle those
(except as general text files), the `CHILDESCorpusReader` is designed for XML files.

CHAT is a format that is specific to CHILDES
(or at least to Talkbank, which kind of grew out of CHILDES)
and is well specified.  The format guidelines are
specific.  Participant and recording information goes at the top in specific forms,
participants are referred to by three-letter codes (CHI = child, MOT = mother, etc.),
and individual utterances begin with `*`  and the participant (`*CHI:` ), while
dependent "tiers" begin with `%`, and so on.  You can read the CHAT manual if you
like, and you can kind of absorb how it works by looking at the browsable transcripts
of any corpus.

When NLTK (or more specifically `CHILDESCorpusReader`) is processing CHILDES files, it
wants them to be in XML format, rather than in CHAT format.  XML adds an extra layer
on, it's still basically in the CHAT format but it is "marked up" to be more
computer-program-friendly.  XML stands for eXtensible Markup Language, and that's
probably a whole separate thing that's not worth spending a lot of time on here.
I talked about it a little bit in class, it's related to the HTML
(HyperText Markup Language) that the web is essentially "written in."
The main thing to know about it I guess is that it defines a kind of hierarchical
structure for a document, like a tree, with nodes that can be assigned properties.
We will look at it a bit more closely later.

Many/most of the corpora in CHILDES are in XML format already, but you need to
go to the XML-specific section to find them (these are not in general linked from the
main page that describes the corpus).  So, once you pick a corpus you want to
use, you want to look in
[the bilingual XML corpora directory](http://childes.talkbank.org/data-xml/Biling/)
to find the XML files for that corpus.

Last time I looked, not *all* the corpora have XML versions already made.
I'm not sure why not.  For example the `FallsChurch` Japanese-English
bilingual one does not seem to be in the list of available XML corpora.
(Though, that one wouldn't be great for this project due to Japanese
not showing root infinitives in any obvious way.)
CHAT is well enough defined though that there is a pretty easy way to convert
from CHAT to XML.  There is a program to do this called
[Chatter](http://talkbank.org/software/chatter.html).
It is easy to download and use on the Mac, and there is a Java version that
is supposed to work on Windows and Linux.  I did not try it on Windows or Linux
though.  To use it, unzip the CHAT files you have downloaded, open the Chatter
program, then choose "Open" in Chatter, select the folder where the CHAT files are,
and it will process them into a folder that it creates alongside the folder with
the CHAT files.  It will give it the same name but with `-xml` at the end.

Once you have XML files, we can start operating with them in NLTK.  So, now we
can do some Python again.

### Finding the files ###

NLTK does have set of designated places where it, supposedly, will look for and
find files.  These places it looks are often referred to as the "search path".
I have for whatever reason run into many and persistent problems with NLTK
finding its data files, though.  So, I am going to recommend putting files
that you want Python to find into the same folder your ipynb files are.
Someday later if you start using NLTK/Python for bigger projects you can
start working out which places are in the search path, but let's not get
sidetracked by that here.

**Task 0.** Download the GNP corpus from the CHILDES site
and put the GNP folder that results from unzipping it into your
work folder (wherever your ipynb files generally are).   

Inside this folder, you should see that there are three folders in the
`GNP` folder: `Both`, `English`, and `French`.  I'm just going to
look at the English one in the examples below.

**Task 0.5.** Bring in NLTK and make sure NLTK can find the corpus.

```python
import nltk
from nltk.corpus.reader import CHILDESCorpusReader
gnpeng = CHILDESCorpusReader('GNP/English/', '.*.xml')
print(gnpeng.fileids())
```

You should get a list of the fileids in the corpus.  This much should
work whatever corpus you are using really (not just the GNP/English one).


At this point we can do the stuff that the `CHILDESCorpusReader` allows
us to do.  But it's a little disappointing.  Still:

## Finding participants ##

We can find out who the participants are the transcripts by using
`.participants()`.  Above we saw that `gnpeng.fileids()` gives us a
list of all the transcripts.  If we give that list of transcript
(filenames) to `.participants()`, it will work out who the participants
are in each one. 

```python
people = gnpeng.participants(gnpeng.fileids())
```

This is a list of things, one thing for each transcript.  Let's look
at the results from the first transcript:

```python
people[0]
```

You'll see some gobbledygook, but embedded in there is information
you can recognize.  There is a `{'CHI': ... {'id': 'CHI', 'role: 'Target_Child', ...}...}`
in there at least.  You can see that for each of these participants
we have information on `id`, `role`, and `language`.

To actually count the participants, you can ask for the length:

```python
len(people[0])
```

To get information on the 'CHI' participant in the first transcript:

```python
people[0]['CHI']
```

Or more specifically the role of the 'CHI' participant in the first transcript:

```python
people[0]['CHI']['role']
```

We probably will not have much call to use this information, but it is available.
To print the participants, we can do:

```python
transcripts = gnpeng.fileids()
people = gnpeng.participants(transcripts)
for i in range(len(transcripts)):
    print('**** transcript: ' + transcripts[i])
    for person in people[i]:
        print('** participant: ' + person)
        for property in people[i][person]:
            print(property + ": " + people[i][person][property])
```

**IMPORTANT NOTE:**
I have fixed the code above since this was originally posted.
It was quite wrong before.  If you got an error about `transcript`
not being defined, that would have been the old version.  It is supposed
to be going through the people in each transcript, and then the properties
of each person.  I really have no idea how the code I had before got onto
this page, because it never had a hope of running right.  The one above
does what I intended and actually makes sense.

What that does is not entirely transparent, but you can probably see how it works
if you look at if for a little while.

## Finding the actual words ##

More interesting is going to be working with the actual words and sentences.
Let's get those.  For now, let's just limit ourselves to the last file
in the set of transcripts for now. 

```python
last_file = gnpeng.fileids()[-1]
```

So, let's get the sentences for the child.

```python
sentences = gnpeng.sents(last_file, speaker='CHI')
```

**Task 1.**
It's probably about time for me to ask you to do something.
So, find the first sentence of these `sentences` we just pulled from 
`last_file` that has a verb in it.  (Just by inspecting each sentence.)

Great, now we have an interesting sentence.  Let's get the tagged
sentences.

```python
tagged_sentences = gnpeng.tagged_sents(last_file, speaker='CHI')
```

Take a look at that same sentence you found, with the verb in it.
(That is, if you had found the first verb in the seventh sentence, then
you would look at `tagged_sentences[6]`)


The thing that's (potentially) disappointing is that there are no tags.
Using `tagged_sents()` or `tagged_words()` just returns a bunch of pairs
of words with empty strings.

So, although there *are* corpora for which the words are tagged, there
are lots of other corpora for which we don't have tags
(tags being usually at least the syntactic category).
So, this means we are not going to have an easy way to just
"search for the verbs" or anything.

## Dealing with the lack of POS tags ##

If we really wanted to look for non-adult verb forms in these corpora,
then, we need a strategy that is more involved than just
"search for things tagged as verbs." 

Here's one possible strategy I thought of, at least.
The point of using Python and programming is that we can work with corpora
without going through them by hand.  But since we're going to need to go through
something by hand, we can at least maybe try to optimize what we look at.
To that end, the plan will be to look just at the occurrences of the most
common verbs.

The only thing we can do with this untagged corpus is to count up the
words and how often then occur, so we will start there.  Then we can
look through the most common words to identify which of the ones near the
top of the list are verbs.

**Task 2.**
You can get all the words with `words = gnpeng.words(gnpeng.fileids())`.
Create a frequency distribution of these and print the most common 20 words.

**Task 3.**
Identify the most common two verbs, not counting *be*, *do*, or *have*
(in any form).  What are they?

The plan then would be to look for all of the instances of these two
most common verbs and see how often they occur in adults forms vs.
non-adult infinitive forms.


## Finding the verbs ##

Ok, now that we have a small set of things we want to look for,
how do we start looking?

Here's where the `CHILDESCorpusReader` is most disappointing, I think.
We can find the sentences in the corpus from a particular speaker,
but we can't see what other speakers said in between.  And if we
just look at all the utterances, we can't tell who is saying which
line.

Yet, for this, it is kind of important to do things like not count
direct repetitions (either of the adult's or the child's own utterances).
But how can we know what is a repetition without the context?

It's probably informative to look at the XML file itself.  Below I've given what
we see in a couple of these utterances.  It's useful to see the structure
here. There is an utterance indicated by an opening `<u ...>` tag
(and closed by a `</u>`), and inside each utterance we have a series of
words enclosed by `<w>` and `</w>` tags.  The utterances have attributes
`who` (for the speaker) and `uID` for the utterance ID.  That's very
interesting/useful to see.  This means that we can pinpoint any utterance
in a transcript by referring to its `uID`.  There are also a couple of other
tags.  One is `<t type="p"></t>` which seems to correspond to clause type
or turn type---it distinguishes between statements (`"p"`) and questions
(`"q"`) at least.  And there is a more arbitrary tag (`<a>...</a>`)
that holds codes of special interest to the original researchers.
The `type="coding"` one marks what language the utterance is in and
to whom it was addressed.  The other one (`type="extension"`)?  I don't know.
Whatever.

```xml
  ...
  <u who="CHI" uID="u12">
    <w>I</w>
    <w>want</w>
    <w>go</w>
    <w>play</w>
    <w>make</w>
    <w>a</w>
    <w>house</w>
    <t type="p"></t>

    <a type="extension" flavor="pho">ai want go ple mek a haus</a>
    <a type="coding">$LAN:E $ADD:MOT</a>
  </u>
  <u who="MOT" uID="u13">
    <w>you</w>
    <w>want</w>
    <w>to</w>
    <w>go</w>
    <w>make</w>
    <w>a</w>
    <w>house</w>
    <t type="p"></t>

    <a type="coding">$LAN:E $ADD:CHI</a>
  </u>
  ...
  ```

So, back to our disappointment with `CHILDESCorpusReader`---it doesn't
(again, as far as I know) give us access to that `uID` attribute of an
utterance when we retrieve it.  However, `CHILDESCorpusReader` is itself
a type of a more general `XMLCorpusReeader`, and using this we can actually
get access to the parsed XML directly.  That will allow us a much more
flexible way into these transcripts, though at the cost of having to
deal with another bit of technology.

So, step one is to get the XML representation of the corpus we read.
This can be done like so:

```python
the_xml = gnpeng.xml(last_file)
```

The `.xml()` call does require exactly one file, so we need to specify
which transcript file we are going to look at.  We'll look at the last
one, which earlier we had named `last_file`.

### Finding our way around the XML ###

There is some brief discussion of using XML in the
[NLTK book chapter 11](http://www.nltk.org/book/ch11.html), section 4.

However, probably the most rigorous place to look for examples is
[the official Python documentation for XML ElementTree](https://docs.python.org/3.6/library/xml.etree.elementtree.html#xpath-support).
I'm going to just mention a couple of things here.

The basic goal here is to be able to look at an utterance and
figure out the speaker (`who`) and the utterance ID (`uID`), which
we know is in the XML file but is inaccessible through the
`CHILDESCorpusReader`.

So, the first thing we'll do is find the utterances by
searching for the `<u>...</u>` tags.  This can be accomplished
by using the `findall()` function called on the XML structure.

This *should* look like this---but, it actually doesn't quite.

```python
utterances = the_xml.findall('u')
```

The thing above will not find anything, even though if you look at the
XML file, there are `u` tags there.  Why?  The source of the issue is
that at the top of the XML file, it specifies a "namespace":

```xml
<CHAT xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns="http://www.talkbank.org/ns/talkbank"
      xsi:schemaLocation="http://www.talkbank.org/ns/talkbank http://talkbank.org/software/talkbank.xsd"
      PID="11312/c-00001462-1"
      Version="2.5.0"
      Lang="eng"
      Corpus="Genesee"
      Date="1994-03-08">
      ...
```

The `xmlns` is the XML Namespace, and it is `http://www.talkbank.org/ns/talkbank`.
The point of specifying this is to allow mixing of tags from different files together.
This file has `u` tags, but other XML files might use `u` not for "utterance" but for
"underline" or something.  So, the *real* tag, as far as the XML parser is concerned,
is not `u` but rather `{http://www.talkbank.org/ns/talkbank}u` -- that is, it is the
namespace in braces preceding the tag we see in the file.  So, what this boils down
to is that to find the utterances we need to do this:

```python
utterances = the_xml.findall('{http://www.talkbank.org/ns/talkbank}u')
```

That will work, but it's clunky, we need to put the namespace before any tag
we are going to search for.
So what I will do is put the namespace in its own variable:

```python
ns ='{http://www.talkbank.org/ns/talkbank}'
utterances = the_xml.findall(ns+'u')
```

We can now interrogate the `who` and `uID` (on, for example, the 5th utterance)
like this:

```python
print(utterances[4].get('uID'))
print(utterances[4].get('who'))
```

Suppose we now want to print out utterance `u4`.
What *is* utterance `u4`?  Looking at the XML, the utterance is parent
to a sequence of words (among other things), so we can collect them like this:

```python
ws = [w for w in utterances[4]]
```

This isn't quite what we want, though.  This has collected the child elements
of the utterance,
but not all of them are words.
And even when they are words, we need to ask the word element
what its `text` is to get the word if we want to print or compare it to something.
So, there are two things we want to do.  One is we want to make sure we are look at words
(the `w` tag), and the second is that we want to collect the text
(since out current side quest is to print the words of the utterance).

```python
words = [w.text for w in utterances[4] if w.tag == ns+'w']
print(words)
```

Ok, good, now we're getting somewhere.  We are starting to be able to get
access to the data in the corpus at a deeper level.

**One last note here**.  This for some reason does occasionally fail,
because the corpus might return `None` for `w.text`.  To avoid problems
with this, find the words like this, by adding `if w.text and` as well
(`if w.text` does not execute if `w.text` is `None`).

```python
words = [w.text for w in utterances[4] if w.text and w.tag == ns+'w']
```

**IMPORTANT ADDENDUM:**
I have fixed the code above since this was originally posted.
I'd accidentally checked for whether `w` was `None` and not for
whether `w.text` was `None`.  The logic of the text above it still
makes sense.

## Searching and finding and being ##

Ok, now, let's see if we can find the verb *is* in the transcript and then
print out a) what we found, and b) the lines surrounding it so that we can
make a human call on whether it is a repetition or not.
This is something that the CHILDES CLAN programs do pretty easily.
Turns out it is more of a chore in NLTK here, but still maybe useful.

Let's say what we want is this:
We want to search all the transcripts, find (exactly) *is* whenever
it is spoken by the child, and then print the two utterances before.

**Task 4.**
This is perhaps kind of challenging but: do that.
Specifically, define a function `be_in_context` that will
go through each transcript in `gnpeng.fileids()`, go through
each utterance in the current transcript, check
(if the speaker of the utterance is 'CHI') whether the word *is* is
in there, and if so print the utterance.  If you get this far,
then try to print the two utterances (from anyone, CHI, MOT, etc.)
before the one that has *is*
in it.
(Note: this is slightly more challenging. There are a couple of
ways to do it, one might be to fairly inefficiently re-scan the
file for the previous utterances, counting on the utterances being
sequentially numbered without skipping.  Another might be to
introduce a "memory" of what you are scanning as you go through
the file, so that you always have the previous two utterances
ready to print in case you find an *is*.  The main task here is 
to get it to work, a secondary concern is the efficiency.)
A third refinement of this would be to print
the utterances as text (not as lists of words), and maybe
also put little lines between groups of three, handle
cases where the first line or two has a child *is* utterance.
And maybe print the transcript name at the beginning of each one.

As a way to check, my version of this (with all those bells, whistles) produces:

```
Transcript: gen13b11b.xml
Transcript: gen13b11m.xml
Transcript: gen23b07m.xml
u158-MOT:we're going to put this one aside cause I know you like it
u159-MOT:we're going to find
u160-CHI:this is yyy
--
u369-MOT:that's not a mushroom
u370-MOT:here let me fix this
u371-CHI:yes it is mushroom
--
Transcript: gen33b01m.xml
u0-CHI:what is that
--
u7-MOT:xxx if you put it in the box your puzzle will fit just nice
u8-MOT:put your puzzle just nice
u9-CHI:where is this one go
...
```

It took a bit of fiddling to get it right.
I anticipate this to be at least a mild challenge.

### Good enough ###

Actually, given that it took me a while to post this, that's good enough
for now.  In class maybe we can extend this.

The next steps in the project would be to make a list of those words
we found that were frequent, and look for those instead of looking for *is*
and then figure out a way to tag the non-adult ones.  So, maybe this will
stretch over a couple of weeks as we work out how to do this, since there
are still some interesting complexities that arise.