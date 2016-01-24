---
layout: post
title:  "Problem solving, creativity and scientific decision making"
date:   2016-01-24 00:00:00
excerpt: "Creativity and Scientific Decision making"
tags: [creativity, innovation, invention, problem solving, decision making]
categories: [others, creativity]
comments: true
---

Problem solving can be seen as one of the challenges or our daily life. Some of
the dilemmas we are confronted with require careful thinking and planning. Solving
them can turn out to be a big task and it seems like a structured, well-defined
path to take isn't always available for achieving a satisfactory result.

The purpose of this article is to demonstrate that any kind of problem, of major
or minor difficulty, irrespective of its nature, can be resolved in a creative,
structured way.

The instrument I will use in order to exemplify this claim is a scientific
method called MCAA.

### The MCAA method

The MCAA (multi-criteria advanced analysis) method can be used with exceptional
results in many situations:

* **different types of rankings**

* **weighing of several alternatives (variants, subjects) and if
necessary, selecting the best**

* **establish an order of value for several variants (alternatives), in light
of some criteria**

* **compare one or more variants of own choice with existing variants of a
product, object, etc**

Consequently, the method can be applied in any domain where creative problem
solving is required. As an example, it can be determined who is the best cyclist
or actor, of all times, or what person to hire in a position, or which variant
of a design best satisfies us from a cost point of view, and so on.

**It is necessary to mention that this method tends to produce results with a high
degree of objectivity.**

MCAA is based on a mathematical model that I chose not to expose here.
Instead, the presentation will be centered on a software program that hides the
complexity and the mathematical nature of MCAA and provides a good high-level
overview in order to give not-so-mathematically-inclined people a chance to
follow.

While very general and complex, this
[wikipedia article](https://en.wikipedia.org/wiki/Multiple-criteria_decision_analysis)
gives an overview of the mathematics behind various decision-making
methodologies.

I had the chance to study decision-making (and MCAA implicitly) with the
occasion of my university studies, and I developed the software I will present
later in this article while working for my graduation project.

The MCAA method consists of several steps:

#### Step 1 - Establish criteria

**A criterion is a clear and well defined point of view of the specialist
(domain expert, analyst), or of any interested person by which he (alone or in
teams) delimits, defines certain properties, attributes, features of the
analyzed object.**

The most important criteria (which can be many of course  - therefore the
analysis is called 'multi-criteria') must be found so that the result will be
meaningful and unambiguous.

It is recommended that finding and choosing the criteria should be done in one
or two brainstorming sessions, followed by a
[morphological analysis](https://en.wikipedia.org/wiki/Morphological_analysis_(problem-solving))
(If there is interest, I will describe in future articles the morphological
analysis technique, together with some examples).

#### Step 2 - Determine the ordering of the criteria

* the ordering of the criteria is established by comparing every 2 between them

* when judging the relative position of two criteria there can exist only 3
situations: a criterion is more important than another, a criterion is as
important as another and a criterion is less important than another

These two factors are some of the strongest reasons that demonstrate the highly
objective nature of the MCAA method.

**Determining the order of the criteria will have as result a weight
coefficient for every one of them**

#### Step 3 - Identify variants

**By variants we understand subjects, products, objects, realistic solutions
that meet the same end uses or purpose.**

#### Step 4 - Grade every variant

A grade should be an integer (maximum 10). It is also called 'importance grade'
or 'contribution of a variant to a criterion'.

**Each variant is examined in light of each criterion (and graded) until all
options are exhausted.**

### Software

In order to have a complete image about MCAA mechanics, I will introduce a piece
of software designed specifically for introducing the method to the world.

> Please note that this program has been written around the year of 2008 and, at
that time, software usability principles were largely unknown to me.
With that being said, I still believe it is worthy enough to be presented here.

>The quality of this software does dot reflect my current skills as a software
craftsman.

### Tutorial

What follows is a demonstration of MCAA using the program as a reference guide.

It is recommended that you follow this tutorial with the program open.
If you find yourself confused then you can always scroll to the description of
each step in order to understand what is expected of you.

**Imagine that a company desires to hire someone for a position, and ‘Tim’, ‘John’
and ‘Jane’ are three candidates that are to be ranked with respect to their
overall suitability for the job.**

#### Step 1 - Establish criteria

The first thing needed is to identify the criteria that are of utmost importance
in the current situation. The fictional company's hiring team came up with the
following:

* Education - has at least a BA
* Experience - has worked at least 3 years
* References - recommended by one of the team members
* Social skills - communicates clearly, displays leadership traits
* Enthusiasm - has read our blog, knows our business model
* Salary - budget for this position: max 40.000$/year

Type (or copy/paste) them in the text area, in the 'Write your criteria' zone,
as exemplified in the following image:

<figure>
<img src="/images/mcaa-1.PNG">
</figure>

When you are done, press the 'Criteria ordering' button.

#### Step 2 - Determine the ordering of the criteria

At this step, the program offers you the opportunity to compare each criterion
with the others.

Move the sliding selector using the arrows `<` `>` and make your choice using
the buttons on the right side of the panel.

As an example, for the first option, you should read: "The selected criterion
(Education) is [more important/as important/less important] than the selected
criterion (Experience).

<figure>
<img src="/images/mcaa-2.PNG">
</figure>

After all comparisons are done, press the 'Apply' button, and you will see the
result in a graphical format, based on the decisions you made.

<figure>
<img src="/images/mcaa-3.PNG">
</figure>

At this stage, had you been mathematically and calculus oriented, you would have
had to work through the calculations. Luckily, the program hides them completely
from you.

Press the 'Save and proceed button', you will be forwarded to the next step.

A side result of this action is the fact that the analysis is written on a
file located in the same directory from where you launched the program (see the
end of the article for an example).

As you've probably gotten used to, a window pops up. The description on the
window is self-explanatory, but note that by choosing 'Criteria' you can add
(even remove) new criteria or modify the choices you previously made. Just try
it out.

<figure>
<img src="/images/mcaa-4.PNG">
</figure>

#### Step 3 - Identify variants

Just write or copy/paste your variants and the press 'Grade variants'.

<figure>
<img src="/images/mcaa-5.PNG">
</figure>

#### Step 4 - Grade every variant

The way you should interpret this: "In light of the selected criterion
(Education) the selected variant (Tim) is worth [1/2/.../10] points"

<figure>
<img src="/images/mcaa-6.PNG">
</figure>

After you finish all the grading, you will be presented the interpreted results
of your analysis, in a graphical format.

<figure>
<img src="/images/mcaa-7.PNG">
</figure>

Next, decide if you want to add/remove another variant, or re-do the grading.
The program allows that, just try it out.

<figure>
<img src="/images/mcaa-8.PNG">
</figure>

As said earlier, the program produces a file called "M.C.A.A-1.0.data". This is
a fragment of that file.

<figure>
<img src="/images/mcaa-9.PNG">
</figure>

### Downloads

In order to run this program, you need java installed.

**[Click here to download java](https://www.java.com/en/download/)**

####For Windows users

You can get the program from **[this link](https://goo.gl/0Mu0Qf)**

####For Mac and Linux users

You can get the program from **[this link](https://goo.gl/RR5jia)**

The download is a java archive, run it by double-clicking it or in a terminal:
`java -jar mcaa-1.0-all.jar`
