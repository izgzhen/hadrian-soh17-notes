Proposal (Draft)
====

Author: Zhen Zhang

## Introduction

I am really excited about pushing Shake build into the official GHC tree. I have known Shake for quite a while, including using it experimentally in one of my old [Haskell static blog generator project](https://github.com/izgzhen/bbq-sg).

Regarding GHC, I was also a contributor to [HaLVM (Haskell Lightweight Virtual Machine) project](https://github.com/GaloisInc/HaLVM), where I had to wrestle with (not *kill*) the GHC `make` scripts. My feeling is that I am kinda good at tinkering this kind of complex beast.

Besides the most relevant bits above, I am a regular open source contributor, a system hacker, and a functional language enthusiast. I participated in last year's GSoC, working on [Servo](https://servo.org) browser engine using Rust language, under the Mozilla organization.

For more information about me and my interests, you can take a look at my [GitHub](https://github.com/izgzhen) and [homepage](https://zhenzhang.me).

## Preliminary Research

- I read the Hadrian paper (Haskell Symposium 2016)
- I went through the sources and the existing issues
- I created or participated in discussions on several issues
    + [Document on debugging Hadrian](https://github.com/snowleopard/hadrian/issues/308)
    + [Track GHC commit](https://github.com/snowleopard/hadrian/issues/306)
    + [runhaskell can't find ghc](https://github.com/snowleopard/hadrian/issues/304)
    + [Implement installation and binary/source distribution rules](https://github.com/snowleopard/hadrian/issues/219)
    + [Fix dynamic way](https://github.com/snowleopard/hadrian/issues/4)
    + [Fix validation failures](https://github.com/snowleopard/hadrian/issues/299)
    + [Missing dist-derivedconstants](https://github.com/snowleopard/hadrian/issues/310)
- I merged some PRs into current Hadrian (thanks to Andrey Mokhov for guiding me and reviewing these changes!):
    + [Disable some warnings](https://github.com/snowleopard/hadrian/pull/307)
    + [Add wrapper for Runhaskell, Fix #304](https://github.com/snowleopard/hadrian/pull/305)
    + [Add Werror to CC and HC](https://github.com/snowleopard/hadrian/pull/309)

## Project Structure

The major priorities are:

- [Fix dynamic way](https://github.com/snowleopard/hadrian/issues/4)
- [Install and binary distribution generation](https://github.com/snowleopard/hadrian/issues/219)
- [Cross compilation](https://github.com/snowleopard/hadrian/issues/177)

I expect to finish them this summer, plus other smaller or less important issues in the meanwhile. By the end of my work, I wish to push the ["tree-tremble"](https://github.com/snowleopard/hadrian/milestone/2) progress from the current level (say 80%) to 95% [1]. Besides contributing directly to the code base, I also wish to help lowering the bars of contribution for newcomers, thus enabling a smoother path from 95% to 100%.

Also, maintainability is important since it is the motivation of this project. We should not replace one hack with another, although there is a natural tendency to throw in some hacks when dealing with such a system.

What's more, we need to make sure our improvements to Hadrian will work as expected on all major platforms, i.e. Linux, macOS and Windows. When developing Hadrian, it is important to keep the high-level build rules abstract and platform-independent, by talking to `autotools` etc. This can greatly ease our future maintaining cost. Also, multi-platform CI [2] will be the watchdog of our changes, but we sure need more tests and validation support to know if it really works.

Finally, some general references:

- [GHC build system](https://ghc.haskell.org/trac/ghc/wiki/Building)
- [Shake home](http://shakebuild.com) [3]
- Shake paper: [Shake Before Building: Replacing Make with Haskell](http://ndmitchell.com/downloads/paper-shake_before_building-10_sep_2012.pdf)
- Hadrian paper: [Non-recursive Make Considered Harmful: Build Systems at Scale](https://www.staff.ncl.ac.uk/andrey.mokhov/Hadrian.pdf)

### Fix the Dynamic Way

#### Overview

The dynamic way is a flag, which when turned on, will instruct the build system to create shared libraries for packages like `base`. So when the dependents are built with dynamic flag on, they will be linked to these shared libraries, rather than static ones. In current Hadrian, we include "ways" as part of the `Context`. In short, this part is about writing build rules for `*.so` targets.

Jose Calderon already had some progress on this issue. I need to talk to him more about this. Recently, GHC team had been reworking the dynamic way on Windows. We need to track this as well.

#### References

Issue: https://github.com/snowleopard/hadrian/issues/4

Affected issues:

- https://github.com/snowleopard/hadrian/issues/248
- https://github.com/snowleopard/hadrian/issues/105

GHC doc:

- https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/shared_libs.html#using-shared-libs
- https://ghc.haskell.org/trac/ghc/wiki/Building/Architecture/Idiom/VanillaWay

Old build system:

- `mk/ways.mk`
- `rules/build-package-way.mk`.

### Source and Binary Distribution Generation

#### Overview

Implement `install` and `binary-dist` rules in makefiles, by writing rules for moving files around and packaging stuff together.

For `install`, it has two command line configurations worth mentioning:

1. `prefix`: the default location for installation, used like `./configure --prefix=...` or `make install --prefix=....`
2. `DESTDIR`: the optional location for installation, used like `make install DESTDIR=...`

#### References

Issue: https://github.com/snowleopard/hadrian/issues/219

Relevant bits in Hadrian: [PR #265: Implement 'sdist-ghc' rule](https://github.com/snowleopard/hadrian/pull/265)

GHC doc: https://ghc.haskell.org/trac/ghc/wiki/Building/Installing

Old build system:

- `rules/bindist.mk`: A rule for adding file to binary distribution list
- `mk/install.mk.in`: Sets up the installation directories
- `ghc.mk:872`: figure out what kinds of files need to be installed, from where, and to where
- `Makefile`: dispatches to `unix-binary-dist-prep` or `windows-...` (defined in `ghc.mk`)

### Cross Compilation

#### Overview

It is important to support cross-compilation in Hadrian, such that user can specify a different target
and build a Stage1 compiler that compiles code for that target.

First, we need to add some command line arguments or user setting entries to turn on cross compilation and configure things like target platform etc.

Second, we need to tweak here and there to make sure that cross compilation configuration will be considered, like changing `CC` flags and instruct the C compiler to output the right object file.

Finally, we need a way to test this -- using an emulator of different architecture would be good enough.

Moritz Angermann is the person to ask on this issue. We also agreed that extending current out-of-tree build to the customizable multi-target build trees would be a nice thing to have.

#### References

Issue: https://github.com/snowleopard/hadrian/issues/177

GHC doc:

- https://ghc.haskell.org/trac/ghc/wiki/CrossCompilation
- https://ghc.haskell.org/trac/ghc/wiki/Building/CrossCompiling

Old build system:

- [CrossCompiling vs Stage1Only](https://github.com/ghc/ghc/blob/master/mk/config.mk.in#L555-L572)
- [Stage1Only vs stage=1](https://github.com/ghc/ghc/blob/master/mk/config.mk.in#L574-L603)
- [No stage2 packages when CrossCompiling or Stage1Only](https://github.com/ghc/ghc/blob/3b6a4909ff579507a7f9527264e0cb8464fbe555/ghc.mk#L1448-L1490)

## Workflow

### Increase the "throughput" of working

One of the hardest part of working on something like Hadrian is to improve your efficiency of development. For even a tiny bit of change to the build system, it is highly possible that I have to rebuild another GHC, which is inefficient. Also, sometimes we can't be super responsive in online discussions because of timezone difference etc. However, there are still some ways to improve the "throughput" of development, thus gaining better overall efficiency:

- **Batch Processing**: I usually deployed multiple pieces of GHCs (one standard, two or three Hadrian based). The good things about this are:
    1. Reviewers and I can track two or more different issues concurrently
    2. I can compare Shake-based system with Make-based system side-by-side
- **Decomposition**: Effective decomposition of a big task into several more tractable issues can greatly improve the throughput, for the same reason that smaller chucks of rocks get through the bottleneck easier

### Communication

GitHub will be our major tool of communication, mostly around some specific issues. Current GitHub review system is good enough, but we can switch to Reviewable.io if it turns out to be too limited.

For more general stuff, we can discuss through emails. A mail-list might be necessary in future if we are overwhelmed by maintaining the forward list.

Also, I will try to keep an openly-accessible weekly report on my completed tasks and planned ones.

We should try to set up a real-time discussion (with IRC/slack) each week, preferably just after my weekly report is read. I think each real time discussion would take around half an hour.

## Timeline

**May ~ June**: Familiarize myself with the project's source code, the usage of Shake, and the `make` magic in GHC. Solve some small issues.

**Before Midterm evaluation (July 18th)**: Finish one to two major priorities (esp. the dynamic way issue, which when solved, can unblock a lot of other possibilities).

**Before the End of Work (September 2nd)**: Finish the left major priorities. Review on what else needs to be done to actually ship Hadrian in GHC tree.

## Remarks

- [1] What is 100%? this is not a well-defined criteria yet, but I can talk with Ben Gamari to make it concrete.
- [2] Current macOS CI on Travis always time-outs because the build runs a bit longer than Travis's limitation. We should solve that in future.
- [3] Neil Mitchell is a great source of advice on Shake-related issues. Note that some important features, like freeze-stage1, are blocked because of Shake's current status
