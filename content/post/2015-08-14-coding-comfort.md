---
title: Coding Comfort
layout: post
author: Jaime Silvela
tags: ["coding", "typography"]
date: 2015-08-14 10:23:32.000000000 +02:00
---

In the last year or so I've made a number of changes to my coding habits that
have had a positive impact on my comfort.

### Touch typing
Some years ago, I read Steve Yegge's popular post about touch-typing. His thesis
was that good programmers must be fast typists. As with much in his blog, I
thought it was self-indulgent crap. If your coding speed is limited by
your typing speed, something is very wrong with the way you work.

Last year I learned to touch-type. I'm not a fast typist by any standard, but
touch typing has an enormous benefit for me: I don't constantly switch my gaze
between screen and keyboard. This reduction of repetitive movement makes typing
a lot more enjoyable.

### Mouse is better than keyboard. Mouse chording is a super power.
I had been an Emacs die-hard ever since learning Scheme in 1998. All that power
hidden within.
The Emacs way is to use the keyboard for everything. There are keyboard
shortcuts to jump back and forth, or to delete, by character, line, word,
paragraph. Mousing works, but it's a bit clunky. For copying & pasting you'll
more likely navigate and select with the keyboard, and use the keys for copying
and pasting (C-w, M-w and C-y, or, if you enable CUA mode (which you should)
C-x, C-c and C-v).
The keyboard culture is not unique to Emacs fans, of course.

Then I learned to use the Acme editor, which was designed from the beginning to
take advantage of graphic displays and to use the mouse heavily.
Here's some background on the [merits of the mouse.](http://plan9.bell-labs.com/wiki/plan9/Mouse_vs._Keyboard/index.html)
Acme uses mouse chording, a great idea that I haven't seen used elsewhere. After
using mouse chording for a while, having to type all those commands seems so silly.
[Russ Cox has a great intro video to Acme and chording.](http://research.swtch.com/acme)

#### But menus suck
All those modern programs with their multi-level menus and flashy toolbars. They
had better make sure that the most important commands are one click away.

### 80 column rule
I used to think this was a silly rule. With our modern widescreen monitors, we
should be taking advantage of the extra screen real estate, no? Surely, Java
programmers need all the width they can get (the ergonomics of Java deserve a
dedicated  article).
Recently I decided to adopt the 80 column rule, and I can discern at least 3
benefits:

1. Code becomes simpler. That 4-level-deep control structure was asking to be
taken apart, and those long identifiers were too verbose.
2. Side by side viewing/diffing becomes a lot easier.
3. Your code can fit printed onto a page, using a monospaced font at normal
sizes. Which segues into ...

### Font size
Many programmers brag about using one of those bitmapped programmer's fonts at 8
or 9 pixels. OK, so they can fit more code on screen. They're also ruining their
eyesight. To each his own.
I propose that you perform a little experiment: take some code that obeys the 80
column rule, with some long lines (78-80 cols.), and paste it into a word
processor. Use your favorite monospaced font, and set the font size as high as
possible, making sure the long lines don't wrap.
Now print the page, and compare the code on screen and on page, keeping each at
the normal distance for you.
Try to match your screen font size to the printed font size. You will likely
need to increase it signifcantly.

I've switched to the 80 column rule, and to large font size, recently. In the
beginning it seemed odd. Those fonts looked so big! Then I got used to it, and
after a while I started to notice the reduced strain on my eyes.
If you want to fit more lines of code into the screen, get a bigger screen;
don't compromise on font size.
