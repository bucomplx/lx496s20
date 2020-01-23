---
layout: single
title: Texts
permalink: "/hw7-semantics/"
---

<div class="title-table clear-table center-table" markdown="1">

| :--------    | :--:        | -:           |
| CAS LX 394 / | NLP/CL      | Homework 7   |
| GRS LX 694   | Spring 2019 | Due WED 4/17 |

</div>

### Note about the due date ###

**We meet on Wednesday next week.**



## Formalizing meaning ##

### First part, mostly just reviewing from class ###

This largely follows the discussion from chapter 10 in the NLTK book, but I will try to elaborate
on it here somewhat.  Also, we went through much of this first part in class.

However, still do the tasks, even if you kind of "got the answers" in class.
This will contain a bit more by way of exercises, to help make it clearer what the concepts are
here.  We will start by creating a little world that we can evaluate sentences against.

In this world, there are four people: Andrea, Bobby, Chris, and Dana.  These are our *individuals*.
An "individual" need not be a person, it's just some kind of entity that we can refer to.
So, let's also add a couple of non-human individuals as well.  To keep things somewhat simple, they
will be the Moon and the Sun.  We are going to pretend their names are `the_sun` and `the_moon`
however (having a space in there makes things not work, so we will use `_` instead of a space).

So, step one, let's define a set of our individuals.  (There is no intrinsic order to these
individuals, they are just the individuals in our model of the world, so it should be a set and
not a list.)

```python
dom = {'a', 'b', 'c', 'd', 'm', 's'}
```

Now we will build up some information about how English maps to these individuals.
First off, we will set up the names.  As we did in class, and as it is done in the textbook, we
will use the `fromstring` function to create these, because it's easier to type.  What we do is
set up a multi-line string first, using then `"""` delimiters on each end, and then we will
create a Valuation by parsing it.

```python
import nltk
names = """
andrea => a
bobby => b
chris => c
dana => d
the_sun => s
the_moon => m
"""
val = nltk.Valuation.fromstring(names)
print(val)
```

Now that we have done this, we can "evaluate" the English words and get their referents.

```python
print(val['bobby'])
print(val['the_moon'])
```

This tells us which individual in our set of individuals is being referred to by "bobby" and by "the moon".
Doing this counts, at least in a certain sense, as translating from English into "semantics."  We are
determining the meaning of the word "bobby," for example.

The individuals in this world have properties and relationships, however, as well.  For example, some of the
individuals are people.  So, we define "person" as being something that holds of the individuals
a, b, c, and d.  For the moment, we are going to create a new Valuation to hold this information,
and we will merge these together shortly.

```python
valp = nltk.Valuation.fromstring("person => {a, b, c, d}")
print(valp)
```

It's kind of a pain to keep typing this `nltk.Valuation.fromstring` thing, let's give it a shorter
name.  I'm going with `vfs` for "valuation-from-string":

```python
vfs = nltk.Valuation.fromstring
```

Now, the sun and the moon are not people, what are they?

```python
valsb = vfs("spaceball => {s, m}")
```

So far, we have three different Valuations (`val`, `valp`, and `valsb`).  But we need to merge
them together into one.  It is possible to combine two Valuations using `update`.  So, let's
add `valp` and `valsb` to `val`:

```python
val.update(valp)
print(val)
val.update(valsb)
print(val)
```

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

The `update` function is actually fairly general.  It is defined for Valuations, but it is
also defined for just regular sets, as well as for dictionaries.  If you call `update` on a
set or a dictionary, it merges the argument of `update` into it (with priority given to the
additions, if there is a conflict).

```python
aset = {1, 2, 3}
aset.update({3, 4, 5})
print(aset) # => {1, 2, 3, 4, 5}
adict = {'a': 1, 'b': 2}
adict.update({'b': 4, 'c': 6})
print(adict) # => {'a': 1, 'b': 4, 'c': 6}
```

</div>

Now, all of our world-building work to date is represented in `val`.  Let's do a little bit more building.
It turns out that Andrea and Bobby are from Boston, while Chris and Dana are from Cambridge.

```python
val.update(vfs("bostonian => {a, b}"))
val.update(vfs("cantabrigian => {c, d}"))
```

Now, we've defined the mapping between names and individuals, and we've defined some nouns/predicates
that hold of sets of individuals.  What remains is to define some relationships between them.
Relationships are asymmetrical, so just because Andrea likes Bobby does not mean that Bobby likes Andrea.
But let's start with that.

```python
val.update(vfs("likes => {(a, b)}"))
```

Ok, now let's (attempt to) make it mutual.

```python
val.update(vfs("likes => {(b, a)}"))
print(val['likes'])
```

Hmm.  That didn't really work.  Instead of making Andrea and Bobby like each other, Bobby started
liking Andrea and Andrea stopped liking Bobby.  This simply replaced the liking pair, rather than
adding to it.  So, we could spell this out fully like this:

```python
val.update(vfs("likes => {(b, a), (a, b)}"))
print(val['likes'])
```

And that gets what we want.  But it would be nice to be able to add relations in stages, rather
than redefine it in full every time.  And you can, with a bit of trickery.  The code below will
add the fact that Chris likes Dana to our model of the world:

```python
val['likes'].update(vfs("x => {(c,d)}")['x'])
print(val['likes'])
```

**Task 1.** Why does that work?  Explain how this added `('c', 'd')` to `val['likes']`.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

We could have just spelled it out (essentially performing the `fromstring` operation ourselves).
The code below has the same effect as the code above.  Arguably more simply.

```python
val['likes'].update({('c', 'd')})
```
However, using `Valuation.fromstring` means that we can just type `(c,d)` instead of `('c', 'd')`
(and the latter could get annoying if we're adding several relations at once).

Similarly, if we wanted to make Bobby a spaceball, we can just do this:

```python
val['spaceball'].update(vfs("x => {b}")['x'])
```

instead of making the 1-tuple with a string in it by hand, as in:

```python
val['spaceball'].update({('b',)})
```

You are free to believe that I have introduced a more complicated way to do a relatively
simple thing.  But wait until

</div>

**Task 2.** Finish setting up `val['likes']` to represent the following world situation:
Andrea likes everyone; everyone likes Dana; Bobby likes Andrea; Dana likes Chris;
Andrea and Bobby like the Sun; Bobby and Chris like the Moon.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

That is, start with what we already have for `val['likes']` at this point, and add the rest in.
You can use whatever method you want, including just redefining it completely from scratch. Just
provide me with whatever command(s) you used to do it.  What we start with for `val['likes']` is:

```python
{('a', 'b'), ('b', 'a'), ('c', 'd')}
```

Also: Assume that if Andrea likes everyone, Andrea also likes Andrea.

</div>

That's complex enough that it's probably worth checking to see if you wound up with what you
intended to.  You can `print(val['likes'])` but it's a long list (set) of pairs, that isn't
necessarily in a helpful order.  So, let's see if Andrea likes everyone (that is, all the people).
If you type the following, you should get `True` if Andrea likes all the people.  (If you get `False`
then you probably set up your world wrong, double check Task 2.)

```python
not False in [(val['andrea'], x) in val['likes'] for (x,) in val['person']]
```

Got `True`? Great. But why? If you just blindly typed it in and got `True` without figuring
out what it is doing, that's fine.  But now we're going to figure out what it is doing.

First of all, remind yourself what `val['andrea']`, `val['person']`, and `val['likes']` are:

```python
print(val['andrea'])
print(val['person'])
print(val['likes'])
```

We are trying to determine whether Andrea likes all the people.  So, we check, for each person,
whether it is true that Andrea likes that person.  When we're done checking people, we should not
have found any that yield `False`.  As you just saw, `val['person']` is a set of 1-tuples, like `('a',)`.
So to go through the people, we want to use `for (x,) in val['person']` in order to set `x` to be
the individual in our domain that corresponds to the person (e.g., `'a'`).  To determine whether
Andrea likes the person in `x`, we need to find out whether the pair that has Andrea as the first
member and `x` as the second member is in the set of "likings" in `val['likes']`.  The individual
that Andrea represents is `val['andrea']` (which will be `'a'`).  So, we evaluate whether the
pair `(val['andrea'], x)` is in `val['likes']`.  The expression `(val['andrea'], x) in val['likes']`
will be `True` if Andrea likes `x` and `False` otherwise.  The list that this list comprehension
builds will be a list of `True` or `False` values (one for each person).  If Andrea indeed likes every person, then the
list should be `[True, True, True, True]`.  Finally, we check to see if `False` is anywhere in that
list.  If it is, we failed: Andrea doesn't like every person.  If there is no `False` in there,
then we succeeded.  So, `not False in [...]` is `True` if we succeeded.

**Task 3.** Use the same technique to verify that every person likes Dana.

Now, let's formalize our model of the world into an official NLTK model.  A model is just a
pairing of a domain and a Valuation function.

```python
m = nltk.Model(dom, val)
```

Once we have a model defined, we can use the model's `evaluate` function to test the truth
of things in the model.  In order to use `evaluate` we also need to set up an "assignment function"
(which can be thought of as a record of who we're pointing to).  To begin with, we'll just set up
an empty assignment function (we aren't pointing at anything).

```python
g = nltk.Assignment(dom)
```

Now, we can verify that Dana likes Chris, and verify that Bobby does not like Chris, like so:

```python
print(m.evaluate('likes(dana, chris)', g))
print(m.evaluate('likes(bobby, chris)', g))
```

**Task 4.** Use `evaluate` to verify that Dana does not like Bobby, and that Chris likes the Moon.

We can also use quantifiers like `all` and `exists` with `evaluate`.  For example, we can re-verify
that Andrea likes every person, like so:

```python
print(m.evaluate('all x.(person(x) -> likes(andrea, x))', g))
```

The way this works is pretty much exactly how our home-spun version from Task 3 worked.  It goes through
all of the individuals in the domain one by one, and for each it checks to see if it's a person,
and if it is a person, then it checks to see if it is the second member of a pair, whose first member
is Andrea, that can be found in the list of "likings".

**Task 5.** Use `evaluate` to verify that everybody likes Dana.

You can also use `exists`, which is true if the condition is met for at least one of the individuals in the
domain.  So, if we want to ascertain that at least *somebody* likes Bobby, we can do the following:

```python
print(m.evaluate('exists x.(person(x) & likes(x, bobby))', g))
```

What that means is that we can find *some* `x` in our domain `dom` such that `x` is both a `person` and
in a `likes` relation with Bobby.

**Task 6.** Use `evaluate` to verify that every Bostonian likes the Sun.

**Task 7.** Use `evaluate` to verify that no spaceballs are from Cambridge.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

You can either use `exists x.(...)` and look for `False` as an answer, or you can use `-exists x.(...)`
and look for `True` as an answer. The meaning of `-exists x.(...)` is: 'it is not the case that there exists
an x such that...'

</div>

The string that we give to `evaluate` is first interpreted as a "semantic Expression"
built from a string.  If we don't want to evaluate immediately, we can define such expressions
directly.  The function that does this is `nltk.sem.Expression.fromstring`.  Like before, we'll
give it a shorter name (`sfs`) to save on some typing.  Then we'll define a formula `f1` to be
"x likes the Moon".

```python
sfs = nltk.sem.Expression.fromstring
f1 = sfs('likes(x, the_moon)')
print(f1)
```

So, is "x likes the Moon" true?  No idea.  We can't decide that until we know who `x` is
supposed to be.  Once we know who `x` is, then we can figure out whether it's true.  Because
we don't know who `x` is, `x` is considered a "free variable."  Although it's kind of obvious,
we can interrogate `f1` to ask it what its free variables are:

```python
print(f1.free())
```

If we want to know who likes the Moon, we can ask the model to tell us which individuals,
when substituted in for `x`, would make `f1` true:

```python
print(m.satisfiers(f1, 'x', g))
```
**Task 8.** Use `satsifiers` to determine who/what Chris likes.

One way that we can set a value for `x` is to use `x` to point to an individual.  That is,
suppose we point (with our "x" finger) at Bobby, and then ask whether "x likes the Moon"
is true.  Since this tells us who `x` is (namely, Bobby), we can decide whether "x likes the Moon"
is true.  It's true if (and only if) Bobby likes the Moon.

This is what the assignment function is for.  It is a record of who/what we are pointing at,
and with which fingers.  (This is really designed to handle pronouns like *he*, *she*, *it*.
If you use those pronouns, it is assumed that something in the discourse is basically pointing
at the individual you mean.  Without some kind of pointing ("deixis") you won't be able to
interpret the referent of a pronoun.)

### Parsing sentences ###

Let's try to build a little grammar that can take sentences and interpret them.  What we
want to do here is create some phrase structure rules that will apply the semantics we
defined to a syntactic structure.  We'll build this up from the bottom.

As a first step, we will define the NPs, which will be just the names we have.
(We are going to build a big multi-line string and then create the grammar using a
`fromstring` function.)

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

What this means is that if the English word 'andrea' is encountered, that can be
interpreted as an NP with the SEM feature being `<andrea>`.  And likewise for the other
proper names.

As for how the whole tree combines, it will start with `S` at the top, which is
formed from an `NP` and a `VP`, and the `VP` is formed from a `V` and an `NP`.
For now, that's all we'll do.

What we want is for the semantics of the VP to combine the semantics of the V
with the semantics of the NP.  So, if the V is "likes(x, y)", and the NP is "bobby",
then we want the VP to be "likes(x, bobby)", more or less.

```python
cfgdef = r"""
% start S
S[SEM=<?vp(?subj)>] -> NP[SEM=?subj] VP[SEM=?vp]
"""
```

The way to understand this is: The semantics of S is the function that
we get from the semantics of VP, applied to the argument that we get from
the semantics of the NP subject.  So, by saying `NP[SEM=?subj]` we are
naming the value of the NP's `SEM` feature (whatever it is) as `?subj`.
We name the value of the VP's `SEM` feature (whatever it is) as `?vp`.
We assume that `?vp` is a function that can take `?subj` as an argument.
And so, the `SEM` feature that we assign to S is whatever we get when
we apply the function `?vp` to the argument `?subj`.

We then do the same thing for the VP.  We assume that the V is going
to be a function that we can apply to the NP.

```python
cfgdef += r"""
VP[SEM=<?v(?obj)>] -> V[SEM=?v] NP[SEM=?obj]
V[SEM=<\y.\x.likes(x,y)>] -> 'likes'
"""
```

So, now we can add in the NP definitions we did at the beginning, and
take a look at the whole grammar.

```python
cfgdef += npdef
print(cfgdef)
```

Now that we have the definition, we can parse it into an actual grammar
that NLTK can use, and then connect it to a parser (we will use the one
called `FeatureChartParser`).

```python
from nltk import grammar
gram = grammar.FeatureGrammar.fromstring(cfgdef)
cp = nltk.FeatureChartParser(gram)
```

And now we can parse some sentences.  Let's start with "bobby likes chris":

```python
parses = list(cp.parse('bobby likes chris'.split()))
print(len(parses))
print(parses[0])
```

If everything worked up to now, you should see that there is 1 parse,
and `print(parses[0])` will show you the parse it got.

The very first line is the overall semantic value for the tree, which we
can get like this:

```python
treesem = parses[0].label()['SEM']
print(treesem)
```

And, now that we have this expression, we can test it against the model
to see if it is actually true.  Note that we are using `satisfy` and not
`evaluate` -- the `evaluate` function takes a string and turns it into a
semantic expression, and then calls `satisfy`.  Since we already have a
semantic expression, we can just call `satisfy` directly.

```python
print(m.satisfy(treesem, g))
```

And thus we learn that, in this model, Bobby does not like Chris.

If we want to know if Bobby likes Dana, we just change the sentence.

```python
parses = list(cp.parse('bobby likes dana'.split()))
print(parses[0])
treesem = parses[0].label()['SEM']
print(treesem)
print(m.satisfy(treesem, g))
```

**Task 9.** Use this grammar to parse sentences telling you whether Chris likes Bobby and
whether Chris likes the Sun.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

Don't forget that the Sun is all one word (`the_sun`) in this grammar.

</div>

That's actually pretty cool.  We can get from a sentence to a tree to truth conditions to
an actual evaluation of whether a sentence is true or false.  Granted, we can't do very
complicated sentences, but we have a place to start and we can kind of see how we could
proceed.

## The new part, that we did not do in class, starts here ##

### Handling quantifiers ###

I'm going to take us one step further, but this is going to get a little bit complex.
What we're going to try to do is allow for quantifiers like "every bostonian".

What we want to get at the top of "every bostonian likes chris" is:
`every x.(bostonian(x) -> likes(x, chris))`

Let's assume that the VP is still going to be: `\y.likes(y, chris)`.  So, the question
then is: what semantics can we give to "every bostonian" that can combine with the VP
to give us what we want for S?  It's pretty clear that there's nothing we can pick for `y`
that we can put into `likes(y, chris)` to get that `every x...` semantics that we want
for S.

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

Above I switched the variable name in the semantic value of the VP.  Instead of saying
`\x.likes(x, chris)` I said `\y.likes(y, chris)`.  I made this change because I think
it will be less confusing later.  But those two functions are completely equivalent.
It doesn't matter what the variable is, it just has to match.  Those are both the same
as `\z.likes(z, chris)` or `\rhinoceros.likes(rhinoceros, chris)`.

</div>


The trick that semanticists pull at this point is to say that actually what is happening
here is *not* that the VP is a function that takes the NP subject as an argument.  Rather,
the *NP* is a function that takes the *VP* as an argument.  That is, the meaning of
"every bostonian" is going to be something that takes a function (like "likes-chris")
and returns the value we want for S.  More concretely, we assume that "every bostonian"
is the function:

`\P.(every x.(bostonian(x) -> P(x)))`

If we apply this function to `\y.likes(y, chris)` then what that means is that we
set `P` (predicate) to be equal to `\y.likes(y, chris)` and so we can substitute
`\y.likes(y, chris)` in for `P` in the part of the definition after the `.`.  That
gives us:

`every x.(bostonian(x) -> \y.likes(y, chris)(x))`

which simplifies to what we want:

`every x.(bostonian(x) -> likes(x, chris))`

<div style="margin-left: 0.5in; padding: 10px; background-color: #efe;" markdown="1">

The "simplification" step here comes up repeatedly.  Remember that this "lambda-notation"
for a function is `\x.(something...x...something)` and what that means is
"given a value, replace all instances of x with that value".  The notation for a
function with an argument is `function(argument)`, and when the function is in lambda-notation,
it looks like `\x.(something...x...something)(argument)`.  Replacing `x` with `argument`, we
get `(something...argument...something)`.  That's what happening in these "simplification" steps.

</div>

I told you it was going to be complicated.  But I think it's still comprehensible, though
it might take a couple of readings-through.

Before we put this into the grammar, let's also deal with the fact that we can also
talk about "every person" as well as "every bostonian".  We want to split up "every" and
the noun, and assign a meaning to each.  The meanings for bostonian, etc. can just be these:

```python
ndef = r"""
N[SEM=<\x.bostonian(x)>] -> 'bostonian'
N[SEM=<\x.cantabrigian(x)>] -> 'cantabrigian'
N[SEM=<\x.spaceball(x)>] -> 'spaceball'
N[SEM=<\x.person(x)>] -> 'person'
"""
```

And then we want the semantics of "every" to take one of those Ns and give us back the
"every" value we outlined above.  Here's what we can set "every" to in order to get that:

```python
ddef = r"""
D[SEM=<\N.(\P.(all x.(N(x) -> P(x))))>] -> 'every'
"""
```

So with that D and the Ns before, we make NPs out of them, and then we need to define S
to apply the subject NP to the VP (the reverse of what we had done before).  So:

```python
cfgdef = r"""
% start S
S[SEM=<?subj(?vp)>] -> NP[SEM=?subj] VP[SEM=?vp]
NP[SEM=<?d(?n)>] -> D[SEM=?d] N[SEM=?n]
VP[SEM=<?v(?obj)>] -> V[SEM=?v] NP[SEM=?obj]
"""
```

Now that we've changed the definition of S so that the NP is the function and the VP
is the argument, we need to fix our proper names.  The proper names used to be just
referring to individuals, but if the subject needs to be a function that takes a predicate
as an argument, we need to make proper names (like "Andrea") be functions as well.  What
semanticists do here is interpret "Andrea" as being not the individual `a`, but rather
a function that is true of any predicate that holds of `a`.  That is:

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

The last problem we need to tackle is that we need to derive the value of the VP
correctly, but now objects are not individuals but functions taking predicates.
We still want the semantics of the VP "likes chris" to be `\x.likes(x, chris)`
but now we need to build that from a combination of whatever semantics we assign to
"likes" and the semantics we just defined above for "Chris".

What we're going to do here is change "likes" so that it still takes "Chris" as an
argument, but just expects it to be this higher type.  It's confusing, I know.  But
I'll walk through it anyway.

The verb "likes" is going to take an argument NP, that argument NP might be "Chris"
and the semantic value of "Chris" is `\P.P(chris)`.  We're going to take that and
call it `X`.  This is the function that is true of any property Chris has.  What we
want to return is `\x.likes(x, chris)`.  Here is how we will define "likes":

```python
likesdef = r"""
V[SEM=<\X y.X(\x.likes(y,x))>] -> 'likes'
"""
```

So, if we are combining "likes" and "chris", then we have:

`\X y.X(\x.likes(y,x)) ( \P.P(chris) )`

Simplifying by replacing `X` with `\P.P(chris)` we get:

`\y.\P.P(chris)(\x.likes(y,x))`

Simplifying by replacing `P` with `\x.likes(y,x)` we get:

`\y.\x.likes(y,x)(chris)`

Simplifying by replacing `x` with `chris` we get:

`\y.likes(y,chris)`

And that is what we wanted.  It's hard to keep track of, I think you probably
would need to work out a bunch of these before you could feel confident that this
is a generally applicable definition for a transitive verb, but let's just
assume it is.  So, we are almost ready to assemble our new grammar.  One other
addition we can make is the quantifier "a", which works in much the same way as
"every" did:

```python
adef = r"""
D[SEM=<\N.(\P.(exists x.(N(x) & P(x))))>] -> 'a'
"""
```

Ok, let's finally build this grammar.  We can pull all the pieces together
like this:

```python
cfgdef += ndef + ddef + adef + npdef + likesdef
print(cfgdef) # just to make sure it looks right
gram2 = grammar.FeatureGrammar.fromstring(cfgdef)
cp2 = nltk.FeatureChartParser(gram2)
```

And now the moment of truth.  Let's try parsing "every person likes dana".

```python
parses = list(cp2.parse('every person likes dana'.split()))
print(parses[0])
treesem = parses[0].label()['SEM']
print(treesem)
print(m.satisfy(treesem, g))
```

If it said `True`, you have my permission to stand up and do a little jig.

If it didn't, it should have, so you probably need to go back and check for typos.

**Task 10.** Check whether Andrea likes every person, whether a spaceball likes a person,
whether every Bostonian likes the Sun.

One could imagine continuing on, but at this point, that's on your own time.


