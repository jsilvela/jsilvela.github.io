---
title: White space in source code, or ditch monospace
layout: post
author: Jaime Silvela
tags: ["coding", "typography"]
date: 2014-10-29 10:23:32.000000000 +02:00
---

White space is very important for readability in general, and for readability of code in particular.
If you look at a book on typography, you’ll see discussions on line spacing, kerning, double spacing and other spatial concerns. For example, check [Butterick’s Practical Typography](http://practicaltypography.com/).

Code is less dense on page than regular prose, but it is very compressed in comparison. Each and every letter and punctuation mark is important, each character must be clearly identifiable.
Many coders use monospaced typefaces to display their programs, and there are several monospaced typefaces designed specifically for code.

Monospaced typefaces have peculiar problems. Because all their letterforms are made to fit the same width, the i’s and l’s are swimming in space, and the m’s and w’s are compressed. This can give rise to distracting changes in visual rhythm, or to letter collisions. In contrast, proportional fonts usually manage to keep a uniform density across the line:

![Issues with monospaced fonts](/images/mono-problems.png)

Aside from their inherent problems, how do monospaced typefaces compare to proportional ones? Let’s look at the Bitstream Vera family, which contains proportional and monospaced fonts.

![Bitstream Vera family](/images/Bitstream_Vera_family.png)

The line set in Vera Mono, at the top, is very loose; looser than necessary, I find. My favorite rendering is the third one, with Vera Serif. In my opinion, it makes it much easier to tell what’s going on, and the letters are perfectly discernible.

Why are so many programmers unwilling to use proportional typefaces? I think one important reason is that ever since C++, Java and CamelCase became the defaults, we’re used to long, un-spaced identifiers, and long lines of code. In those circumstances, the accidental whitespace patterns created by monospaced typefaces are a relief for the eye. Proportional typefaces can create bleak walls of text. <br/>Let’s compare Vera Mono vs. Vera Serif:

![Long lines of code (Vera fonts)](/images/no_underscores.png)

What would happen if we ditched CamelCase, and separated the sub-words with underscores? Let’s compare Vera Mono, with no underscores, vs. Vera Serif with underscores:

![Long lines, with underscores (Vera fonts)](/images/underscores.png)

Much better, right? The spacing is deliberate, and the eye is drawn to the gaps within the identifiers. Even with the extra characters, the code set in Vera Serif is more compact than in Vera Mono.

Many proportional typefaces are unsuitable for coding, but some are as good at distinguishing letters as any monospaced. For instance, Verdana, Charter, Bitstream Vera Serif (and its descendant DejaVu Serif), Clear Sans, ITC Officina Sans, the Input family…

Should proportional typefaces be used for code? We’re all free to choose; after all, we each get our own monitor.
Here’s my take: a good proportional font, and judicious coding conventions, will improve the readability of your code base.
Watch your spaces!