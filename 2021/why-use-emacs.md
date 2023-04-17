# Why I use emacs (and can I stop?)

Author: David Beazley ([@dabeaz](https://www.dabeaz.com))
August 29, 2021

In a recent class, someone remarked that I must have some kind of crazy emacs setup--especially given my certain reputation for live coding, and, well, using emacs.  However, he was somewhat shocked when I said I never customize emacs.  This got me thinking about my relationship to emacs--an editor that I've now been using for 30 years.

## Why I use emacs?

In short--I blame my summer student internship from 1991. On the first day, they showed me my desk, a Sun 3 workstation, a copy of the K&R C Programming book and said "go figure it out."  I had never even used Unix before.  It did not go well. "ls? man? What the fuck?!?!?"  Ahem. Pardon me.   Oh yeah, "also go learn this editor."   Needless to say, the first couple of weeks were rough going.

Did I mention that I was also supposed to write code to solve stochastic differential equations? Did I know anything about stochastic differential equations?  No, I did not.   So, there was also a small stack of dense math papers involved.  Learning the text editor was the least of my various head-exploding problems at that point.

Anyways, my use of emacs stuck from that point forward.  Almost all of my later work took place in the terminal and Unix environments.  I didn't even own a computer from 1988-1995.  For a short time, I owned a VT220 terminal that I'd use for dial-up into the university. Emacs was almost always available and it worked decently well on that setup. 

As for my continued use of emacs in the modern era, I just use it straight-away in the text terminal (typically iTerm2)--no GUI or anything.   There's no real reason for doing that other than the fact that I'm used to it.  

## How much emacs do I actually know?

Not much as it turns out.  Here is the combined sum knowledge of stuff I generally know about emacs:

* Quitting.
* Basic cursor movement and scrolling around.
* Loading and saving of files.
* Search-and-replace.
* Split-screen (both vertical and horizontal!).
* M-x font-lock-mode (to disable colors in an emergency if giving a live demo on a bad projector)
* I once defined a keyboard macro (but had to look up how)

Also, I think I once had to customize `.emacs` back in 1996 when I was first learning Python and I wanted to install Python-mode.

Honestly, that's about it when it comes to me and emacs.  Honestly, it's not something that I spend a lot of time thinking about. For what it's worth, the first three bullet points also describe the sum total knowledge I have about using vim. I mean, sometimes you're in a real pinch and you might have to edit the Makefile for emacs.  I digress.  

## Can I actually switch to something else?

There's always a danger in getting too set in one's ways.  So, seeing a three-week window before my next course, perhaps now would be a good time to just abandon emacs cold turkey. Learn something else--maybe even one of those new-fangled IDEs that all of the cool kids are using. 

Anyways, I wrote this blog post in VSCode. I'll report back later with further progress. 

## Discussion

Want to make a comment?  Hit '.' to edit this page. Then submit a pull request. 

### Emacs -> Vim -> VSCode
Roni here! I started out using Emacs in January 2000 (the same year I started using Linux). Then, when I started at Kitware in 2012, I thought I'd like a change and switched over to Vim. Now it's 2021 and all the cool kids are using VSCode, so I finally took the plunge and discovered that its integrated LSP support is pretty sweet. I'm feeling that slow slide away from Vim now, even though it wrecks some of my "old man" appeal.

### VSCode --> Neovim (AstroNvim config)
When I first started learning how to code, I used VSCode because it's mechanics mirror a word processor. I didn't want to learn how to use an editor on top of learning Python, design patterns, Git, and other important skills and concepts. Within the first few hours of my first professional software engineering role, I decided to use Neovim. I saw all of the senior engineers using it and marvelled at how quickly they could edit and insert text. At first, I was going to commit to learning VSCode features and shortcuts. I realized that these shortcuts would still require me to use my mouse more often than I would like. Also, there is no underlying grammar behind the shortcuts. I've enjoyed learning Vim motions because there is an underlying grammar behind them. This means that I can infer shortcuts based on previous motions and actions I've learned. Each time I find myself doing something slowly or with a mouse, I Google it and learn how to speed up my work. There have been some annoying moments, and I'm not a Vim expert yet, but using Vim has helped me drop my mouse usage and spend more time in the terminal. As for the features that make VSCode great, I'm using an [AstroNvim](https://astronvim.com/) config that gives me access to an LSP, terminal windows within Vim, a great fuzzyfinder, package management, etc. I feel like I have a [PDE](https://www.youtube.com/watch?v=QMVIJhC9Veg) instead of an IDE.





