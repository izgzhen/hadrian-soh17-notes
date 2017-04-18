Proposal (Draft)
====

## Introduction

I am really excited about help pushing Shake build into the official GHC tree. I have been knowing Shake for quite a bit of time (used it experimentally in one of my old [Haskell static blog generator project](https://github.com/izgzhen/bbq-sg)).

Regarding GHC, I was also a contributor to [HaLVM project](https://github.com/GaloisInc/HaLVM), where I had to mangle with the GHC build script as well (though not as deep as Hadrian needs to). So my conclusion is: I am kinda good at tinkering this kind of complex system.

Besides the most relevant parts, I am a regular open source contributor, a system hacker, and a functional language enthusiast. I participated in last year's GSoC, working on [Servo](https://servo.org) browser engine using Rust language under the mentoring of Mozilla Org. I am also involved in many other stuff, which can be accessed through https://github.com/izgzhen.

Finally, I will stay in China from the entire working period (I will graduate at June!!), and after that, I will come to U.S. at around Sept. 10th for starting my PhD on [PL/SE at UW](http://uwplse.org).

## Preliminary Research

- I've read the paper on Haskell 16'
- Going through the sources while flipping over the existing issues
- Participated in discussion about several issues: ????
- Submitted / Solved some issues: ????

## Project Overview

### Significance -- The Last Mile


### Workflow
We sure need two pieces of build, one standard, one hadrian, so we know the difference and can compare side by side

Hadrian team is smaller and the project is not as big and mature as my GSoC project, but I think it is a chance to promote better community working style such as through email/IRC/slack etc.

I will keep a weekly report on my completed tasks and planned ones.

## Project Structure

- Cross compilation
- Source and binary distribution generation
- Dynamic way
- Test suite support

### Cross Compilation



### Dynamic way

https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/shared_libs.html#using-shared-libs

https://github.com/snowleopard/hadrian/issues/4

https://ghc.haskell.org/trac/ghc/wiki/Building/Architecture/Idiom/VanillaWay

`mk/ways.mk`

### Source and Binary Distribution Generation

### Test Suite Support

## Timeline

Though "June 13th" is the official begin-to-work day, I will start working incrementally & in a slow pace and refining the targets from the end of April. It doesn't really matter even I will not be accepted, so just take it as myself evaluating the project.

Before Midterm evaluation (July 18rd), I hope to be familiar with the majority of the code base, of the GHC old build system script meanings, and solve one to two major targets (from dynamic way and test suite).

Before the End of Work (September 2nd), I plan to finish the left major goals (all rest, since @snowleopard doesn't know much about it).


## Some random notes

### GHC build system

https://ghc.haskell.org/trac/ghc/wiki/Building

### Meta

You get to understand what each important issue you'd like address means. Even this can construct a several page run-down. As far as solutions, what is current here (none or partial), and what needs to be done (in principle, and in referencing the old build system).

Also, you have to be able to read the old Make scripts...
