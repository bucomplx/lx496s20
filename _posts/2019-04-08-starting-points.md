---
layout: single
title: Starting points
tags: notes
date: 2019-04-08 13:30:00 -0400
---

This is just something to serve as some copying and pasting
starting points for following along in class.

The tale of Andrea, Bobby, Chris, Dana, and the Spaceballs.

```python
import nltk
vfs = nltk.Valuation.fromstring
dom = {'a', 'b', 'c', 'd', 'm', 's'}
names = """
andrea => a
bobby => b
chris => c
dana => d
the_sun => s
the_moon => m
"""
val = vfs(names)
val.update(vfs("person => {a, b, c, d}"))
val.update(vfs("spaceball => {s, m}"))
val.update(vfs("bostonian => {a, b}"))
val.update(vfs("cantabrigian => {c, d}"))
```

Defining names pass one:

```python
npdef = """
NP[SEM=<andrea>] -> 'andrea'
NP[SEM=<bobby>] -> 'bobby'
NP[SEM=<chris>] -> 'chris'
NP[SEM=<dana>] -> 'dana'
NP[SEM=<the_sun>] -> 'the_sun'
NP[SEM=<the_moon>] -> 'the_moon'
"""
```

Defining names pass two:

```python
npdef = r"""
NP[SEM=<\P.P(andrea)>] -> 'andrea'
NP[SEM=<\P.P(bobby)>] -> 'bobby'
NP[SEM=<\P.P(chris)>] -> 'chris'
NP[SEM=<\P.P(dana)>] -> 'dana'
NP[SEM=<\P.P(the_sun)>] -> 'the_sun'
NP[SEM=<\P.P(the_moon)>] -> 'the_moon'
"""
```

SHRDLU start:

```python
import nltk
print("Setting up the world.")
squares = ['s1', 's2', 's3', 's4', 's5', 's6', 's7', 's8']
dom = {'a', 'b', 'c', 'd', 'e'} | set(squares)
valstr = """
square => {s1, s2, s3, s4, s5, s6, s7, s8}
odd => {s1, s3, s5, s7}
even => {s2, s4, s6, s8}
block => {a, b}
pyramid => {c, e}
table => {d}
thing => {a, b, c, d, e}
red => {a}
blue => {b, e}
green => {c, d}
on => {(a,s1),(b,s2),(d,s4),(c,d)}
"""
val = nltk.sem.Valuation.fromstring(valstr)
val['held'] = {('e',)}
m = nltk.Model(dom, val)
g = nltk.Assignment(dom)
```