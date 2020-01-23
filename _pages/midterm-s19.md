---
layout: single
title: Midterm
permalink: "/midterms19/"
exclude_from_nav: true
---

<div class="title-table clear-table center-table" markdown="1">

| :--------    | :--:        | -:            |
| CAS LX 394 / | NLP/CL      | "Midterm"     |
| GRS LX 694   | Spring 2019 | due Mon 3/25  |

</div>


### Ground rules ###

This is a "take-home" midterm, because I don't want to add unnecessary anxiety, or
run into technical trouble that causes things to get delayed in what would be a short
time to take an in-class midterm.

However, it is still of course important that you do this on your own.  What that
means specifically is not consulting with classmates, roommates, etc.  I do not
plan for this to be difficult.  If you (feel that you) have been doing relatively
well on understanding the homework, then these things should not be particularly
challenging.  If you are having technical trouble, please let me know, and I will
try to help you troubleshoot.

One thing you are hereby explicitly allowed to do, however, is consult static
online reference sources.  So, looking things up in the book, in the detailed
documentation on python.org, nltk.org, StackExchange, etc., is fine.  That is,
after all, how you would proceed in the real world if you have a problem you want
to solve.  If you have questions, you can ask me.

I don't think there will be much cause to worry about the ground rules, but I
still wanted to state them.  I expect this to take not (much?) more than the
time it would have taken to do this in class time.

### Problem 1 ###

*Adapted from the NLTK book, ch. 3, exercise 10.*

Read and understand what is happening in the following code.

First the setup:

```python
text = 'The dog gave John the newspaper'
sent = text.split()
```

Then a part that creates a list called `result`:

```python
result = []
for word in sent:
    word_len = (word, len(word))
    result.append(word_len)
```

After running those commands, `result` should contain the following list. 
```python
[('The', 3),  ('dog', 3), ('gave', 4), ('John', 4), ('the', 3), ('newspaper', 9)]
```

**Task 1.** The task for you is: convert the logic above (in the second block) into a list comprehension.
More specifically, your task is to replace the lines in the second block with a single line,
defining the list `result` using a list comprehension to get the same list as you
got above.

### Problem 2 ###

*Adapted from the NLTK book, ch. 3, number 25*

Pig Latin is a simple transformation of English text.
Each word of the text is converted as follows:
move any consonant (or consonant cluster) that appears at the start of the word to the end,
then append "ay".  For example: *string*
&rarr;
*ingstray*,
*idle*
 → 
 *idleay*.

See: [http://en.wikipedia.org/wiki/Pig_Latin](http://en.wikipedia.org/wiki/Pig_Latin)

The first thing we are going to do is work out a way to find the first vowel in a word.
Since we haven't really done this exactly, we can walk through one way to do it.
There is a better and more efficient way to do it that we'll cover later.
But for now:

The end goal here is to create a function that has the following structure,
where we are going to fill in the details.  This takes a string (`english_word`),
creates a list of the positions where each of the vowel types are located, narrows it down to just the
ones that actually occurred in the word, and returns the location of the first vowel it finds.

```python
def find_vowel(english_word):
    indices = [...]
    found = [...]
    # some logic here to return the position of the first vowel
    # or -1 if there were no vowels 
    return ...
```

I am going to give you a big version of this function below, and your first couple
of tasks will be to make it more compact by converting the definitions of
`indices` and `found` into single-line list comprehensions.  Much as in Task 1.

You can ask a string to tell you where a certain substring is within it.  So, you can say:

```python
>>> "Hello".find("e")
1
>>> "Hello".find("o")
4
>>> "Hello".find("H")
0
>>> "Hello".find("x")
-1
```

(Here, I'm using the same convention as in the book,
where what you type follows `>>>` and what you get back is beneath it.
It won't look quite exactly like that in Jupyter Notebook, but that's what
it would look like if you were on a less sophisticated Python command line.)

So, in the big version of the `find_vowel` function, I'll go through the
vowels, find each of them, and return the position of the one that is closest to the
beginning (or, return -1 if there were no vowels).

```python
def find_vowel_big(english_word):
    indices = []
    for v in ['a', 'e', 'i', 'o', 'u']:
        indices.append(english_word.find(v))
    found = []
    for i in indices:
        if i > -1:
            found.append(i)
    if len(found) > 0:
        vindex = sorted(found)[0]
    else:
        vindex = -1
    return vindex
```

#### Task 2a ####

Using the logic of `find_vowel_big()` above, write a more compact function
`find_vowel()`
(that does the same thing) by defining `indices` as a list comprehension,
and then by defining `found` as a list comprehension. 
(This is basically the same kind of task as in Task 1.)

The results (for either function) should be:

```python
>>> find_vowel('avenue')
0
>>> find_vowel('street')
3
>>> find_vowel('blvd')
-1
```

#### Task 2b ####

Write a function to convert a word to Pig Latin.

Specifically, the function should take a string (a word) and return a string
(that word, in Pig Latin).  So, if you called the function `pig_latin_word`, you
want to be able to recreate the following.

```python
>>> print(pig_latin_word('string'))
ingstray
>>> print(pig_latin_word('idle'))
idleay
>>> print(pig_latin_word('sun'))
unsay
>>> print(pig_latin_word('trash'))
ashtray
>>> print(pig_latin_word('beast'))
eastbay
>>> print(pig_latin_word('m'))
may
```

To begin, consider how these are formed and what use finding the leftmost
vowel is.  To work through a single example by hand:  For `string`, 
`find_vowel("string")` will return 3.

```python
>>> english_word = "string"
>>> find_vowel(english_word)
3
>>> english_word[:3]
str
>>> english_word[3:]
ing
>>> english_word[3:] + english_word[:3] + 'ay'
ingstray
```

Given that,
write the `pig_latin_word()` function that gives the results above.


#### Task 2c ####

Extend the `pig_latin_word`
function (name the new function `better_pig_latin_word`) to do three things:

- preserve capitalization (if a word is capitalized in English, it should also be in Pig Latin)
- keep *qu* together, so that *quiet* becomes *ietquay* and *squeak* becomes *eaksquay*.
- treat *y* as a consonant when it is a consonant (*yellow*) and as a vowel when it is a vowel (*style*)

Doing these things requires a bit of thinking through the problem, to figure out what we need.
Ideas:
Recall that `istitle()` can tell you if a word is in "title-case" (capitalized in the relevant sense).
So, `"Hello".istitle()` is `True`, `"hello".istitle()` and `"HELLO".istitle()` are `False`.
Similarly, `title()` can transform a string into title-case (like `lower()` transforms into lowercase).
Handling the *qu* sequence is pretty specific.  You can check to see if your first vowel is
*u* and if your initial consonants end in *q* and handle it appropriately.  And on the *y* situation,
think about the contexts in which *y* seems to be a vowel.  It is only ever going to matter
(here) when *y* is the first vowel in the word.  Note that `find_vowel()` as it stands will not
find *y* as a vowel, so either you need to modify `find_vowel()` to look for *y* too or else 
check for *y* separately. 

```python
>>> print(pig_latin_word('Quiet'))
uietQay
>>> print(better_pig_latin_word('Quiet'))
Ietquay
>>> print(better_pig_latin_word('Squeak'))
Eaksquay
>>> print(better_pig_latin_word('style'))
ylestay
>>> print(better_pig_latin_word('yahoo'))
ahooyay
>>> print(better_pig_latin_word('ugly'))
uglyay
```

That last part may take a while to get, so that's good enough.

Let me know if you run into any major roadblocks (or if you see something that looks like a typo).

