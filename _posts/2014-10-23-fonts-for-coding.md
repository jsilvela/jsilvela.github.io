---
title: Fonts for coding
layout: post
---
# Fonts, readability, coding
I’ve been on a typography bender lately, so this is a quick summary of my findings.

## Size matters
Some coders have a bit of a macho attitude  to see how small you can make your coding font, on the theory that the more lines of code you can display at once, the better. One often sees posts from coders claiming that Monaco, DejaVu, Consolas etc. at 9–10pt is their font of choice.
Now that I’m approaching middle age, my eyes get tired more easily. I did a test: I printed some code onto a sheet of paper I could read comfortably. Then I compared it side by side against my monitor at regular distance. The results were shocking. For my setup (Mac Mini with a 21" screen), I need about 20pt fonts to match regular printed letter size. No wonder my eyes were getting tired!

## Fonts for coding. Criteria
0. Not necessarily monospaced!
1. Render well on your editor/OS.
2. Distinct capital-i, lowercase-l and number-one. Good size of punctuation. Good size of brackets, braces, parens.
This disqualifies many sans-serif fonts, which often have similar characters. Some exceptions: Verdana, Clear Type, ITC Officina.
3. Numbers are clearly distinct from letters. Notably, zero and one. Fonts with “old style characters” fail here. For example: Georgia.

## Some recommendations
* Verdana is great. The characters are distinct, there is good spacing, punctuation and brackets/braces/parens have room.
But I find that the long, non-spaced sequences often found in code become forbiddingly dense when set in Verdana. Eg:
bytes, errfile := ioutil.ReadFile(filename)
* Input Sans. A proportional font designed for coders! It gets everything right. Except … for some reason, I find it ugly. I keep noticing it when I use it, instead of it getting out of the way. I don’t know why. But do try it!
I hope Input Sans is a sign of things to come.
* FF OCR-F. This is a wonderful font. It’s proportional, and it keeps generous spacing between letters. In fact, more space than Verdana, and it avoids the cramped feeling of long non-spaced code. The only issue I have with it is that the font weights are extreme, either too light or too strong.
* Two monospaced fonts worth downloading: Fira Mono, Source Code Pro.
