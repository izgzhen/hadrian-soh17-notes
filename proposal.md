Proposal (Draft)
====

## Introduction

I am really excited about help pushing Shake build into the official GHC tree. I have known Shake for quite a bit of time (used it experimentally in one of my old [Haskell static blog generator project](https://github.com/izgzhen/bbq-sg)).

Regarding GHC, I was also a contributor to [HaLVM project](https://github.com/GaloisInc/HaLVM), where I had to wrestle with the GHC build script as well (though not trying to *kill* them). So my conclusion is: I am kinda good at tinkering this kind of complex nasty system.

Besides the most relevant parts, I am a regular open source contributor, a system hacker, and a functional language enthusiast. I participated in last year's GSoC, working on [Servo](https://servo.org) browser engine using Rust language, under the Mozilla organization.

Finally, I will stay in China from the entire working period (I will graduate at June!!), and after that, I will come to U.S. at around Sept. 10th for starting my PhD on [PL/SE at UW](http://uwplse.org).

For more information about me and my interest, you can take a look at my [GitHub](https://github.com/izgzhen) and my [homepage](https://zhenzhang.me).

## Preliminary Research

- I read the Hadrian paper (Haskell Symposium 2016)
- I went through the sources while flipping over the existing issues
- I created or participated in discussions on several issues
    + [Document on debugging Hadrian](https://github.com/snowleopard/hadrian/issues/308)
    + [Track GHC commit](https://github.com/snowleopard/hadrian/issues/306)
    + [runhaskell can't find ghc](https://github.com/snowleopard/hadrian/issues/304)
    + [Implement installation and binary/source distribution rules](https://github.com/snowleopard/hadrian/issues/219)
    + [Fix dynamic way](https://github.com/snowleopard/hadrian/issues/4)
    + [Fix validation failures](https://github.com/snowleopard/hadrian/issues/299)
- Submitted some small PRs:
    + [Disable some warnings](https://github.com/snowleopard/hadrian/pull/307)
    + [Add wrapper for Runhaskell, Fix #304](https://github.com/snowleopard/hadrian/pull/305)

## Project Structure

Our major priorities are:

- [Fix dynamic way](https://github.com/snowleopard/hadrian/issues/4)
- [Install and binary distribution generation](https://github.com/snowleopard/hadrian/issues/219)
- [Cross compilation](https://github.com/snowleopard/hadrian/issues/177)

I plan to solve at least these three major issues during the summer, and also some other smaller ones in the meanwhile.

Some general references:

- [GHC build system](https://ghc.haskell.org/trac/ghc/wiki/Building)
- [Shake home](http://shakebuild.com)
- Shake paper: [Shake Before Building: Replacing Make with Haskell](http://ndmitchell.com/downloads/paper-shake_before_building-10_sep_2012.pdf)
- Hadrian paper: [Non-recursive Make Considered Harmful: Build Systems at Scale](https://www.staff.ncl.ac.uk/andrey.mokhov/Hadrian.pdf)

### Fix the Dynamic Way

#### Overview

The dynamic way is a flag, which when turned on, will instruct the build system to create shared libs for packages, such as `base`. So when the dependents are built with dynamic flag on, they will be linked to these shared libraries, rather than static ones.

So basically this is to write build rules for `*.so` objects.

Jose Calderon (@jmct) already had some progress on this issue. I need to talk to him more about this.

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

Implement `install` and `binary-dist` rules in makefiles. This is about writing rules for moving files around and packaging stuff together.

For `install`, it has two command line configurations worth mentioning:

1. `prefix`: the default location for installation, used like `./configure --prefix=...` or `make install --prefix=....`
2. `DESTDIR`: the optional location for installation, used like `make install DESTDIR=...`

#### References

Issue: https://github.com/snowleopard/hadrian/issues/219

Relevant bits in Hadrian: [PR #265: Implement 'sdist-ghc' rule](https://github.com/snowleopard/hadrian/pull/265)

GHC doc: https://ghc.haskell.org/trac/ghc/wiki/Building/Installing

Old build system:

- `rules/bindist.mk`: A rule to add file to binary dist list
- `mk/install.mk.in`: Sets up the installation directories
- install: `ghc.mk:872`: figure out what kinds of files need to be installed, from where, and to where.
- binary-dist: `Makefile`, dispatches to `unix-binary-dist-prep` or `windows-...` (in `ghc.mk`)

### Cross Compilation

#### Overview

We want to support cross-compilation in Hadrian, that user can specify a different target
and build a Stage1 compiler that compiles code for that target.

First, we need a compiler flag to turn on cross compilation. And we need to write some configs in the build settings (like `Settings.hs`).

Second, we need to tweak here and there to make sure that cross compilation config will be considered and lead to some changes in output (like some changes in `CC` flags).

Finally, we need a way to test this -- using an emulator of different architecture would be good enough.

#### References

Issue: https://github.com/snowleopard/hadrian/issues/177

GHC doc:

- https://ghc.haskell.org/trac/ghc/wiki/CrossCompilation
- https://ghc.haskell.org/trac/ghc/wiki/Building/CrossCompiling

Old build system:

- [CrossCompiling vs Stage1Only](https://github.com/ghc/ghc/blob/master/mk/config.mk.in#L555-L572)
- [Stage1Only vs stage=1](https://github.com/ghc/ghc/blob/master/mk/config.mk.in#L574-L603)
- [No stage2 packages when CrossCompiling or Stage1Only](https://github.com/ghc/ghc/blob/3b6a4909ff579507a7f9527264e0cb8464fbe555/ghc.mk#L1448-L1490)

### Test and Validate Support

#### Overview

The [testsuite](https://github.com/ghc/ghc/tree/master/testsuite) are a completely separate part, written for `make`, and currently [used by Hadrian through a python wrapper](https://github.com/snowleopard/hadrian/blob/master/src/Rules/Test.hs). However, to improve the maintainability of this test suite, rewriting it entirely with Shake will be profitable, even though it is a relatively independent part from what we have now.

#### References

Issues:

- https://github.com/snowleopard/hadrian/issues/197
- https://github.com/snowleopard/hadrian/issues/187

GHC doc:

- https://ghc.haskell.org/trac/ghc/wiki/Building/RunningTests/Running
- https://ghc.haskell.org/trac/ghc/wiki/TestingPatches

## Workflow

### Increase the "throughput" of working

One of the hardest part of working on something like Hadrian, is to improve your efficiency of development. For even a tiny bit of chance to the build system, it is highly possible that I have to rebuild another GHC, which can be a burden on my Macbook. Also, we work in very different time zones, and sometimes we can't be very responsive regarding certain discussion. However, there are some ways to improve the "throughput" of the development, thus gaining better overall efficiency:

- **Batch Processing**: I usually deployed multiple pieces of GHC (one standard, two or three Hadrian based). First in this setting, reviewer and I can track two or more different issues concurrently. And secondly, I can compare Shake-based system with Make-based system side-by-side
- **Decomposition**: Effective decomposition of a big task into several more tractable issues can greatly improve the throughput, for the same reason that smaller chucks of rocks gets through the bottleneck easier

### Communication

GitHub will be our major tool of communication, mostly resolving around a specific issue.

For more general stuff, we can discuss through emails. A mail-list might be necessary in future if we find it hard to track the emails (which is unlikely to happen now).

We should try to set up a real-time discussion each week though, preferably closely after my weekly report is read. We can discussion about them in a much faster pace, through either IRC or slack. I think each real time discussion would take around half an hour to one hour.

Also, I will try to keep a openly-accessible weekly report on my completed tasks and planned ones.

## Timeline

**May ~ June**: Familiarize with the project's source code, the usage of Shake, and the usage of Make in GHC. Solve some small issues.

**Before Midterm evaluation (July 18rd)**: Solve one to two major priorities and other smaller issues.

**Before the End of Work (September 2nd)**: Solve the left major priorities. Review on what else needs to be done to actually ship Hadrian.
