+++
tags = ["coding"]
categories = []
title = "The programmer as a Non-Fungible Artisan"
date = 2021-05-23T09:29:18+01:00
draft = false
+++

> Is it possible that software is not like anything else, that it is meant to be
> discarded: that the whole point is to always see it as a soap bubble?

Alan J. Perlis

Oh, the fads affecting the practice of programming: I've witnessed the rise
or/and fall of Object-Oriented Design, Patterns, TDD, AOP, "scripting
languages", JVM, Hadoop, XML/SOAP, REST, etc.

Probably the original programming fad is *Top-Down Design*. \
Paul Krugman would call it a zombie idea: it never quite dies. I had a
colleague at Amazon who was constantly evangelizing Top-Down design. Any issue
with a project had him flying into an accusation that you had not worshipped at
the altar of Top-Down design.

This is the problem with these mantras: each one may be an interesting
idea, perhaps a worthy tool for your arsenal. Top-Down thinking certainly has
its place. But the software industry prizes stars and winners and "best
practices." Once an idea is found to have value, there is pressure to apply it
everywhere, always.

On the issue of Top-Down design, I love this critique from Hal Abelson in
[the 1986 lectures on *Structure and Interpretation of Computer Programs*](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/video-lectures/3a-henderson-escher-example/)
(quoted text begins at the 1:08:20 mark):

> Right, you end up with a marvelous tree, where you've broken your task into
> sub-tasks and broken each of these into sub-tasks and broken those into
> sub-tasks, right. And each of these nodes is exactly and precisely defined to do
> the wonderful, beautiful task to make it fit into the whole edifice, right.
> That's this mythology. See, only a computer scientist could possibly believe
> that you build a complex system like that.
>
> […] so if you go and change your specifications a little bit,
>
> […] a design like this is not going to be robust,
> because if I go and change something that's in here, that might affect the
> entire way that I decomposed everything down, further down the tree."

The whole lecture series, and the book it's based on (aka SICP [^1],)
espouses a different way of building
software: viewing your task as building a set of linguistic layers of increasing
sophistication. You might call this *bottom-up* design.

David Parnas's 1972 paper on module decomposition [^2]
doesn't attack Top-Down design explicitly, but it proposes an *information-hiding*
criterium to modularize systems, which stands in contrast with (then) conventional
wisdom. Comparing the modularization he proposes with another one
that has clearly inferior properties, he notes about the latter:

> Experience on a small scale indicates that this is approximately the
> decomposition that would be proposed by most programmers for the task
> specified

SICP's linguistic layers and Parnas's information hiding point to a similar
philosophy of programming. Neither the book nor the paper are easy
to reduce to a mantra. They're too rich for that.

Thinking about top-down design, I am reminded of the way artists draw. If you
watch an artist drawing, you'll notice that [^3] at any
stage of the process, they're working at different levels of detail
simultaneously. They probably begin with a rough outline of a portrait, then
start adding some background color. At one point perhaps they're giving a
lot of detail to an ear, while the nose is still just a pencil stroke. They look
at the proportions again, adjust the outline. Perhaps the chin is looking too
bulky, they concentrate on it for a while; re-evaluate, readjust, repeat.

This is, I think, the natural way to build systems and to program. It is the way most
programmers code, if they're not trying to impress someone. It's a simple recipe:
outline, build, step back, re-evaluate, correct.

I'm skeptical of the specific prescriptions I've seen proposed. "You begin
by writing a failing test" … oh don't make me laugh.

I wonder about the desire for these mantras.

Perhaps there is an esthetic yearning for a single silver bullet, rather
than a collection of techniques that don't work universally.

I think an important factor is that the industry is wary of
"unruly coders", and prefers an army of disciplined foot soldiers. It may also
be easier to look for "front-end coder with TDD and Agile" than for
"talented builder of systems." And, to boot, searching and hiring based on
buzzword compliance leads to a sense that your hires are fungible. \
I wonder if the people paying the bills are comforted when told "our engineers
follow Agile practices and TDD."

I'm dismayed when I see engineers calling for reductive methodologies and use
them to label themselves and others.

And I find the dilemma of skilled artist vs. fungible disciplined soldier a false
one.  There is no reason a skilled system builder should be un-disciplined or
unable to play well with others, any more than there is assurance that a loyal
practitioner of TDD will write solid code or be the colleague your team needed.

This is all personal, of course. I've met engineers who were fans of Bob Martin,
for instance. \
For me, a great recent book on building software is John Ousterhout's
*A Philosophy of Software Design* [^4].
This is the kind of book you'd take to a desert island to hone your programming.
Like good art, it asks more questions than it answers.

[^1]: *Structure and Interpretation of Computer Programs, Second Edition*,
  Harold Abelson and Gerald Jay Sussman, MIT Press, 1996 \
[Free online version](https://mitpress.mit.edu/sites/default/files/sicp/index.html)

[^2]: *On the criteria to be used in decomposing systems into modules*,
  David L. Parnas, Communications of the ACM, Vol. 15 (12),1972 pp. 1053-1058

[^3]: Bob Ross notwithstanding

[^4]: *A Philosophy of Software Design*, John Oustherhout, Yaknyam Press, 2018
