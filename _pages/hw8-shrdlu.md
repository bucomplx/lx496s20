---
layout: single
title: SHRDLU
permalink: "/hw8-shrdlu/"
exclude_from_nav: true
---

<div class="title-table clear-table center-table" markdown="1">

| :--------    | :--:        | -:            |
| CAS LX 390 / | NLP/CL      | Homework 8    |
| GRS LX 690   | Spring 2019 | due Mon 4/29  |

</div>

## Note about the due date ##

**Due to the last class having been on Wednesday,
and due to this being kind of a long project, it is due about
a week and a half after having been assigned, Apr 29.**

However, it's long.  Interesting I think.  But it's really long.

## SHRDLU ##

The first part of this we did in class, kind of.
I've made some changes, particularly to the world-drawing
to simplify it first (and then make it more complicated later)

As we saw briefly in class, SHRDLU is Terry Winograd's 1971 natural language
"understander" that took user input, interpreted it in the context of a small
"block world" and seemed to be pretty good at conversing about it.

What we're going to do here is make a "little" version of this.  It's still 
actually pretty extensive, but there will still be a lot left to do by the
time we're done.

There are few different separable parts of this endeavor, some more related
to Python than they are to NLTK, but by the end you should have a cool little
program you can interact with in a limited way.

The basic parts of the program are:

- representation of the objects in the world
- grammar for syntactic parsing and semantic composition
- display module to show the current state of the world
- user input loop
- interpretation of user input to respond

The goal by the end is going to be for "SHRDLU" to be able to
answer questions like "Is the red block on an even square" and
perform actions like "put a pyramid on a block". 

*Note:* There is a lot of reading here, and in most cases I'm giving you
the pieces of the program.  It should generally be possible to copy and
paste from this web page into the Python text editor so it does not all
need to be typed.  But even if you are copying and pasting it you should
be trying to understand what it is doing, why it is written that way,
how it works.

## Setting up the world ##

The setup of the world is the same sort of thing that we did in
the previous homework about semantics.
We are creating a model of the world, with a domain
of individuals, and a valuation function that determines how
the objects are arranged in the world and what properties they
have.

Rather than having a 3D world, for simplicity and since it
doesn't really matter, our world just has 8 squares in a line
on which things can be piled.

So, let's begin.  You can start with this:

```python
import nltk
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

This completely specifies the world now.  We have 13 objects, 8 of which are
squares that represent the floor (4 of which are odd, 4 of which are even),
and 5 of which are shapes of various kinds (block, pyramid, table) with various
properies (red, big, etc.).  Three of the objects are on the floor, one object
is on another one, and one is in the robot hand.

## Looking at the world ##

It's kind of nice to be able to visualize what's happening, so we next want to
set up a way to display the state of the world.  Rather than risk using graphics,
we will be satisfied with a text representation.

So, the plan is this: Each of the floor squares will represent a column, and
we will stack shapes up from the floor.  A little bit above the stacks will
be the robot hand and the shape that it is holding.

Let's think a little bit about how we will put the information about our
world onto the screen.  This is a problem to solve, even if it is not really
natural language specific.  We want to have a 2D display, with the squares
along the bottom, and the objects arranged above them.  At the top,
we'll have some kind of representation of the robot hand.

The most simplified version of the world we just set up might look like
this:

```python
<A

   A
OO T
########
```

Where `#` represents a square, `O` represents a block, `T` represents
a table, `A` represents a pyramid, and `<` represents the robot hand.
In this image, we can see what shape everything is, and how they are
stacked up.

So, let's first try to display the world like this.  As you know from
our discussions in class, we will do something more complex later, but
I think starting with this simple representation of the world may help
clarify what we're doing in the more complex version.

I'm going to try to build this up by prototyping functions as we go.
First, something that, when given a list of rows, will print them.

```python
def draw_simple_world(rows):
    print('\n'.join(rows))
```

And the rows that we want for our world in its initial state, for this
simple representation, would be:

```python
initial_rows = [
    '<A      ',
    '        ',
    '   A    ',
    'OO T    ',
    '########'
    ]
``` 

And indeed, if we give those rows to `draw_simple_world()` we get the
desired output.  So far, nothing very difficult or surprising has happened.

```python
draw_simple_world(initial_rows)
```

But, now: How do we make this reflect the actual state of the model,
whatever it is?  How do we define those rows in an algorithmic way?

The top two rows are somewhat easy, since those rows are just the hand and
then an empty row.  So we'll start by looking at how to draw the first row,
the hand and what it's holding.

How do we know what to put in the hand?  We have
defined the predicate "held" in our valuation function, so the object
in the hand will be the one that has the property of being "held".

```python
print(val['held'])
```

We see that we can get the object being held by asking for `val['held']`
but what we want is to get `e` back (not `{('e',)}`.
So: how to get `'e'` out of that?  Well, it's the first element of the first/only
element, except that the concept of "first element" doesn't make sense for
a set.  So, we need to convert this to a list, retrieve the first element
(which will be `('e',)`) and then retrieve the first element of that.
The following function will do just this, and if nothing is being held, it
will return `None` instead.

```python
def obj_in_hand():
    if len(val['held']) == 0:
        return None
    else:
        return list(val['held'])[0][0]
```

You can see that it works by trying this:

```python
obj_in_hand()
```

Now that we know what object is being held, what shape is it?  Should we
draw an `A` (pyramid) in the hand?  Or a `O` (block)?  Or what?
To figure this out, we need to figure out what properties this object
has (apart from being "held").

What are properties?  They are defined in our model.  Specifically, each
property is a list of 
objects that have the property.  For example, the objects that have
the property of being a pyramid are:

```python
val['pyramid']
```

We see that `e` is in there, `e` is a pyramid (and `e` is also held).
So we know that we want to draw a pyramid in the hand because the thing
that is being held has the property of being a pyramid.

We are more generally going to want to know the properties of an object,
not just whether it is a pyramid, but also whether it is blue, etc.  So,
generally what we want is, for a given object, to be able to get a list
of all the properties/predicates it is a member of.
What are the predicates? The following will print a full list of them:

```python
print([v for v in val])
```

We want to include only those predicates that have a given object in
them.  So, the properties of `e` would be:

```python
print([v for v in val if ('e',) in val[v]])
```

It's a pyramid, it's blue, it's a thing, and it's being held.
The more general form, for any object, then, is:

```python
def obj_properties(obj):
    return {v for v in val if (obj,) in val[v]}
```

You should find that `obj_properties('e')` gives the same result as
the one we just got by hand.

**TASK.** What are the properties of object `a`?

If we want to figure out what shape an object is, we need to find
out which of the "shape properties" it has.  There are four properties
in our model that determine the shape of something.  Let's make a set
of those.

```python
shape_properties = {'block', 'pyramid', 'table', 'square'}
```

The shape of an object is whichever of the shape properties it has.
Since `obj_properties()` returns a full set of properties, we can take
that set and intersect it with a set of possible shape options,
so that whatever is in both sets (that is, is a possible shape option and is
a property of the object) will be the shape.

```python
shapes_only = obj_properties('e') & shape_properties
print(shapes_only)
```

The result is a (singleton, we are presuming) set containing
the object's shape.  We can extract the shape from the singleton
set like this:  

```python
print(list(shapes_only)[0])
```

Having done this by hand, we can define a function to automate the
process of getting us the shape more generally:

```python
def obj_shape(obj):
    shape_properties = {'block', 'pyramid', 'table', 'square'}
    shape_property = obj_properties(obj) & shape_properties
    if len(shape_property) == 0:
        return None
    else:
        return list(shape_property)[0]
```

Try it to see if it works:

```python
obj_shape('e')
```

Almost there, let's define a function that will draw the shape
that corresponds to an object.

```python
def draw_simple_shape(obj):
    shape_map = {'square': '#', 'pyramid': 'A',
                'block': 'O', 'table': 'T', None: ' '}
    shape = obj_shape(obj)
    return shape_map[shape]
```

Make sure it does what you expect.  The following should draw an
`A` (a pyramid).

```python
draw_simple_shape('e')
```

**TASK.** Show that this will draw a block (`O`) for a block,
a table (`T`) for a table, and a square (`#`) for a square.

OK!
We can now draw the first two lines, the one with the hand and the blank
line.  Let's get to it!  We'll draw the first two lines first, and
just hard-code the main part of the world for the moment.  Because this is
just the first version of this function, I've called it `build_simple_rows_v1()`.

```python
def build_simple_rows_v1(m, g):
    held_shape = draw_simple_shape(obj_in_hand())
    top_row = '<' + held_shape
    lower_rows = [
        '   A    ',
        'OO T    ',
        '########']
    display_rows = [top_row, ''] + lower_rows
    return display_rows
```

It still draws our world.

```python
draw_simple_world(build_simple_rows_v1(m, g))
```

And if we change what is held, the thing in the hand changes.

```python
val['held'] = {('a',)}
draw_simple_world(build_simple_rows_v1(m, g))
```

Let's change it back, though.

```python
val['held'] = {('e',)}
draw_simple_world(build_simple_rows_v1(m, g))
```

Ok, now to the more important part.  How do we draw those bottom lines?
How do we derive from our model that the definition of `lower_rows`
should yield this (based on the initial setup)?

```python
    lower_rows = [
        '   A    ',
        'OO T    ',
        '########']
```

For the first row, we need to know to draw three spaces, and then a pyramid.
The reason that there are three spaces is that the things stacked on squares
1, 2, and 3 are not tall enough to reach this row.  But the things stacked on
square 4 do reach this row, and what is in this row above square 4 is a pyramid.

So, it should be clear at this point that before we build these rows, we need
to figure out what is stacked on each square so that we know how tall the stacks
are (and what the objects are in the stacks).  And in order to know what is
stacked on all the squares, we need to first be able to determine what is stacked
on any individual single square.  So that's our new sub-task.

To figure out what is on a square (and then what is on that object, and
so on), we consult the predicate "on" in our valuation function.  This is
a relation between objects that tells us what is on what.

Let's define `whats_on(obj)` to tell us what has the property of being "on"
a given object:

```python
def whats_on(obj):
    # ask: what is on the current support?
    f = nltk.sem.Expression.fromstring("on(x,s)")
    g2 = nltk.Assignment(dom, [('s', obj)])
    try:
        next_obj = list(m.satisfiers(f, 'x', g2))[0]
    except:
        next_obj = None
    return next_obj
```

This is fairly straightforward.  We set up a formula that contains two variables
(`x` and `s`) and set an assignment to point to the `obj` we are checking with `s`.
Then we ask the model for a set of the `x`es that are `satisfiers` of `on(x,s)`.
(In prose, that is asking "what are the things that are on `s`?")
The reason that we put the check for satisfiers in a `try` block is that if there are
no individuals in the domain that satisfy the formula (that is, if nothing is on this
object), then it throws an error.  By using `try...except` we can check for the error
and, if there is an error, set `next_obj` to be `None`.  As for what's happening in the
successful case...

**TASK.** Describe what is happening in the line between `try` and `except`.  Maybe
more specifically,
why did I enclose `m.satsifiers(...)` in `list(...)[0]`?

So, we now have a function `whats_on(obj)` that gives us back what object is on `obj` (or
`None` if nothing is on `obj`).  We're well on our way to constructing the stacks now,
we just need to see what's on a square, what's on that, what's on that, until we're at the
top of the stack, for each stack.

**ACTIVITY**. Try it out.  Use `whats_on()` to see what's on
square 1, square 3, square 4, and on 'd'.

Now, to construct a stack above a single square, we can start with the square,
add the object that is on it, and any on that, etc., until we have a list of objects
that represent the stack on a given square.

```python
def build_stack(square):
    stack = [square]
    while True:
        next_obj = whats_on(stack[-1])
        if next_obj:
            stack.append(next_obj)
        else:
            break
    return stack
```

Here, we start by putting the square at the bottom of the stack,
and we look for what's on the last object in the stack repeatedly until
there isn't anything more.  If `whats_on()` returns `None` it will `break` 
out of the `while True` loop and return the stack we have built.

**ACTIVITY**. Try it out, this should also work.  Just give `build_stack()` a
square and it should give you the stack on that square (including the square).
Square 4 is the only one that has more than one thing on it.

We have all the parts in place now to draw the simple world.  So now we can make
a more sophisticated version of `build_simple_rows()`, one that includes the
creation of the `lower_rows` part as well.

Let's do a couple of steps by hand first.  To begin, let us assemble the whole
set of stacks.

```python
stacks = [build_stack(s) for s in squares]
print(stacks)
```

To assemble a row (for example: row 2), we need to know what is at
the 2nd position in each of the 8 stacks.  So, let's define a function
to tell us that.  We give it a stack and a position, and it returns
the thing in the stack at that position.  The reason we would do this,
rather than just use `stack[position]`, is that we want to handle the
case where the stack ends below the current row.  By handling that case
here, we can simplify our row-building code later.

```python
def object_at(stack, y):
    if y < len(stack):
        return stack[y]
    return None
```

So to find what is in the first stack/column, first row, you can ask for:

```python
object_at(stacks[0], 0)
``` 

To see what's in the first stack/column, but in the second row, you can ask for:

```python
object_at(stacks[0], 1)
``` 

And so to build a list of all 8 things in the second row, you can do this:

```python
row = [object_at(build_stack(s), 1) for s in squares]
print(row)
```

Once we know the things in the row, we can build up the string for that row.
We make a list of the representations for each object using
`draw_simple_shape()` (which gives us `A` for pyramids,
a space for `None`, etc.).
Then we join the representations together into a single string, and return
the now-drawn row string.

```python
def row_string(row_objects):
    shapes = [draw_simple_shape(x) for x in row_objects]
    return ''.join(shapes)
```

You can try that with the `row` we defined just above, for the second row.
You should get a string starting with `'OO T '`, the row that's just above the squares.

```python
row_string(row)
```

So, let's formalize/generalize the building of a row in a function:

```python
def build_simple_row(stacks, y):
    return [object_at(stack, y) for stack in stacks]
```

You should get the same thing as we got before if we use this to gather
together the second row:

```python
print(build_simple_row(stacks, 1))
```

When we are building up the image from the bottom, we want to keep
gathering rows until we get to a row
that doesn't have anything in it.  We know that in our initial world
setup, the fourth row is such a row.
Let's build the row object for that to see what it is going to look like.

```python
print(build_simple_row(stacks, 3))
```

It has nothing but `None` values in it.  So that's how we know we're done.
If `set(row) == {None}` we can stop building rows.

And then we can finally finish up our revised version of 
`build_simple_rows()`.  The first two lines and last two 
lines below are the same as from the previous
version of the function,
but now we are actually *building* the `lower_rows` list.
Note that we build the rows up from the bottom initially, but
then draw them down from the top, so this is why when we turn
the rows of objects into rows of representations, it goes through
`reversed(rows)`, which just goes through `rows` backwards.

```python
def build_simple_rows(m, g):
    held_shape = draw_simple_shape(obj_in_hand())
    top_row = '<' + held_shape
    stacks = [build_stack(s) for s in squares]
    rows = []
    while True:
        row = build_simple_row(stacks, len(rows))
        if set(row) == {None}:
            break
        rows.append(row)
    lower_rows = [row_string(row) for row in reversed(rows)]
    display_rows = [top_row, ''] + lower_rows
    return display_rows
```

And if everything has been working correctly up to now, you can
again try to draw the world and it will appear
(but now actually based on the state of the model!).

```python
draw_simple_world(build_simple_rows(m,g))
```

Perfect. Let's leave that for now, we have enough to draw the world
in at least a simple way.  We can now move on to the grammar part.


## Talking about the world ##

Now that the representation of the world is under control, we can move
to the interpretation of English sentences.  Unlike when SHRDLU was designed
initially, we have NLTK available to make this a bit more straightforward.

```python
from nltk import grammar
```

Now we need to construct the grammar.  As before, we will do this
by defining a big multi-line string and then letting NLTK parse it into
a FeatureGrammar from the string.  But it is big and somewhat complicated, so
we will build it up in parts.

First of all, we might as well automate the simple parts, so for the simple
lexical items, we can define sets of words that will be converted into lines
in the string we will eventually parse as a grammar.

```python
nouns = {'block', 'pyramid', 'table', 'square', 'thing'}
adjectives = {'red', 'blue', 'green', 'odd', 'even'}
prepositions = {'on'}
```

Given those sets of lexical items, we can define functions that will construct
the lines of the grammar corresponding to them, as below.  Nouns and adjectives
are simple 1-place predicates, and prepositions are 2-place predicates.

```python
# NP[SEM=<\x.block(x)>] -> 'block'
def npstring(noun):
    return r"NP[SEM=<\x.{n}(x)>] -> '{n}'".format(n=noun)

# Adj[SEM=<\x.big(x)>] -> 'big'
def adjstring(adj):
    return r"Adj[SEM=<\x.{a}(x)>] -> '{a}'".format(a=adj)

# P[SEM=<\X x.X(\y.on(x,y))>] -> 'on'
def pstring(prep):
    return r"P[SEM=<\X x.X(\y.{p}(x,y))>] -> '{p}'".format(p=prep)
```

That will save us a bunch of typing (and potential typos).  We can
assemble them as follows:

```python
npdef = "\n".join([npstring(noun) for noun in nouns])
adjdef = "\n".join([adjstring(adj) for adj in adjectives])
pdef = "\n".join([pstring(prep) for prep in prepositions])
```

**TASK**. What does `npdef` now contain?

The other parts of the grammar are a bit less easy to take shortcuts on.
The first part we'll concentrate on are the noun phrases.  This was implicit
above, but the grammar we're using here is one where the subjects and objects
are "DP"s ("determiner phrases").  This is standard in theoretical syntax, even
if it isn't standard in the NLTK book.  So, something like "a block" has a
determiner ("a") and an NP ("block") that together form a DP.  We can also
add any number of adjectives between the Det and NP, so we formalize that by
saying that we can take an Adj and an NP and make an NP out of them.
The semantic representation of Adj-N is intersective: a red block is something
that is both red and a block.

```python
dpdef = r"""
DP[SEM=<?det(?np)>, DEF=?def] -> Det[SEM=?det, DEF=?def] NP[SEM=?np]
NP[SEM=<\x.(?adj(x) & ?np(x))>] -> Adj[SEM=?adj] NP[SEM=?np]
"""
```

We haven't defined the Dets yet (that's next), but they will carry a
distinction in definiteness ("a" is indefinite, "the" is definite), and
that is why above we say that the DEF feature of a DP is inherited from
the DEF feature of the Det forming it.

The semantics of the determiners is basically what we've seen before.
The only new thing here is that I added in "the", with a +DEF feature
(definite).  We will handle that later on when we try to interpret
sentences.

```python
detdef = r"""
Det[SEM=<\P Q.exists x.(P(x) & Q(x))>, -DEF] -> 'a'
Det[SEM=<\P Q.exists x.(P(x) & Q(x))>, -DEF] -> 'an'
Det[SEM=<\P Q.all x.(P(x) -> Q(x))>, -DEF] -> 'every'
Det[SEM=<\P Q.exists x.(P(x) & Q(x))>, +DEF] -> 'the'
"""
```

Now, here is a place where we are going to take a shortcut.  We are really
only going to handle one kind of sentence here, which are those where the
"verb" is "be PP".  So, like "a red block is on a table".  That means our only
verb is "be" and it doesn't mean anything (all of the meaning is carried by the
PP "on the table").  So we will define a VP that has just "is" and a PP in it,
and we will define PP in much the same way we defined transitive verbs before.

```python
ppdef = r"""
PP[SEM=<?p(?dp)>] -> P[SEM=?p] DP[SEM=?dp]
"""

vpdef = r"""
VP[SEM=?pp] -> VCOP PP[SEM=?pp]
VCOP -> 'is'
"""
```

Almost there (for now).  We need to finish this off by defining the top of
the tree.  Again following recent syntax (rather than the older style syntax
that the NLTK book follows), we are going to say that trees start at "CP"
(rather than "S").  This will be useful later when we add questions in.
For now we are going to stick to declarative statements, so we say that the
trees start with CP, and the meaning of CP comes from S through a CBAR node.

```python
# declarative S
sdef = r"""
% start CP
CP[SEM=?cbar, CT=?ct] -> CBAR[SEM=?cbar, CT=?ct]
CBAR[SEM=?s, CT=?ct] -> S[SEM=?s, CT=?ct]
S[SEM=<?subj(?vp)>, CT='dec'] -> DP[SEM=?subj] VP[SEM=?vp]
"""
```

CP has a `CT` feature, which stands for "clause type."  These are the rules
for a declarative sentence.  Before we are done we will have 'ynq' 
(yes-no question) and 'imp' (imperative) clause types as well.

That will do, so we can add all of those strings defined above together,
and then build a `FeatureGrammar` out of it.

```python
print("Assembling grammar.")
cfgdef = "\n".join([sdef, adjdef, npdef, dpdef, detdef, ppdef, pdef, vpdef])
gram = grammar.FeatureGrammar.fromstring(cfgdef)
cp = nltk.FeatureChartParser(gram)
```

At this point, you should be able to actually try it out.

```python
sent = 'a block is on the table'
parses = list(cp.parse(sent.split()))
print(parses[0])
```

To get the semantics of just the top of the tree (the truth conditions):

```python
print(parses[0].label()['SEM'])
```

And the clause type of the sentence can be retrieved by replacing `SEM` 
with `CT`:

```python
print(parses[0].label()['CT'])
```

So, is it true (in our model)?  Is a block on the table?  Well, we can ask
our model:

```python
sem = parses[0].label()['SEM']
m.satisfy(sem, g)
```

**TASK**. Try it yourself, but with two sentences that *are* true in the model.
That is, have it parse each sentence, then ask the model whether each sentence is true.


## Talking to the robot ##

Let's add some interactivity, now that the world and parser are set up.
We can add this to the end of the file.  This will print some instructions,
then wait for input, and then process the sentence.  We will start with a simple
version that we can expand a bit later.

The main thing this is going to do at first is check the truth of a sentence we give it.
So, let's first define something that will check whether a sentence is true,
following the same procedure we just did above by hand.  It takes some parses
of a sentence and checks whether the truth conditions of the first parse
are met.

```python
def check_truth(parses):
    treesem = parses[0].label()['SEM']
    return m.satisfy(treesem, g)
```

And now we can define the function that actually allows us to talk to the
robot. 

```python
def chat():
    print('Type bye to leave, type look to show the scene.')
    print()
    while True:
        sent = input("> ").lower()
        if sent == 'bye':
            break
        elif sent == 'look':
            draw_simple_world(build_simple_rows(m,g))
            continue
        try:
            parses = list(cp.parse(sent.split()))
            treetype = parses[0].label()['CT']
    
            if treetype == 'dec':
                if check_truth(parses):
                    print('That is true.')
                else:
                    print('That is not true.')
        except:
            print("Sorry, what?")
    print("Bye!")
```

Take a moment to try to understand what it is doing and then give
it a spin.

```python
chat()
```

There are two "magic" statements you can say to the robot.  One is
'bye', which will end it (`break` exits the `while True` loop);
the other is 'look', which will draw the world and then go back and
get more input (`continue` goes back up and starts the `while True`
loop again).

If it gets an error while parsing (for example, if you use a word
it does not know), then it will print "Sorry, what?" and get more input.

**TASK.** Have the robot verify that a pyramid is on the table, then
have the robot verify that a block is not on the table (by having it
tell you that "a block is on the table" is not true).

*Note*. The way `try...except` works, it will catch any error at all
that occurs in the `try` block and execute the `except` block if it
hits one.  The most obvious/relevant kind of error is one where you type a word
that the grammar does not know how to parse.  However, other errors
like typos in the program can also have this effect, and because we
are generically dealing with all errors by saying "Sorry, what?",
we lose some of our ability to debug problems.  When I was working on it
myself, I had the following as the last couple of lines, rather than the
two (`except:` and `print(...)`) above.:

```python
    except Exception as e:
        print("Sorry, what?")
        print(e)
```

What this does is puts the text of the actual error in `e` and then
prints out what `e` is.  So, if you use a word it doesn't know, it
might say `Grammar does not cover some of the input words: "'cheese'"`.
If you find that your program is saying "Sorry, what?" when you were
not expecting it to, you might want to do this as well, so you can
see what specific error it ran into.


## Questioning the world ##

It is a bit unnatural to just say things and have the robot confirm
whether it is or is not true.  It would be nicer if we could ask it.

All we need for yes-no questions is to fix up the parser so that it
understands when a yes-no question is asked.  The basic task the
program performs once the semantics is established is identical to
checking the truth of a statement (except it will say "Yes" instead
of "That is true").

The form of a yes-no question, in this limited grammar, is just "is DP PP?".
Specifically, the auxiliary "is" appears at the front instead of between
the DP and PP.

```python
# ynq S

ynqdef = r"""
CBAR[SEM=?s, CT=?ct] -> VCOP SXAUX[SEM=?s, CT=?ct]
SXAUX[SEM=<?subj(?vp)>, CT='ynq'] -> DP[SEM=?subj] VPXAUX[SEM=?vp]
VPXAUX[SEM=?pp] -> PP[SEM=?pp]
"""
```

Here, we defined a version of CBAR that has the VCOP first, and an
"S-without-an-aux" (SXAUX) that has just a DP and a "VP-without-an-aux"
(VPXAUX), which itself contains just a PP.

Then we make a new version of our grammar definition that includes this,
build a feature grammar and a parser.  The new parser is called `cp2`.

```python
cfgdef2 = "\n".join([cfgdef, ynqdef])
gram2 = grammar.FeatureGrammar.fromstring(cfgdef2)
cp2 = nltk.FeatureChartParser(gram2)
```

To make it work, you need to add the following into the
`chat()` function.

**TASK.** Define `chat2()` based on `chat()` but with the following lines
added, and with `cp` replaced by `cp2`.  If you're following along you
should be able to see where these lines go in the function.

```python
            elif treetype == 'ynq':
                if check_truth(parses):
                    print('Yes.')
                else:
                    print('No.')
```

**TASK**. Show that it worked by asking it
"is a red block on an odd square".

*Note:* You should not put a question mark at the end, the grammar will not know
what to do with punctuation.

**ACTIVITY**. Try asking it some more questions now using `chat2()`.


## Changing the world ##

After kind of a slow start, we've made a bunch of progress pretty quickly.
We now have a world, and we can state and ask things about it, and the robot
understands to the extent that it can determine the truth of statements based
on the facts of the world.

The last major thing we'll do is add the ability to change the configuration of
the world, by adding imperatives to the parser.  This is also where we have to
add a lot more "smarts" to the system, because although NLTK was able to take
care of the heavy lifting in the domain of parsing and evaluation of logical
formulae, we need to tell the robot how to actually affect the world.

The first thing we need to do is revise the grammar to have one more clause
type, an imperative.  An imperative has a silent (implicit) subject, but
is otherwise mostly the same as a statement.  So, all we really need to do
is add one more alternative definition of S:

```python
# imperative S

impdef = r"""
S[SEM=<?vp(hand)>, CT='imp'] -> VP[SEM=?vp]
"""
```

As you can
see, what it essentially does is looks for a sentence with no subject, and
assumes that "hand" (the robot's actor) is the subject, and sets the clause
type to 'imp'.

This is not quite enough to be satisfying, though, we need to add some verbs.
We are going to add just two verbs: *take* and *put*.  What we want the
robot to do if we tell it to *take* something is to put it in its hand, and
if we tell it to *put* something somewhere, it will do that.

Although we'll go ahead and fill in the semantics here, we actually are
not going to wind up using a lot of the semantics it computes.  But, here
is a little more to add.  We are adding a transitive verb "take",
which is very much like the transitive P "on", and a ditransitive
verb "put" that has both a DP and PP argument. 

```python
vpactdef = r"""
VP[SEM=<?obj(?v)>] -> V[SEM=?v] DP[SEM=?obj]
V[SEM=<\x y.take(y,x)>] -> 'take'
VP[SEM=<?v(?obj,?pp)>] -> VDT[SEM=?v] DP[SEM=?obj] PP[SEM=?pp]
VDT[SEM=<\Y X x.X(\z.Y(\y.put(x,y,z)))>] -> 'put'
"""
```

We can assemble the grammar (now at iteration 3) as follows:

```python
cfgdef3 = "\n".join([cfgdef2, impdef, vpactdef])
gram3 = grammar.FeatureGrammar.fromstring(cfgdef3)
cp3 = nltk.FeatureChartParser(gram3)
```

At this point it will already be able to parse
"put a red block on an even square"
but the robot doesn't know how to make the command happen.

And this is where things get a little hairier, because there are a lot
of things to take into account.  To begin, let's add something in the
input processor that notices when an imperative sentence is used and
calls a function (which we have not yet defined) to handle it.

**TASK.** Define a function `chat3()` based on `chat2()` that has the
following lines added in the appropriate place, and has changed the parser
from `cp2` to `cp3`. 

```python
            elif treetype == 'imp':
                process_command(parses)
```

And now we need to define `process_command()`.
The `process_command()` function is supposed to look at the imperative
tree, determine what verb was used (*take* or *put*), and then take the
action.  We're going start with *take* because it's simpler.

Think first about what the robot is supposed to do if it is asked to
take something.  If you say "take a block" then there are several
possible choices.  The robot can just pick one.  However, you can't
take something that's underneath something else, so it needs to check for
that.  Also, taking something means putting it in its hand, but if it
is already holding something, it needs to put that thing down first.

Let's try a couple of things by hand before we try to automate anything.
First, we will parse the sentence "take a red block" by hand:

```python
parses = list(cp3.parse('take a red block'.split()))
print(parses[0].label()['CT']) # should say 'imp'
print(parses[0])
print(parses[0][0])
print(parses[0][0][0])
```

As you can see, we're kind of working our way down the tree.  We can
get the VP like this:

```python
vp = parses[0][0][0][0]
print(vp)
print(vp[0])
print(vp[0][0])
```

After a whole lot of `[0]`s, we managed to zoom into the actual verb.
Because we have a very limited grammar, this is reliable enough.  That is,
we can count on being able to figure out what verb we have by looking at
`vp[0][0][0]`.  So, in particular, we can check to see if it is "take".

Likewise, we can get the object (the thing we're asking the robot to take)
with this:

```python
obj = vp[1]
print(obj)
```

For the object, we actually do need to use the semantics NLTK computed
for us.  Specifically, we need to figure out what individuals in the
model the object can describe (so we know what the robot is choosing
between when it takes something).

Here, I wound up doing something a bit tricky to get this into a form
that we can use NLTK's evaluation functions for.  The logical formula for
the object is:

```python
obj_sem = obj.label()['SEM']
print(obj_sem) # \Q.exists x.(red(x) & block(x) & Q(x))
```

This is the right form for the subject of a sentence, it takes a
predicate (`Q`) and is true if there is an `x` such that `x` is red,
a block, and `Q` is true of it.  But our goal right now is not to
evaluate a sentence, but to find out what the objects are that this
DP can represent.  We want to know what the red blocks are.
After some reflection, the simplest thing I could come up with is
to make `Q` is kind of predicate that means "is y".
So, what we're going to be asking is: What are the values of y
that make "there is a red block that is y" true?  It's a little
bit roundabout.  (If you see a better way to do this, let me know!)

So, to implement this, we will define a predicate I'll call
`isthis`:

```python
isthis = nltk.sem.Expression.fromstring(r'\x.(x=y)')
```

This is true for any individual that is... this, whatever y is pointing to.
So, if we substitute `isthis` in for `Q`:

```python
this_obj = nltk.sem.ApplicationExpression(obj_sem, isthis).simplify()
print(this_obj) # exists x.(red(x) & block(x) & (x = y))
```

We now have an open formula (on `y`) that we can check for satisfiers on.
As I indicated, this is wandering around in the weeds a bit, but the main
thing is that if we say:

```python
options = m.satisfiers(this_obj, 'y', g)
print(options) # {'a'}
```

we can get a list of those individuals that are red blocks.

Having walked through that, let's just commit that to a function that will
do all of that and give us a set of options. 

```python
def find_options(obj):
    obj_sem = obj.label()['SEM']
    isthis = nltk.sem.Expression.fromstring(r'\x.(x=y)')
    this_obj = nltk.sem.ApplicationExpression(obj_sem, isthis).simplify()
    options = m.satisfiers(this_obj, 'y', g)
    return options
```

To make sure it worked, you can see if you get `{'a'}` for this:

```python
print(find_options(obj))
```

We still haven't defined the `process_command()` function (and so the robot
can't yet handle imperatives), but we now know better what it should look like.
It should find the VP, then the verb, and if the verb is "take", it should find
the possible objects it could be referring to.  The structure of the function
is like this:

```python
def process_command(parses):
    vp = parses[0][0][0][0]
    v = vp[0][0]
    if v == 'take':
        obj_opts = find_options(vp[1])
        if len(obj_opts) == 0:
            print('There is no such object to take.')
        else:
            # do it
            print('out of order.')
    else:
        print("I'm unsure what you are asking me to do.")
```

Of course, we want to replace the "out of order" message with some actual
action.  But at this point, `chat3()` should at least run, and give you an
"out of order" message if you try to "take a red block", and a
"there is no such object to take" message if you try to "take a blue table".

Now, let's consider all the possible scenarios, assuming that there are
some options available for what object it could be taking.

If there are several options, it needs to pick one.  If it already has
something in its hand, it might be what it needs, in which case it
doesn't need to do anything.  But if it has something different
in its hand, it needs to put that down first, somewhere.
And it can't pick something up that's underneath something else.

It's surprisingly complicated once we start involving the world.

Since the world is designed in such a way that it has more squares
than other objects, the robot is guaranteed to always have a place
to put down the object it is holding onto an empty square.
We can define a function
that will locate an empty square, and be confident that it'll find one.

```python
def empty_square():
    for square in squares:
        if not whats_on(square):
            return square
```

If you execute this function, it should find an empty square.
In fact, it should find `s3` at present.

And we can define a function that will remove from the options any
objects that are not visible (because they are hidden under something
else).

```python
def just_visible(objects):
    return {obj for obj in objects if not whats_on(obj)}
```

Since the robot can't pick up the floor squares, we also want to
eliminate those from the options of pick-up-able things.  We don't
want them to be invisible/hidden (because we can put things on them),
but we can add an extra filter to remove any squares.  Since
invisible things are also not takable, we will incorporate the
visibility filter in `just_takable()` as well:

```python
def just_takable(objects):
    return {obj for obj in just_visible(objects) if 'square' not in obj_properties(obj)}
```

Now, to put an object down, the robot will put the object on another
object.  For right now, it will just put the object on an empty square.
This means that we remove the object from the robot's hand, and we put
it in an `on` relation to another object.  So, here is a function that makes
this happen.

```python
def put_on(target):
    obj_to_place = obj_in_hand()
    val['held'] = {}
    val['on'] |= {(obj_to_place, target)}
```

I used a kind of nifty little construct above in the last line there.
The `|=` operator is kind of like `+=`, it's a shorthand.
So the last line is equivalent to

```python
    val['on'] = val['on'] | {(obj_to_place, target)}
```

Since both `val['on']` and `{(obj_to_place, target)}` are sets,
the `|` operator performs a set union, meaning that it adds all of members
together and puts them in `val['on']`.  Effectively, it is adding one more
"on" relation to the model.

Lastly, to pick an object up, we put it in the robot's hand, and remove
the `on` relation that indicated it was on top of something.

```python
def pick_up(obj):
    val['held'] = {(obj,)}
    val['on'] = {(x,y) for (x,y) in val['on'] if x != obj}
```

Those are all the pieces we need to implement *take*.  There is one
more consideration however, thinking ahead.  We are going to want to
implement *put* next, and conceptually, *take* is part of *put*.
That is, in order for the robot to put something somewhere, it first
needs to take that thing.  So, to allow both *take* and *put* to use
the "taking" script, we will define a function that implements *take*.
We will give it a set of objects that meet the description, and it will
take it from there.

Describing the function in English is probably not much clearer than
just reading the code.  But: if there are no options, it prints a
message saying so.  If there are options but one is already in hand,
print a message saying so and report success (`True`).
Otherwise, narrow the options to the takable objects (those that
are not squares and not underneath something).  If that leaves no
options, print a message saying so.  Otherwise, put the thing
currently in hand down on an empty square, pick up the first of the viable
options, and report success (`True`).  If we haven't already reported
success, then report failure (`False`).

```python
def do_take(obj_opts):
    if len(obj_opts) == 0:
        print('There is no such object to take.')
    else:
        if obj_in_hand() in obj_opts:
            print('I am already holding such a thing.')
            return True
        else:
            takable = just_takable(obj_opts)
            if len(takable) == 0:
                print('I cannot see such a thing to take.')
            else:
                put_on(empty_square())
                # take the first option
                obj_to_take = list(takable)[0]
                pick_up(obj_to_take)
                print('Object taken.')
                return True
    return False
```

We can then replace the "out of order" message in `process_command()` with `do_take()`.


```python
def process_command(parses):
    vp = parses[0][0][0][0]
    v = vp[0][0]
    if v == 'take':
        obj_opts = find_options(vp[1])
        if do_take(obj_opts):
            draw_simple_world(build_simple_rows(m,g))
    else:
        print("I'm unsure what you are asking me to do.")
```

**ACTIVITY**. Play around with it a little by running `chat3()`.
It's kind of fun.  Take a block.
Take a red block.  Maybe it's not all that fun.  But it's a little bit fun.

The next thing to implement is *put*, but we already have a lot of those
pieces in place just from *take*.  In order to put something somewhere, the
robot must take it first and then put it in the target location.  Now the
target need not be a floor square, so we can make stacks of things.

So, first of all, we just execute *take* on the object we are supposed to
be putting somewhere.

If the verb is *put*, then we need to determine whether we can take
the object, and then we need to determine whether the place we want to put
it is visible.  And then we do it.  So, we add the following into
`process_command()` in the appropriate place.

**TASK.** Redefine `process_command()` in order to put the following
in the appopriate place.

```python
    (...)
    elif v == 'put':
        obj_opts = find_options(vp[1])
        if do_take(obj_opts):
            pp = vp[2]
            p = pp[0][0]
            loc_opts = find_options(pp[1])
            if len(loc_opts) == 0:
                print('There is no such place to put anything.')
            else:
                visible = just_visible(loc_opts) - {obj_in_hand()}
                if len(visible) == 0:
                    print("I cannot see such a place to put anything.")
                else:
                    # pick the first option
                    target = list(visible)[0]
                    put_on(target)
                    print('Object placed.')
                    draw_simple_world(build_simple_rows(m,g))
    else:
    (...)
```

Now you should be able to run `chat3()` and start putting things on
other things.  Like "put a red block on a pyramid".

At this point, since you are changing the world around, you might want to
be able to just reset it to its initial state.  If you find you want to
do that, you can define and use this:

```python
def reset_world():
    val['on'] = {('a','s1'),('b','s2'),('d','s4'),('c','d')}
    val['held'] = {('e',)}
```


## Improving the view and adding labels ###

Let's go back to the display.  Right now the display is very small,
and we don't know which objects are red, etc.  So, we can go back
to the world-drawing code and change it around a little bit so that the
objects are bigger shapes with labels on them.  We won't use graphics,
but we'll instead fashion "text graphics"
that represent the shapes, with a label that indicates something about the
properties the object has.  So, before we can construct the whole image of
the world, we need to be able to draw an individual block, pyramid, etc.

Conceptually, what we're going to do is replace the `draw_simple_shape()`
function that we were using before with a `draw_shape()` function that
draws larger, 8x4-character objects.

Here's the basic idea in a simple version before we do the whole thing:

```python
def draw_shape_v1(obj):
    s = r"""
/------\
|      |
|{: ^6}|
\------/
""".format('label!')
    return s.split("\n")[1:-1]
```

To make it easier to work with, it defines the shape as a multi-line string
using the `r"""` delimiter.  The `r` (for "raw text") is important there, because we are using
the `\` character, and we do not want it to be interpreted as "escaping" the next
character.  Notice too that we have to put the shape all the way on the left
(not indented) or else it will include a bunch of leading spaces we do not want to have.
So, we draw the shape we want, including a format string for the label in the
third line, and then insert the string "label!" in there.  The format string says that
the inserted string should be 6 characters long (minimally), and padded with spaces on either
side in order for it to be centered.  Once the multi-line string is defined in `s`, it is
then split on line breaks (`\n`), and then the first and last lines are discarded.

**ACTIVITY**. Try it out, see what `draw_shape_v1('a')` gives you.

If you want to see the shape actually drawn:

```python
print("\n".join(draw_shape_v1('a')))
```

With the proof of concept out of the way, we can now make a more sophisticated
`draw_shape(obj)` function that will check to see what type of object `obj` is, 
draw the right shape for the object, and fill in the label based on the other
(non-shape) properties.

One of those properties (block) is relevant to knowing what shape we are going
to draw, and the rest of the properties will be used for the label.  So, let's
also define a function that takes the properties and separates out the shape property,
then builds a label with the rest of the properties by using the first two letters of
at most three of them (since we have only 6 characters) to work with.  (So, a big red
block would be a block carrying a label of "bire" or "rebi".)

```python
def obj_form(obj):
    shape_properties = {'block', 'pyramid', 'table', 'square'}
    shape_property = obj_properties(obj) & shape_properties
    if len(shape_property) == 0:
        return None
    shape_name = list(shape_property)[0]
    properties = obj_properties(obj) - {'held', 'thing', shape_name}
    abbrevs = [prop[:2] for prop in properties]
    label = "".join(abbrevs)[:6]
    return(shape_name, label)
```

So, what's happening here?  We retrieve the object's properties, and then find
which shape property the object has (by intersecting the object properties
with a set of shape properties).  We want to disregard the "held" property
(which is true of the object that is currently in the robot hand), because
we don't want that as part of the label.  Same for "thing", which is a
property of all of the shapes, and for the shape property itself.
We then take the rest of the properties and
extract the first two letters of each property name into a list, join them
together, cut it off at 6 characters (in case it would have been longer).
We then return a pair, the first member of which is the shape property, and
the second member of which is the label.  If there is no shape property, it
returns `None`.

**TASK**. Try this out.  What is the shape and label of object `a`?

Now, we're all set to finish up the `draw_shape()` function so that it
can draw more than just a block.  There are two special shapes that we
want to be able to draw apart from blocks, pyramids, squares, and tables:
the robot hand, and an empty space.  So, we're going to set up
`draw_shape()` to respond to a couple of "magic" values.  If we send it
`"HAND"` as the object, it will draw the hand, and if we send it `None`
it will draw an empty space.  Otherwise, it will look up the object
in the model and get its properties and label.

If you type this in (rather than copying and pasting it), be careful
in particular about the pyramid and in the line under the robot hand.
There are spaces after the right edge, and under the hand,
since every line in the multi-line string needs to be 8 characters long.

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

To make sure it worked, try `draw_shape('a')` and `draw_shape('HAND')`.

Now, we want to integrate this into our drawing procedure.  Most of the
previous `build_simple_rows()` remains logically unchanged, except that when
it comes time to translate the rows into display strings, we have 4x8 lists
of strings instead of a simple character.

So, we replace `row_string()` with `row_tiles()`:

```python
def row_tiles(row_objects):
    return [draw_shape(x) for x in row_objects]
```

And then `build_rows()` that makes use of `row_tiles()`.

```python
def build_rows(m, g):
    stacks = [build_stack(s) for s in squares]
    rows = []
    while True:
        row = build_simple_row(stacks, len(rows))
        if set(row) == {None}:
            break
        rows.append(row)
    rows += [[None], ['HAND', obj_in_hand()]]
    display_rows = [row_tiles(row) for row in reversed(rows)]
    return display_rows
```

And then instead of `draw_simple_world()`, which just printed each row
string, we want to print each row group (since each representation is now
4 rows tall).

```python
def draw_world(rows):
    for row in rows:
        for line in range(4):
            print(''.join([col[line] for col in row]))
```

And now to draw the enhanced world, you can just do this:

```python
draw_world(build_rows(m,g))
```

The last step is to define `chat4()` and `process_command()` to use
`draw_world(build_rows(m,g))` instead of `draw_simple_world(build_simple_rows(m,g))`.
Once you've done that, you can run `chat4()` and play around fairly effectively
in this block world.  You can now take a red block, put a red block on
a table, ask if a red block is on an odd square, or check the truth of
a red block is on an odd square. 


## Being definite with history  ##

There are a bunch of things that would be neat to add, and are kind of
within reach.  For example: the real SHRDLU had a planning module, so that if you
said you wanted to pick up something that was covered, it would move
things out of the way so it could get it.  That would be relatively easy to
implement.  We could also add *wh*-questions, to handle things like
"what is on the blue pyramid" or "how many blocks are on even squares" or
"where is the blue pyramid".  Our support for the universal quantifier is
a bit incomplete as well; you can ask "is every block on an even square"
but you can't "put every pyramid on a block".  SHRDLU also had the ability
to learn things like that a stack of a block and pyramid can be called a
"steeple" (and then "steeples" can be referred to from then on), and could
absorb facts like "I like red blocks" and then answer questions about them.
Another easy-ish fix we could add is to deal with the fact that our robot
only knows singulars, not plurals.

So, though this is already kind of sophisticated, there are still a lot of places it
can be expanded even just with respect to this blocks world.  However, it also feels like
we could get pretty close to being able to handle almost anything a person
asked the robot about pertaining to the blocks world.

One (nearly final) thing that SHRDLU can do that we'll approximate here is handling
definite noun phrases.  At the moment, our robot treats "the pyramid" and
"a pyramid" exactly the same (except that it marks "the pyramid" as definite).
But if we ask the robot to do something with "the block" it should be unsure
which block we meant.  Unless we had just been interacting with a block
recently, in which case we can assume that it was that block we meant.

So, to handle this, we will keep track of the last five objects we
did something with, and if that disambiguates sufficiently for a definite
noun phrase, the robot will not complain.

We will keep track of recent objects in a list (in the global space)
called `recent_objects`, and every time we interact with an object we
call `add_recent()` to add the object to the recently addressed objects
(rolling off the oldest ones in the process, to keep it to 5).

```python
recent_objects = []

def add_recent(obj):
    global recent_objects
    recent_objects = ([obj] + recent_objects)[:5]
```

We haven't been very careful about distinguishing between local and global
scope for variables, but in `add_recent()` above we need to add the
`global recent_objects` line in there to keep Python from assuming that
the definition within is supposed to hold just within the function. 

Then, in `do_take()` we add `add_recent(obj_to_take)` after it is picked up,
and in the "put" section of `process_command()` we add `add_recent(target)`
after it an object put on the target.

**TASK**. Do that, by defining `do_take_mem()` based on `do_take()`,
`process_command_mem()` based on `process_command()`. 

I was slightly less specific about where these go,
but if you're still following along it shouldn't be too hard to get those
`add_recent()` lines in the right place.

The place where this will matter is in `find_options_mem()` where it is finding
the possible options for a referring description.  We can insert a check for
a definite DP before the `return` line:

```python
def find_options_mem(obj):
    obj_sem = obj.label()['SEM']
    id_obj = nltk.sem.ApplicationExpression(obj_sem, isthis).simplify()
    options = m.satisfiers(id_obj, 'y', g)
    if obj.label()['DEF']:
        if len(options) != 1:
            recent_options = options.intersection(set(recent_objects))
            if len(recent_options) != 1:
                print("I don't know which one you mean.")
                options = {}
            else:
                options = recent_options
    return options
```

**NOTE.** There was a typo above earlier, it said `idfun` instead
of `isthis` in the definition of `find_options_mem()`. 

As a last step, define 
`chat5()` based on `chat4()` that makes use of the `_mem`
versions of the functions we just defined.

**ACTIVITY.** Try it out.  It should complain if you say "take the pyramid",
but if you say "take the green pyramid" and then
"put the pyramid on the table" it should
not complain (and, moreover, it should
put the green pyramid (back) on the table).

## Pyramids are pointy ##

This one I'm not going to give you explicit guidance on,
but your final challenge is:

**TASK**.  Modify the program so that if prevents you from putting anything
on a pyramid.

