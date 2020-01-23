---
layout: single
title: Starting points II
tags: notes
date: 2019-04-17 13:30:00 -0400
---

This is just something to serve as some copying and pasting
starting points for following along in class.

SHRDLU start (again):

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

SHRDLU `obj_form`:

```python
def obj_form(obj):
    properties = obj_properties(obj) - {'held', 'thing'}
    shape = {'block', 'pyramid', 'table', 'square'} & properties
    abbrevs = [prop[:2] for prop in properties - shape]
    label = "".join(abbrevs)[:6]
    return(shape.pop(), label)
```

SHRDLU `draw_shape(obj)` (revised):

```python
def draw_shape(obj):
    if obj:
        if obj == "HAND":
            s = r"""
       /
======<=
       \
        
"""
        else:
            (shape, label) = obj_form(obj)
            if shape == 'block':
                s = r"""
/------\
|      |
|{: ^6}|
\------/
""".format(label)
            elif shape == 'pyramid':
                s = r"""
   /\   
  /  \  
 /    \ 
/{:_^6}\
""".format(label)
            elif shape == 'table':
                s = r"""
|------|
|      |
|{: ^6}|
|      |
""".format(label)
            else:
                s = r"""
========
|======|
|{:=^6}|
|======|
""".format(label)
        return s.split("\n")[1:-1]
    else:
        return [' '*8]*4
```

```python
def draw(m, g):
    stacks = [build_stack(square) for square in squares]
    # build up from the bottom
    rows = []
    while True:
        current_row = len(rows)
        empty = True
        row = []
        for stack in stacks:
            if current_row < len(stack):
                row.append(draw_shape(stack[current_row]))
                empty = False
            else:
                row.append(draw_shape(None))
        if empty:
            break
        rows.append(row)
    rows.append([[' ',' ']])
    rows.append([draw_shape('HAND'), draw_shape(obj_in_hand())])
    for row in reversed(rows):
        for line in range(len(row[0])):
            print("".join([col[line] for col in row]))
```


