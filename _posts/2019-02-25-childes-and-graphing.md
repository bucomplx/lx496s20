---
layout: single
title: CHILDES and graphing
tags: notes
date: 2019-02-25 22:30:00 -0400
---

Here are some notes from class, and also a (slightly editing and annotated)
Jupyter notebook file representing what we did.

Notes relating somewhat to tasks from homework 4:

If you see an "index out of range" error it almost always means that
you have tried to do something with an element of a list that turned out
to be empty.  You probably didn't expect the list to be empty.

To demonstrate this, here's an example where you might very reasonably
have not thought the list would be empty.
Suppose that you've done the following, created a little grammar,
and created a parser for that grammar, and parse the sentence
'I left' by using `parser.parse(sentence)`.  

```python
import nltk
littlegrammar = nltk.CFG.fromstring("""
S -> NP VP
NP -> "I"
VP -> "left"
""")
sentence = ['I', 'left']
parser = nltk.RecursiveDescentParser(littlegrammar)
treegen = parser.parse(sentence)
```

If at this point you check to see that you have what you want, you might
do something like...

```python
list(treegen)
```

...which will, happily, display a list of trees resulting
from parsing the sentence...

```python
[Tree('S', [Tree('NP', ['I']), Tree('VP', ['left'])])]
```

"Perfect!" you think to yourself, and decide you'd like to give it a name
so you can refer to it later...

```python
trees = list(treegen)
```

...and then try to draw the tree...

```python
trees[0].draw()
```

...you will be confronted with an unfriendly `list index out of range` error.
"But that's nonsense!" you think to yourself.  "I just looked at it, and 
`list(treegen)` gave me a non-empty list.  See?"

```python
print(trees)
```

Which shows you

```python
[]
```

See, it turns out that using the generator that `parser.parse(sentence)`
provided you with actually uses it up.  You can only use it once.
And we burned through it just checking it out by displaying
`list(trees)`.  After that, there was no generator left, so doing
`list(trees)` a second time just produced an empty list.

It's subtle, even perhaps darkly magical, but it is a way that you might
wind up with an empty list you weren't expecting.  So just be aware that
sometimes when you get a generator back, you may only be able to use it
once. 

As for the rest of what we did in class, I'll include the Jupyter notebook
file here, which has been lightly annotated and organized a bit from when
it was originally typed.

[class-20190225.ipynb]({{ site.baseurl }}/assets/ipynb/class-20190225.ipynb)


