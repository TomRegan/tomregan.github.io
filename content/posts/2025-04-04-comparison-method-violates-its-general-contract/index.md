+++
date = '2025-04-04'
tags = ['java', 'comparators', 'collections']
title = 'Comparison method violates its general contract!'

[cover]
image = 'posts/2025-04-04-comparison-method-violates-its-general-contract/cover.jpg'
+++

**[ComparatorVerifier](https://github.com/TomRegan/comparatorverifier) is a library for verifying Comparators. Check it out!**

Java's standard sorting algorithm, TimSort, is a clever hybrid algorithm
that uses a deceptively simple insight into the typical pattern of elements in
arrays. Unfortunately the insight is couched deep in mathematical
obscurity:

> Entropy considerations provide a natural estimate of the number of
> comparisons to sort incompletely shuffled data. From an information
> theoretic viewpoint, the number of comparisons needed to sort a
> permutation is an upper bound on the Kolmogorov complexity of the
> permutation. A sorting algorithm generates a string of comparisons
> unique to each permutation. These strings form a prefix-free encoding
> for permutations. Since there are N 1 permutations, most permutations
> will require at least 1gN! comparisons. The challenge is to determine
> classes of permutations which require fewer, and to construct sorting
> algorithms whose comparison counts approach the information theoretic
> limit within these classes.

*Excerpt from "[Optimistic Sorting and Information Theoretic
Complexity](https://dl.acm.org/doi/pdf/10.5555/313559.313859)" by Peter
McIlroy.*

Wow. Let's spend some time breaking this down, and we'll see if we can
come to a better understanding of the tricks TimSort uses, and why it
has to throw an exception when things go wrong:

``` text
java.lang.IllegalArgumentException: Comparison method violates its general contract!
    at java.util.ComparableTimSort.mergeHi(ComparableTimSort.java:835)
    at java.util.ComparableTimSort.mergeAt(ComparableTimSort.java:453)
    at java.util.ComparableTimSort.mergeForceCollapse(ComparableTimSort.java:392)
    at java.util.ComparableTimSort.sort(ComparableTimSort.java:191)
    at java.util.ComparableTimSort.sort(ComparableTimSort.java:146)
    at java.util.Arrays.sort(Arrays.java:472)
    at java.util.Collections.sort(Collections.java:155)
    ...
```

## First Steps With TimSort

When I have to reason about sorting, I usually think about a bookshelf,
so let's use that idea to get an insight into what TimSort is doing.

- **What does TimSort do?** It examines a chaotic collection of books
  and identifies sequences where the books are already arranged
  alphabetically.
- **How?** It scans through the shelf left to right and "filters out"
  already-ordered groups (called runs). Then it merges these groups,
  ensuring the books remain in order.
- **Why?** It's much easier to merge small, sorted groups than to
  re-sort the entire collection from scratch. This approach saves time
  and effort,
- **What if something goes wrong?** Imagine if some books had
  contradictory labels: to the left of the shelf TimSort decides that
  Book A should come before Book B, while to the right it thinks that
  Book A comes after Book B. When you try to merge these groups, the
  inconsistency becomes impossible to resolve. In TimSort, when it
  detects such conflicting orders during merging, it stops the process
  and throws the error: "Comparison method violates its general
  contract!"

This ordering mix-up is a very strange situation for TimSort get itself
into! Key to this is the fact that TimSort uses outside help to work out
what the ordering of elements in an array should be. It uses a
Comparator. In our example of the bookshelf, the comparator would be
responsible for knowing the alphabet, and when it's handed two books,
giving the correct answer about which book goes on the shelf first, and
which is second.

Before we examine TimSort, let's throw in one last idea to help
understand how the comparator might be thinking about the alphabet. You
and I probably think of the alphabet running left to right, starting at
A and *ending* at Z. But we should imagine a comparator doesn't see the
alphabet that way! Imagine the alphabet mixed up and arranged around a
clock face.

![An image of the alphabet out of order and arranged around a clock
face](alphabetical-clock-face.png)

Our new jumbled up alphabet runs from N to M, and it goes N, O, P, Q, T,
U, X, L, H, C, Z, G, D, C, B, A, M, *and back to N*. Let's think about
that letter N in the 12 o'clock position. Starting from 12 o'clock, N
comes after M and before O; O comes after N and after M; but what about
M? As the minute hand goes around the clock from 12 it reaches N, and
later on M: N comes before M. But after passing M it reaches N. So N
comes after M. Clearly N can't come both after *and* before M: there's
something wrong with our model of the alphabet, and this is similar to
the way a comparator can fail to impose *total ordering* over its
elements. In fact the property it's breaking here is called
transitivity.

### Transitivity

In a comparator, transitivity is the property that states, for three
elements, 1, 2, and 3, if 1 is smaller than 2, and 2 is smaller than 3,
then 1 must be smaller than 3.

Formally:

    If x ≤ y and y ≤ z, then x ≤ z

Returning to the clock face, we can see how a cycle might be introduced
by an incorrectly written comparator, and how it's possible to get into a
loop, like 1 → 2 → 3 → 1.

## TimSort Step by Step

Now that we've got to grips with this metaphorically, we'll take a look
at an example which is a little closer to the facts. We'll skip over a
lot of technical detail unless it's helpful to understanding why the
exception is thrown. To begin, let's trace through the sorting of a
short list of numbers.

### Step 0: Initial Array

Let's take a simple array that's partially ordered, which is ideal for
TimSort:

    [2, 3, 5, 7, 6, 4, 1, 8, 9]

You can visually spot a few runs in there:

    [2, 3, 5, 7, 6, 4, 1, 8, 9]
    [2, 3, 5, 7]                   // ascending
                [6, 4, 1]          // descending 
                         [8, 9]    // ascending

### Step 1: Detect Runs

TimSort will scan the array and identify these sequences. If a run is
descending, it reverses it to make it ascending.

    [6, 4, 1] → [1, 4, 6]

So after detecting and adjusting runs, we get:

    [2, 3, 5, 7]    // already ascending
    [1, 4, 6]       // reversed from [6, 4, 1]
    [8, 9]          // already ascending

At this point, TimSort has a stack of runs like:

    runs ← [
      [2, 3, 5, 7],
      [1, 4, 6],
      [8, 9]
    ]

TimSort will continue to add runs to the run stack until a merge is
triggered.

### Step 2: Merge The Runs

Now we start merging the runs. First we merge `[2, 3, 5, 7]` with
`[1, 4, 6]`. To merge them, we compare each run element by element,
putting them into a temporary array:

``` text
compare(2, 1) → 1 wins → [1]
compare(2, 4) → 2 wins → [1, 2]
compare(3, 4) → 3 wins → [1, 2, 3]
compare(5, 4) → 4 wins → [1, 2, 3, 4]
compare(5, 6) → 5 wins → [1, 2, 3, 4, 5]
compare(7, 6) → 6 wins → [1, 2, 3, 4, 5, 6]
                7 left → [1, 2, 3, 4, 5, 6, 7]
```

The newly created run is pushed back onto the stack, and the remaining
run stack looks like this:

``` yaml
runs ← [
  [1, 2, 3, 4, 5, 6, 7],
  [8, 9]
]
```

### Step 3: Final Merge

Merge `[1, 2, 3, 4, 5, 6, 7]` with `[8, 9]`.

The first list is entirely smaller than the second, so the result is
just the full sorted array:

    [1, 2, 3, 4, 5, 6, 7, 8, 9]

## Triggering a Merge

At the end of step one, we said that runs would be stacked up until a
merge is triggered. So what triggers a merge? This is key, because the
same condition which triggers a merge, is also responsible for detecting
that the sort is failing because the comparator is incorrect.

The condition is made up of two rules. If either of these rules is
broken, a merge is triggered:

1.  For three consecutive runs on the stack the length of the top-most
    run is greater than the combined length of the next two runs
2.  The run at the very top of the stack is longer than the next run
    beneath it

When either 1 or 2 is untrue, TimSort stops searching and begins merging
the run stack. If these rules are still broken at the end of
merging, TimSort sees that the comparator didn't do its job, and
that's when the exception is thrown.

## Galloping (aka binary search): The Final Puzzle Piece

We have to understand galloping because it's during galloping mode that
TimSort detects intransitivity in the Comparator.

Galloping allows us to "jump ahead" in a run. Let's look at this in the
top two runs from our stack:

``` yaml
runA ← [2, 3, 5, 7]
runB ← [1, 4, 6]
```

First we do `compare(2, 1)` and put 1 in our temporary array. We
continue:

``` text
compare(2, 1) second run wins → [1]
compare(2, 4) first run wins  → [1, 2]
compare(3, 4) first run wins  → [1, 2, 3]
```

Because our first run is contributing more elements to the temporary
array, we're going to designate it the "winning run," and change our
strategy. Instead of looking at every element, we're going to perform a
binary search by jumping ahead a few elements into a region of
uncertainty.

We're guessing the winning run is already so well-ordered that it'll be several
comparisons before we find the next element which is smaller than the
element in the second run.

## Comparison method violates its general contract

We've examined how TimSort is supposed to work. Now we'll look at how it
fails. Let's continue with our example of galloping to see how TimSort
detects inconsistencies in Comparators. We'll go back to our first
successful merge, except this time we're going to imagine we've been
using an incorrect comparator all along. This comparator has strange
ideas about the number 4, and returns different results depending on the
number 4 is being compared with. Luckily it's worked correctly until
now, but that's about to change.

After the first merge, our run stack looked like this:

``` yaml
runs ← [
  [1, 2, 3, 4, 5, 6, 7],
  [8, 9]
]
```

Let's imagine we're galloping through the first run:

``` yaml
                 ↓
runA ← [1, 2, 3, 4, 5, 6, 7]
        ↓
runB ← [8, 9]
```

Our first comparison is between 4 and 8. Our comparator incorrectly says
that 8 is *less than* 4 because it has been written improperly. Our next
comparison is with 3. Now the comparator returns the correct result, that
8 is greater than 3. We've already determined, using the same
comparator, that 3 is less than 4, because we've previously sorted this
run, so our comparator is telling us both that 4 is less than 8 and
greater than 3, and that 3 is less than 8 and 4 is greater than 3. We
can compare this with our formal definition of transitivity from earlier
on:

    If x ≤ y and y ≤ z, then x ≤ z
    If 3 ≤ 4 and 4 ≤ 8, then 3 ≤ 8

But our comparator has decided:

    3 < 4 and 3 < 8, but 4 > 8

This could leave TimSort to search endlessly up and down the run, like
the minute hand circling a clock, if it didn't stop to throw the
exception.

## Writing Better Comparators

Avoid writing your own comparators unnecessarily. Familiarize yourself with the `Comparator`
API in Java, prefer those comparators when they're available.

If you're writing your own comparator, make sure you understand the
comparator contract, which is given in the [Comparator
Javadoc](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html).
A few smells I've noticed over the years are stateful comparators,
modulo arithmetic, and the use of random values, but human creativity is
limitless and a full survey of potential faults is impossible.

If you want to use a formal verification framework to test your
comparators, you can try out a library I created,
[ComparatorVerifier](https://github.com/TomRegan/comparatorverifier).
It's easy to use, and it can detect violations of the comparator
contract at test time, rather than mid-way through sorting!

## Further Reading

* [Comparison Method Violates Its General Contract! (Part 1) by Stuart Marks](https://www.youtube.com/watch?v=Enwbh6wpnYs)
* [Comparison Method Violates Its General Contract! (Part 2) by Stuart Marks](https://www.youtube.com/watch?v=bvnmbRo7a1Y)

## AI Used in this Post

* I used ChatGPT to proofread and fact check this post. The
chat is at: https://chatgpt.com/share/67f158a2-56ac-8000-b86f-3ca6d50e8c28.
* I used ChatGPT to generate the image of a clock with letters around it. 