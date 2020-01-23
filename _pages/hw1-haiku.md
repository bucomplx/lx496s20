---
layout: single
title: Haiku
permalink: "/hw1-haiku/"
---

<div class="title-table clear-table center-table" markdown="1">

| :--------    | :--:        | -:           |
| CAS LX 496 / | NLP/CL      | Homework 1   |
| GRS LX 706   | Spring 2020 | due Tue 2/19 |

</div>

People can argue about this, but let's just say that something will qualify as haiku if it is in three lines, with specific syllable counts: 5-7-5.  Thus: [might be attributable to Rolf Nelson]

> Haikus are easy
>
> But sometimes they don't make sense
>
> Refrigerator

So, suppose that we want to use a corpus to create such things.  We need to be able to determine whether we have met the syllable constraints.  Let's do that with the CMU pronunciation corpus.

```python
import nltk
from nltk.corpus import cmudict
```

One of the ways we can look at the CMU pronunciation corpus is in the form of a "dictionary", so we'll ask the corpus for that form and give it a name (`pro`).

```python
pro = cmudict.dict()
```

We can now look up the pronunciation of a word in the following way:

```python
pro['trimmed']
```

In response, you get:

```python
[['T', 'R', 'IH1', 'M', 'D']]
```

**Task 1**. The output for "trimmed" above looked a little bit strange because it started with two square brackets.  But this is meaningful and intentional.  The information we wanted ("how do you pronounce 'trimmed'?") is represented as a list (an ordered sequence of "phones"), starting with a "T", followed by an "R", followed by an "IH" vowel with primary stress, etc.  We know it's a list because it has entries seperated by commas and surrounded by square brackets.  This accounts for the innermost square brackets above, but the outermost brackets also are defining a list.  Look at the output of `pro['trimmed']` and compare it to the output of `pro['fire']`. The question is: What kind of thing do you get (that is, "a list of...") when you look up an entry using `pro[word]`?

**Task 2**. Look up "madelyn" this way. How many syllables does this word have?

Think about how we might be able to count the number of syllables in a pronunciation programmatically.  What is there one of in a single syllable word, two of in a bisyllabic word, three of in a trisyllabic word, etc.?  There are a couple of approaches.  One might be just to make a list of the "syllable-defining" phones (like, vowels), and count how many of those there are in the pronunciation.  But the coding of this corpus actually provides us an easier avenue: syllables are marked for stress.  We can count the number of stress marks (0=unstressed, 1=primary stress, 2=secondary stress) and that will tell us how many syllables there are.


**Task 3**. Write a function that, given a word (a string), will look up the pronunciation, count the syllables in the first pronunciation and return it.  *Note*: see immediately below for hints.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

The function takes a word and returns a number of syllables. So it would start with something like:

```python
def count_syllables(word):
    # here, some computation using the word that was passed in
    # put the number of syllables into a variable (num_syllables)
    return num_syllables
```

The first thing we are going to need to do is look up the pronunciation of the word that is passed in.  We know from above that when we look up the pronunciations of `word` using `pro[word]`, we get a list.  If we want the first (and often only) pronunciation from that list, we need to focus on the first element.  How do we get the first element of a list? Right. So, the list of phones we are going to check (the first pronunciation) is `pro[word][0]`.
 
Each element in this list of phones will look like  `IH1` or `T` or `ZH`.  To figure out how many syllables are in that we need to count how many have a stress mark (that is, ends in a `0`, `1`, or `2`).  How can we get the last letter of an arbitrary string?  Remember that you can use a negative index in a slice in order to count backwards from the end.  The textbook made use of a special function `isdigit()` that we can use here to distinguish digits from non-digits, and that is good enough.  A reasonable strategy here would be to make a list of just those phones with stress marks; the number of syllables would be the length of that list.  This is a good place to use a "list comprehension": we want to make a list of `x`es drawn from `pro[word][0]` where (`if`) the last character (`x[-1]`) is a digit (`.isdigit()`).  I'll leave it up to you to compress that into a list comprehension, but it's basically all right there.  And then we want to set `num_syllables` to be the length of that list.

If you have succeeded, you should get 2 for `count_syllables('fire')`, 5 for `count_syllables('participated')`, 3 for `count_syllables('madelyn')`.

If you're very concise, you can get this all on one line without using a variable like `num_syllables`, as below. Once you have it working above, you will probably see what I mean.  This is at this point just artistic. The main thing is just to get `count_syllables()` to return the number of syllables, however you do it.

```python
    return len([x for x in #... etc. 
```

</div>

The goal is to construct lines of 5 and 7 syllables.  Maybe the simplest way to do this, as a first pass, would be to just find all the 5 syllable words and 7 syllable words, then just use two 5 syllable words separated by a 7 syllable one.  Though this feels a bit like cheating, it's a good place to start.  This is going to lead to inspired haikus like:

```python
situational
anesthesiology
agricultural
```

So the new sub-goal is to filter through the words in the dictionary and make a list of the 5 and 7 syllable words.

**Task 4**.  Construct a list of the 5 syllable words.  Then construct a list of the 7 syllable words.  (And tell me how you did it.)

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

*Notes*: The list we want is a list of `x`es drawn from the list of words (`pro`) such that `count_syllables(x)` is 5.

If we were looking for words with more than 5 syllables, we'd check `if count_syllables(x) > 5`, but here we want words that are exactly 5 syllables long.  There is something to watch out for here.  You'd think that you might just use `=` instead of `>` if you want to test for syllable counts that are equal to 5.  But that's not right.  The reason is that `=` means "assign".  When you say `v = 5` you are assigning the value 5 to the variable `v` (or, alternatively, pointing the symbol `v` at the value of 5).  When you want to *test* for equality, you need to use a double equal sign (`==`).  So: `v = 5` sets `v` to be 5, and `v == 5` is `True` if `v` is 5 and `False` otherwise.  Usually, Python will warn you (by stopping with a syntax error) if you  accidentally use `=` instead of `==`.

I called my lists `fives` and `sevens` and you can see how many you found by using `len()` on the list.  For me, `len(fives)` is 3398, `len(sevens)` is 108.  You should wind up with the same numbers.

</div>

**Task 5**. Write a function that prints terrible haikus.  Specifically, write a function that will pick two 5 syllable words at random, and one 7 syllable word at random, and print the 7 syllable word between the two 5 syllable words.  Provide a couple such haikus.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

We want to define a function, so this would start like `def bad_haiku():` and it does not need to return anything.  Instead it will just `print()` to the screen.

In order to make a random choice, you will want to `import random` so that we get access to the `random.choice(list)` function.  Once you have done this, to pick randomly from a list, you can use `x = random.choice(listname)` so, for example, `middle_line = random.choice(sevens)` based on what I called my list of seven syllable words.

Printing to the screen is accomplished with `print()`.

You can combine things more concisely, and, rather than finding words first and then printing them, you can print a word as you find it (and thus not need to store the word in a variable), like `print(random.choice(fives))`

</div>

These are pretty terrible haikus.  It would be nice if at least it would be able to combine some shorter words to make the 5 and 7 syllable lines.  Maybe stylistically, we'd like to keep the last line at a single word, but the others really should be made of shorter words if we're going to convince anyone that we have produced something deep and meaningful.

What is involved in that?  We need to be able to get the lines to exactly 5 and 7 syllables, so we are going to need to sort out the shorter words as well.  We need a list of the 1 syllable words, 2 syllable words, 3 syllable words, ..., 7 syllable words.  We already made the 5 and 7 lists, and we could just find the other 5 by hand as well, but this solution doesn't "scale".  It's very repetitive, and doing repetitive things is something that the computer is great at.  The difference between finding 5 and 7 syllable words was just changing a 5 to a 7 in the list comprehension that finds the relevant words.  So, let's make a function that can find words of whatever syllable length we pass in.

**Task 6**. Write a function that takes a number (representing the number of syllables we want) and returns a list of words that have exactly that many syllables.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

Generally, this should be exactly like what you did to find the 5 syllable words, except that instead of using "5" we are going to use a number that was passed in as an argument to the function.

The function would start something like `def words_of_syllable_length(num_syllables):` and would create a list of words that have exactly `num_syllables` syllables, then `return` that list.

```python
def words_of_syllable_length(num_syllables):
    # create a list of words (word_list) that have
    # exactly num_syllables syllables
    return word_list
```

To be sure it is working, make sure that `words_of_syllable_length(5)` returns a list that has 3398 words in it, and `words_of_syllable_length(7)` returns a list that has 108 words in it.

</div>

Now, suppose that we want to form a 5 syllable line by picking a random 3 syllable word and then a random 2 syllable word.  With the new function we have, we could do something like 

```python
first_word = random.choice(words_of_syllable_length(3))
second_word = random.choice(words_of_syllable_length(2))
```

But there's one thing to notice here.  It takes a little while to compute `words_of_syllable_length(3)`.  So, if we are going to be needing to select a few different 3 syllable words, it would be better to just compute this once and store the result somewhere, rather than searching the whole list for 3 syllable words over and over again (and getting the same result each time).

To keep things simple, let's set up a list of lists.  Read what I did below, but don't bother typing it in because we're going to try to do better.

```python
ones = words_of_syllable_length(1)
twos = words_of_syllable_length(2)
threes = words_of_syllable_length(3)
fours = words_of_syllable_length(4)
fives = words_of_syllable_length(5)
sixes = words_of_syllable_length(6)
sevens = words_of_syllable_length(7)
sorted_lists = [ones, twos, threes, fours, fives, sixes, sevens]
```

The nice thing about `sorted_lists` is that the list of words with 5 syllables is stored at `sorted_lists[4]`, and the list of words with 7 syllables is stored at `sorted_lists[6]`---that is, whatever number of syllables we want, we can get the list of words that long by looking at the that element of the list (subtracting one to get the right index).

Creating that was kind of ridiculously repetitive, though.  Computers are good at repetitive tasks.

**Task 7.** Define `sorted_lists` using a list comprehension instead.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

The `for` clause of the list comprehension should be counting integers between 0 and 6.  If the list comprehension contains `for n in range(7)` then it will iterate over values of `n` from 0 to 6.  You can then get the word list using `words_of_syllable_length(n + 1)`.  What you are trying to do is make a list whose elements are each `words_of_syllable_length(n + 1)` for `n` between 0 and 6.

</div>

Now we can start improving our haiku writer.  Let's first try writing it so that instead of producing a 5 syllable word for the first line and a 7 syllable word for the second, it produces a 2-3 / 4-1-2 / 5 pattern.

We'll be getting things like this:

```python
backstage schueneman
electrocute cross flagship
accommodating
```

**Task 8.** Define a slightly-less-bad-haiku function that prints randomly chosen words in a 2-3 / 4-1-2 / 5 pattern.  Give a couple of examples of what you get.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

This will be like the `bad_haiku()` function except with just a bit more printing and finding of words.  Remember that if you want to print something with a space after it (instead of a carriage return), you need to `print(word, end=' ')`.

</div>

There are a couple of different ways to create a 5 syllable line.  You could have a 5 syllable word, or a 1 syllable word followed by a 4 syllable word, or five 1 syllable words.  Let's improve the haiku writer even more by letting it pick its words more freely.

So, how many different combinations are there that can lead to a 5 syllable line, and to a 7 syllable line?  We know how to do a pattern, perhaps we can just list out all of the possible patterns and then randomly choose one.

If you think about this a little bit, you'll find that it would be a big pain in the neck to list out all the possible ways you can get 7 syllables.  7 1s, or 5 1s and a 2, or 4 1s and a 3, or 4 1s and a 2 and a 1, or, or, or.  This is boring, and the kind of things computers are better at anyway.

Thinking about it further, too, you might see that there's a different way to approach it, a bit more like you would if you were writing one yourself.  You have 5 syllables.  You pick a word.  That word has 3 syllables.  You have 2 syllables left.  You pick another word, either a 1 or a 2 syllable word.  If you picked a 1 syllable word, you pick one last 1 syllable word.  And you have 5.  Let's make the haiku writer do basically this.

More specifically, let's make it keep track of how many syllables it has left, pick a random word that has any number of syllables up to the number of syllables it has left, and then pick again if it still has syllables left.

We know we are going to have to do this for 5 and for 7, so let's make a more general function that takes the number of syllables we are aiming for as an argument.

Before we do that, let's try a kind of practice run.  This is going to be a function that takes a number as an argument and will return a list of numbers that add up to the original number.

```python
def numberparts(total):
    new_number = random.randrange(total) + 1
    remaining = total - new_number
    if remaining == 0:
        number_list = [new_number]
    else:
        number_list = [new_number] + numberparts(remaining)
    return number_list
```

When I use `print(numberparts(15))` three different times, I get three different answers, like this:

```python
[3, 2, 3, 4, 2, 1]
[1, 10, 4]
[9, 4, 2]
```

The way `numberparts` works is a little bit weird.  I'll walk through it here, and I'll repeat the line I'm talking about as I talk about it.

```python
    new_number = random.randrange(total) + 1
```

First it finds a random (integer) number greater than or equal to 0, and less than `total`.  This is accomplished using `random.randrange(total)`.  (`random.randrange()` is like `range()` in that the number you provide is one higher than the function can reach.  So `range(15)` gives a list of numbers from 0 to 14, but not 15; and `random.randrange(15)` could return anything from 0 to 14, but not 15.)  Since we don't want to include 0, we add one, meaning that `new_number` winds up able to be any number from 1 to 15.

**Task 9.** Why don't we want to include 0?  

Ok, continuing on:

```python
    remaining = total - new_number
```

Then, it figures out how many we have left after our new number before the numbers add up to the `total`.  So, for a `total` of 15, if `new_number` turned out to be 10, then `remaining` would be 5.  That is, we have to find more numbers that add up to 5.

```python
    if remaining == 0:
        number_list = [new_number]
```

It's possible that we are finished already.  If the number we picked is already as big as the total we are aiming for, then the list we will return (`number_list`) can be just the simple list containing that one number.

```python
    else:
        number_list = [new_number] + numberparts(remaining)
```

If, on the other hand, we still have more numbers to find, then the list we want to return is one that contains this number (`new_number`) and then some more numbers that add up to `remaining`.  So, this is accomplished by adding together the list with `new_number` in it and the list we get from `numberparts(remaining)` which will be a list of numbers that add up to `remaining`.

```python
    return number_list
```

Then we return the `number_list` we created in one of the two last code fragments.

If your brain hurts now, or you suspect witchcraft, welcome to the world of recursive programming.  Even though it seems kind of intuitive if you were doing this yourself as a human, the weird thing about `numberparts` is that in that penultimate line we actually *use the function we are defining as part of its definition.*  Why does this not simply cause the universe to implode?

If you actually trace it through and think about what it is doing, it may make a little bit more sense.  Let me try to represent this in a table (below).  The explanation is below the table, but the idea is that as part of computing `numberparts(15)` we need to compute `numberparts(5)` first.  And as part of computing `numberparts(5)` we need to compute `numberparts(2)` first.

| --------------------:           | -------------------:            | ----------------------:    |
| numberparts(15)                 |                                 |                            |
| picked **10** (5 remain)        |                                 |                            |
| (find a list that adds to 5) => | numberparts(5)                  |                            |
|                                 | picked **3** (2 remain)         |                            |
|                                 | (find a list that adds to 2) => | numberparts(2)             |
|                                 |                                 | picked **2** (none remain) |
|                                 | ([2] is such a list) <=         | return [**2**]             |
|                                 | return [**3**] + [2]            |                            |
| ([3, 2] is such a list) <=      | (a.k.a. [3, 2])                 |                            |
| return [**10**] + [3, 2]        |                                 |                            |
| (a.k.a. [10, 3, 2])             |                                 |                            |

In words: We were computing `numberparts(15)`.  We picked a random number, it was **10**.  That by itself does not add to 15.  In addition to the **10**, we need a list of numbers that will add up to the remaining 5.  `numberparts(5)` can find a list with that property.   While computing `numberparts(5)`, we pick a random number, and it was **3**.  That does not add to 5, we also need a list of numbers that will add up to the remaining 2.  So, we call `numberparts(2)` to get such a list.  It randomly picks **2**, which does add up to 2 (that was the goal), so it returns `[2]`, which is a list that adds up to 2.  We can now finish evaluating `numberparts(5)`, which adds the `[3]` it found to the `[2]` it got back, and returns `[3, 2]` (which is a list that adds up to 5).  And then we can finish evaluating `numberparts(15)`, it adds the `[10]` it found to the `[3, 2]` it got back, and returns `[10, 3, 2]` (which is a list that adds to 15).

I went through all that because we can use this (pretty directly) to construct a list of words whose syllables add up to five, or seven, or whatever we want.  Really, the only difference between `numberparts()` and the function that we want for our haiku-maker is that instead of making lists of numbers, we want to make lists of words.

**Task 10.** Write a function `construct_line(total)` that takes a number of syllables as an argument and returns a list of words whose syllables add up to the number of syllables that was passed in.

<!-- in the future, maybe it would be better to build construct_line using numberparts, rather than rebuilding it -->

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

You want to base `construct_line(total)` on `numberparts(total)`.  The basic logic can be the same, but where `numberparts()` sets `number_list` to a list of numbers, you instead want to set `word_list` to a list of words.  (It doesn't actually matter whether you call it `number_list` or `word_list` or `steve_the_variable`, it just needs to be a list of words, and `word_list` seems like an easily-remembered name for such a thing.)

So, instead of having a list like `[2]` you want to have a list that has a randomly chosen 2 syllable word.  Fortunately, we have a list of two syllable words here: `sorted_lists[1]` (Remember?  Back before we started worrying about the universe imploding?).  A randomly chosen 2-syllable word, then, might be `random.choice(sorted_lists[1])`.

And, more generally, if, say, we want to find a randomly chosen word of length `new_number`, this can be accomplished using `random.choice(sorted_lists[new_number])`.

*Be cautious about indices*.  `new_number` in `numberparts` was 1-based (could be from, say, 1 to 15, if `total` was 15).  But `sorted_lists[]` and `random.randrange()` are both 0-based (1 syllable words are in `sorted_lists[0]`).  So make sure you're adding or subtracting 1 in the right places. 

</div>

**Task 11.** Write a slightly better haiku generation function that will use `construct_line()` three times, in order to print a 5 syllable line, a 7 syllable line, and another 5 syllable line.  Give a couple of examples you get.

Here is one I wound up with.  They look like this because I just used, e.g., `print(construct_line(5))` to print a 5-syllable line, and that will print the list (with square brackets, commas, and quote/quotation marks).

```python
>>> slightly_better_haiku()
['hegemony', "debt's"]
['unigesco', 'naderite']
['inexplicable']

```

**Optional Fun Task.** The `construct_line()` function returns a list of words.  If you want to be particularly fancy, you can print the words as (something resembling) sentences rather than printing the lists.  To assist you in that endeavor, observe what `join()` does below.

```python
>>> print(' '.join(['this', 'is', 'a', 'sentence']))
'this is a sentence'
>>> print(' !pizza! '.join(['this', 'is', 'a', 'sentence']))
'this !pizza! is !pizza! a !pizza! sentence'
```

These haikus are still pretty terrible.  It's just random words jumbled together.  So, let's take one last step to trying to make these more palatable.  *Spoilers*: they're still going to be terrible.

The plan is to use bigrams and conditional frequency distributions to try to chain the words together better, so that as much as possible, the choice of what word comes next is constrained by what has been seen to come next in the corpus.

We'll look at the "romance" category (since this seems most likely to provide poetry), and the idea is that we will look at a "romance" word, find out how many syllables it has by looking up its pronunciation, and then proceed to the next word based on words that have been seen to follow it in the "romance" corpus.  Notice that this means we need to be able to find the "romance" word in the pronunciation corpus (because we need to know how many syllables it has).  So, this is only going to work for words that are in both corpora.  Another point about this: The Brown corpus contains some words that are capitalized, but all of the words in the CMU pronunciation corpus are lowercase.  So, as a first step, we will extract the "romance" category, and then make it all lowercase.

```python
from nltk.corpus import brown
corp = brown.words(categories='romance')
lc_corp = [w.lower() for w in corp]
```

Now that we have the corpus in lowercase, we can form the bigrams (the pairs of word and next word) using `bigrams(lc_corp)` but then eliminate all of those that contain words we cannot look up in the pronunciation corpus.  So, below, we form `common_bigrams` by going through each bigram `(x,y)` and adding it to the list only when both `x` and `y` are in our pronunciation corpus.

```python
from nltk.util import bigrams
common_bigrams = [(x,y) for (x,y) in bigrams(lc_corp) if x in pro and y in pro]
```

Finally, we can create a conditional frequency distribution, that will provide for us a list of observed continuations after each word.  Recall that what this allows us to do is evaluate something like `cfd['angry']` and get a frequency distribution telling us what words followed "angry" in the corpus, and how often they did.  So, "angry at" occurred twice, and "angry had", "angry knot", and "angry with" each occurred once.

```python
cfd = nltk.ConditionalFreqDist(common_bigrams)
cfd['angry']
FreqDist({'at': 2, 'had': 1, 'knot': 1, 'with': 1})
```

In our previous haiku generator, the word choice was based on first picking a random number of syllables (within the range that we had left on the line), and then picking a random word that has that many syllables.  In our improved haiku-writer, we need to constrain this more.

Let's think about how this would work if we did it by hand.  Suppose below that we want to construct a line (a list of words) that add up to 5 syllables and which match a bigram from the "romance" corpus.

```python
>>> # we will start with "unhappy", we want 5 syllables
>>> count_syllables('unhappy')
3
>>> # we need 2 more syllables
>>> # what words can come after "unhappy"?
>>> cfd['unhappy']
FreqDist({'conviction': 1, 'his': 1, 'memory': 1, 'success': 1})
>>> # which of these can we use?
>>> # "conviction" and "memory" have too many syllables.
>>> # "success" or "his" would work.
>>> count_syllables('success')
2
>>> # that's five
>>> line = ['unhappy', 'success']
```

**Task 12**. Try building a 7 syllable line by hand on your own.  Start with the word "truth", and do it like in the example above.

Now, we will try to formalize this into a new version of the `construct_line()` function.  I will call it `construct_better_line()` and like before, we need to tell it how many syllables the line should have.  So, it should start *something* like this:

```python
def construct_better_line(total):
```

But that's not quite good enough. The `construct_better_line()` function is used each time we need to find a next word, but now the choice of the next word depends on what the previous word was.  So the function needs access to the previous word, not just the target length.  That means that we need to add the previous word as one of the things that the function takes as an argument.  So, really, it should be something like this:

```python
def construct_better_line(total, previous_word):
```

But if we're just starting a line at the beginning, there is no previous word.  Python has a special value for things that don't exist, called `None`.  We can set up this function with a "default" for the `previous_word` such that, if no previous word is provided, it is assumed to be `None`. Like so:

```python
def construct_better_line(total, previous_word = None):
```

This means there are two situations to consider, one where we have a previous word (we are in the middle of a line), and one where we don't (we are at the beginning of a line).  If there is no previous word, the choice of the next word is relatively unconstrained, we can just pick a random word (that has fewer than `total` syllables).  If there is a previous word, then we need to consult the conditional frequency distribution we built from the bigrams.  You can test for whether `previous_word` is `None` or not by treating `previous_word` as if it were a True/False value.  `None` will evaluate as `False`, anything else will evaluate as `True`.

```python
def construct_better_line(total, previous_word = None):
    if previous_word:
        # select the next word based on the previous one
    else:
        # select the first word however we like
```

Let's start with what happens if we are at the beginning of a line.  I demonstrated this above by starting with "unhappy" and then had you do one that started with "truth".  Can you just pick any random word?  Well, let's see.

**Task 13.** How many different words can follow "linda"?

**Task 14.** One of the words that can follow "linda" is "socially".  So we know that "socially" is in both corpora.  How many different words can follow "socially"?

It is not guaranteed that there will be continuations available from any word you pick.  It might be that there simply are no examples in the corpus of words following the word you picked, or it might be that all of the examples there are have too many syllables to fit on what's left of your line.  We will probably need to deal with this contingency, but at least when we are starting a line, we want to pick a word that has somewhere to go.

So, let's make a list of good starting words.  The plan is to find the 100 words with the highest number of possible following words, and when we start a new line, we will pick one of those.  This doesn't really address the issue directly, but it's a fairly easy way to give the haiku generator a fighting chance.

We need to know for each word how many continuations it has.  If we were consider "unhappy", it has 4 cotinuations.

```python
>>> cfd['unhappy']
FreqDist({'conviction': 1, 'his': 1, 'memory': 1, 'success': 1})
>>> len(cfd['unhappy'])
4
```

We need this for every word in `cfd` though.  We can make that list as follows:

```python
>>> num_next = [(len(cfd[x]), x) for x in cfd]
>>> num_next[:3]
[(1, 'patrol'), (1, 'skirt'), (2, 'rubbing')]
```

So, there is one word that follows "patrol", two words that follow "rubbing", etc.  What is useful about putting them in this format, with the number as the first element of each pair, is that we can just sort on that number to find the ones with the most continuations.

```python
>>> sorted_next = sorted(num_next)
>>> sorted_next[:3]
[(1, "a's"), (1, 'abandoning'), (1, 'able')]
>>> sorted_next[-3:]
[(766, 'a'), (859, 'and'), (1364, 'the')]
```

The words with the most continuations are at the end (it sorts from low to high), and so the 100 "most continuable" words would be `sorted_next[-100:]`. 

```python
>>> good_starts = sorted_next[-100:]
>>> good_starts[0]
(35, 'eyes')
```

Getting back to what happens at the beginning of a line in `construct_better_line`, we want to pick one of those "good starts" as the next word if there is no previous word.  It is possible that some of those words have too many syllables, though.  It's not really likely, but we should still take that into account.  So, we need to filter the list so that it just has words that have fewer than `total` syllables, and then make a random choice from among those.

```python
def construct_better_line(total, previous_word = None):
    if previous_word:
        # select the next word based on the previous one
    else:
        # select the first word however we like
        ok_starts = [w for (n, w) in good_starts if count_syllables(w) <= total]
        first_word = random.choice(ok_starts)
```

Now that we have the first word, we determine how many syllables it has, how many syllables we still need to fill, and then ask `construct_better_line` to find us a set of words that has that many syllables following our first word.

```python
def construct_better_line(total, previous_word = None):
    if previous_word:
        # select the next word based on the previous one
    else:
        # select the first word however we like
        ok_starts = [w for (n, w) in good_starts if count_syllables(w) <= total]
        first_word = random.choice(ok_starts)
        num_syllables = count_syllables(first_word)
        remaining = total - num_syllables
        line = [first_word] + construct_better_line(remaining, first_word)
    return line
```

Now, let's turn our attention to what happens when `construct_better_line` is called with a previous word.  In that case, we need to figure out what the possible continuations are from that word, filter them down to just those that do not have too many syllables, and then pick one of them.

```python
def construct_better_line(total, previous_word = None):
    if previous_word:
        # select the next word based on the previous one
        next_words = [w for w in cfd[previous_word] if count_syllables(w) <= total]
        next_word = random.choice(next_words)
```

And then we find out how many syllables the next word has, how many syllables we have left to find, and call `construct_better_line` to find the rest of them.

```python
def construct_better_line(total, previous_word = None):
    if previous_word:
        # select the next word based on the previous one
        next_words = [w for w in cfd[previous_word] if count_syllables(w) <= total]
        next_word = random.choice(next_words)
        num_syllables = count_syllables(next_word)
        remaining = total - num_syllables
        line = [next_word] + construct_better_line(remaining, next_word)
```

Now, if you're kind of following what's going on here, you will probably have noticed that there are a couple of terrible problems with what we have so far.  The most glaring of them is that it goes on forever, never actually finishing.  That's because we never check to see if we actually got all the syllables that we need.

There are a couple of ways to handle this.  One would be to check to see if `remaining` is 0, and if it is, just return `[next_word]` rather than also calling `construct_better_line` again.  But another would be to notice if we called `construct_better_line` with a `total` of 0, and if so, just return an empty list (which is, after all, a list of words that is zero syllables long).  Let's take the second route, which amounts to taking everything we had already and embedding it inside a conditional block.

```python
def construct_better_line(total, previous_word = None):
    if total == 0:
        line = []
    else:
        if previous_word:
            # select the next word based on the previous one
            next_words = [w for w in cfd[previous_word] if count_syllables(w) <= total]
            next_word = random.choice(next_words)
            num_syllables = count_syllables(next_word)
            remaining = total - num_syllables
            line = [next_word] + construct_better_line(remaining, next_word)
        else:
            # select the first word however we like
            ok_starts = [w for (n, w) in good_starts if count_syllables(w) <= total]
            first_word = random.choice(ok_starts)
            num_syllables = count_syllables(first_word)
            remaining = total - num_syllables
            line = [first_word] + construct_better_line(remaining, first_word)
    return line
```

**Task 15.** Suppose we call `construct_better_line(2, "the")`.  This is looking for a list of words totalling 2 syllables that can follow the word "the".  Suppose that our random choice gave us the word "pickle", so `num_syllables` gets set to 2.  What does `line` get set to?

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

The `total` variable was not 0, and there was a previous word, so we're in that middle block.  The value that `line` gets is `[next_word]` plus whatever `construct_better_line` returns.  We know what `remaining` will be, and `next_word` is by hypothesis "pickle".

What I'm asking you to think about here is what happens when we call `construct_better_line` with `(0, "pickle")`, and what does ultimately mean `line` gets set to?

</div>

We're almost done here.  There is one prominent special case that we have so far failed to handle.  If there is a previous word, we set `next_words` to be a list of possible continuations that do not have too many syllables.  But what happens if either there are no possible continuations, or if the ones there are are too long?  In that case `next_words` would be an empty list, and we can't choose a member from an empty list.

This situation could quite easily arise, so we have to think about what to do if it does.  In order to keep this relatively simple, let's say that if we wind up in that situation, we simply start over as if we didn't have a previous word.  We can just take the next word from the list of "good starts".

We can tell if we are in that situation right after we compute `next_words`, so let's add a conditional block there:

```python
            next_words = [w for w in cfd[previous_word] if count_syllables(w) <= total]
            if len(next_words) == 0:
                # problem! We need to pick a new start word
            else:
                # no problem! do as before
                next_word = random.choice(next_words)
```

The idea is that we want to pick the next word not on the basis of the previous word, but as if we did not have a previous word.  Although there is surely a more elegant way to do this, let's just copy what we do when there is no previous word:

```python
            next_words = [w for w in cfd[previous_word] if count_syllables(w) <= total]
            if len(next_words) == 0:
                # problem! We need to pick a new start word
                ok_starts = [w for (n, w) in good_starts if count_syllables(w) <= total]
                next_word = random.choice(ok_starts)
            else:
                # no problem! do as before
                next_word = random.choice(next_words)
```

At this point, we are basically finished.  There is one situation we have not handled, which is if there are also no words in `good_starts` that are short enough to fit.  However, looking at `good_starts`, it is full of one-syllable words, so there will never be a situation where we can't find something in there that will fit.  Given this assumption about the corpus, we can safely leave that contingency unaddressed in our function.

So, here is the whole function, then:

```python
def construct_better_line(total, previous_word = None):
    if total == 0:
        line = []
    else:
        if previous_word:
            # select the next word based on the previous one
            next_words = [w for w in cfd[previous_word] if count_syllables(w) <= total]
            if len(next_words) == 0:
                # problem! We need to pick a new start word
                ok_starts = [w for (n, w) in good_starts if count_syllables(w) <= total]
                next_word = random.choice(ok_starts)
            else:
                # no problem! do as before
                next_word = random.choice(next_words)
            num_syllables = count_syllables(next_word)
            remaining = total - num_syllables
            line = [next_word] + construct_better_line(remaining, next_word)
        else:
            # select the first word however we like
            ok_starts = [w for (n, w) in good_starts if count_syllables(w) <= total]
            first_word = random.choice(ok_starts)
            num_syllables = count_syllables(first_word)
            remaining = total - num_syllables
            line = [first_word] + construct_better_line(remaining, first_word)
    return line
```

For how long it took us to build it, it seems kind of short.  But, it should do what we said we wanted to do.

So, now we can actually test it.  Let's see how `better_haiku()` fares.

```python
def better_haiku():
    print(construct_better_line(5))
    print(construct_better_line(7))
    print(construct_better_line(5))
    print()
```

Here are a couple that I got.  They're still horrible, but maybe a little bit more coherent.

```python
>>> better_haiku()
['with', 'questioning', 'eyes']
['their', 'personal', 'god', 'rest', 'last']
['them', 'you', 'wear', 'something']

>>> better_haiku()
['are', 'talking', 'down', 'at']
['she', 'walked', 'away', 'in', 'its', 'dark']
['if', "you'd", 'rather', 'a']

```

**Task 16.** Try it on a couple of different corpora.  Earlier, we set `corp` to be `brown.words(categories='romance')`, but try it with `news` or `humor` or `religion`.  Give me two haikus from each of two different categories in the Brown corpus.

**Task 17.** Congratulate yourself for having stuck with this very verbose homework assignment.

**Optional Tasks For Fun.** There are lots of ways this could be improved or optimized.  Would the results be better if you isolated words that occur at the start of sentences (rather than words that have the most possible continuations)?  Would it be useful to remove stopwords from the "good starts"?  Would it be better to try to make more common continuations more likely to be chosen?  Can you extend this to make a limerick generator?  Or iambic pentameter?






