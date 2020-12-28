+++
title = "Serif fonts for coding"
date = 2020-12-26T20:49:18+02:00
draft = false
+++

TL;DR you should try a serif monospace font for coding. It may well render
sharper, and/or be more readable than the sans-serif monospace fonts
you're likely using.

There aren't many serif monospaced fonts available. Two free ones are
[*Go Mono*](https://blog.golang.org/go-fonts) and
[*Courier Code*](https://fontlibrary.org/en/font/courier-code).\
A brilliant commercial one is
[*Lexia Mono*](https://www.daltonmaag.com/library/lexia-mono). I highly
recommend it, and it's quite affordable.

---

For many years now, I've been messing with coding fonts, antialiasing settings
on different operating systems, different monitors. Text rendering bothered me
slightly, and I noticed especially while programming. \
I managed improvements, but satisfaction was only ever temporary.

Last month I had my first ever near-range eye exam, and had my first ever
reading glasses made. Presbyopia had been creeping on me.
With the new glasses, I started seeing books and screens better than I had
since I couldn't remember when.

On my tablet and my phone's high definition screens, everything looked crisp
and detailed. But on my 2560x1440 monitor, I could now see more clearly
the differences in text rendering between Linux, Windows and macOS
(I use all three regularly.)

I've used *Consolas* as my main coding for years now. Sometimes I'd switch
because on some system, Consolas would appear a bit fuzzy. Perhaps *Menlo* or
*IBM Plex Mono* or *Fira Mono* would appear sharper. \
I also switched to *Verdana* for some time, and I was pumped about
[proportional fonts for coding]({{< ref "2014-10-29-white-space-mono-space.md" >}}).
Mostly, though, I think I just liked how well Verdana rendered,
and how different the shape of text was.

With the recent *macOS Big Sur* release, my font smoothing tweaks were
no longer available. I noticed that the monospaced fonts were fuzzy again.
And while I cursed Apple for undoing my smoothing tweaks, I noticed
that on most websites, the text now looked better, sharper.

How was that possible? Perhaps it was font-dependent?

In my obsessive testing of fonts, I had noticed that some fonts seemed
to render so much sharper. *DejaVu Serif*, in particular, rendered like a dream.
However, I did not like the letter shapes for some reason, and it didn't really
work for coding. *DejaVu Sans* and *DejaVu Sans Mono*, and its rehash *Menlo*,
didn't quite match the sharpness of *DejaVu Serif*. \
Perhaps the serifs helped?

I went to my text editors, tried different fonts and sizes. There was a pattern:
fonts with serifs just seemed to be more "in focus" to me.
I only found two monospaced fonts with serifs: *Courier* (yes, Courier!)
and *Go Mono*. Both were looking sharp on my 1440p monitor, with macOS
font smoothing. \
Fishing for other monospaced serif fonts, I decided to get a license for
[*Lexia Mono*](https://www.daltonmaag.com/library/lexia-mono).

It's not just the serifs. Serif fonts usually have some variability in thickness,
and it all adds to a more textured feel. In contrast, sans-serif monospaced
fonts, which are relatively featureless to begin with, seem to
become totally devoid of texture when anti-aliased.

We're in a transitional period for computer screen quality.
One day we'll all have screens that match paper, and all fonts will render
crisply. Until then, each OS handles text rendering and font smoothing differently,
and some fonts cope better.
