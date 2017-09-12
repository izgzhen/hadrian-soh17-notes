Final note on my Summer of Haskell project
===

In this awesome summer, I helped implement three major roadblocks ([dynamic linking](https://github.com/snowleopard/hadrian/pull/325), [installation rules](https://github.com/snowleopard/hadrian/pull/312), and [cross compilation](https://github.com/snowleopard/hadrian/pull/401)) before [Hadrian](https://github.com/snowleopard/hadrian)'s big day -- being merged into the GHC master branch, maybe in the 8.4 version. Besides the three major tickets, I also fixed many other smaller bugs. What's more, I helped improved the tooling & infrastructure of the project, by setting up a full-fledged nightly CI, tracking project roadmap in a better way, and helped make the project more contributor-friendly by tagging, documenting and collecting the easy issues.

In this post, I will touch on the general ideas behind the three milestones, my experience of fighting with them, and how they fit into the larger picture. After that, I will try to summarize the overall status of Hadrian project, from my perspective. I will also briefly talk about the future of this project, esp. on how to make it more friendly to the larger community and attract more contributors.

By the way, although this note is called "final", it only means the end of Summer. My plan is to keep contributing to this project on a long-term basis. One reason is that this project has very few maintainers, unlike Servo; another reason is that there are just a few more steps to be taken to make it a reality for every GHC developers, so how can I miss that event!

## Understanding the Installation Process

The installation of any software is essentially just "copying stuff". But for GHC, it is a bit more complex since we also need to register packages in the database. So the problem is, identify the built artifacts in `_build` directory etc. and try to copy them to the right place.

First thing to note is that the installation tree is different from the build tree. GHC follows UNIX convention, putting them under `prefix`, organized as `lib`, `bin`, `share` etc. A curious bit is that `lib/bin` is where the actual binaries live, while in `bin` are all script wrappers. The main task of wrapper is appending arguments that specifies where the libraries live.

Now, you can invoke `install` target after the build, and use `--install-destdir=<DESTDIR>` to specify a customized installation path.

When implementing this feature, the most painful point is that I have to copy inplace built libraries into a `build` folder at the same level, just because the `ghc-cabal copy` tool hardcoded this path. Patching the source of `ghc-cabal` tool is not enough, since there are other parts in the old system that assume that. From this HACK, you can see that our project is taking a more practical approach towards the final goal of replacing Makefile and unifying all build system relevant bits of GHC. We want a working version first that can be accepted by the wider community and put into production as soon as possible, and continuously improve the system in an incremental way.

## Make it Dynamic

Dynamic linking is the opposite of static linking -- instead of linking objects together at compile time and produce a large binary, you can also link to the shared libraries in pre-defined paths dynamically at runtime, thus saving up the space and increasing the flexibility. Thus, when compiling the libraries, we also need to produce shared ones (`.dll`, `.dylib` or `.so` depending on your OS) aside from static objects (`*.a`).

In this part, I made `dynamic` way work. "Way", is like compilation modes, including vanilla, dynamic, profiled etc. Each flavor combines a different set of ways, since sometimes you want a statically linked GHC (`quick-cross`), but sometimes you don't (`default`).

The essence of this part is to pass proper compiler arguments to GHC, GCC etc., like `-shared`, `-fPIC`. And like what Andrey said, the infrastructure for implementing this is already there (we are already selectively passing compiler flags), so the only thing left is to understand what is missing and add it! Sounds easy enough, but to finish this, I have to come up with my own workflow. We don't always need a new build system -- so some tricks must be invented by yourself.

### First: reverse engineering

Initially, I thought hacking on Hadrian would be translating from Makefiles to `.hs` line by line -- but it turned out to be quite different. First, you must have a deep understanding of what you are really going to implement. Also, a lot of times you need to check out what the Makefile-based system actually *did*, compare the verbose output with Hadrian's, and then implement the diff as new Haskell code. This lets us skip reading unreadable Makefiles entirely, resulting a more Shake-flavor, clean slate implementation.

### Second: proper way of testing

It is very important to clearly specify how you are going to test this feature, esp. for a build system, since the net effect is unclear solely from the `*.hs` diff.

For this reason, we need to compile the program dynamically, and then execute it to see if any error will appear:

```
inplace/bin/ghc-stage2 -dynamic --make ~/Desktop/Playground/Main.hs
```

## Cross compiling GHC

Cross compiling GHC is a GHC that runs on build & host platform (e.g. x86-64), but compiling binaries that run on a different target platform (e.g. ARM). First, there will be no stage2 compiler (so `Stage1Only` is implied in our case), since we don't need to build a compiler that runs on target platform, but only the necessary stage2 libraries which stage 1 GHC can link binary to. Second, building a cross compiler is basically all about using right set of toolchain and add proper arguments to `ghc-cabal` etc., although the code generation pipeline in GHC build system complicates this.

For me, while the real work is done in a single day, exploring the way how to do it effectively and the way to test it conveniently, took me nearly a month.

Change to `src/Settings/Builders/Common.hs` reflects that we might include wrong headers in building cross compiling GHC. This will leads to the size unmatched described in [issue 349](https://github.com/snowleopard/hadrian/issues/349).

Other changes include adding `-f-terminfo` in `ghc-cabal configure` of some packages, so they don't depend on `terminfo` in this mode (we might not readily have curses library for that platform).

The test is fairly complex, involving the use of QEMU, and barely automated. But for sake of simple testing it is good enough. In future, we should set up a SSH-able ARM VM and run built binaries on that.

## Status Report

### Performance

For Hadrian (git hash `29046aa`) and GHC (git hash `b2c2e3e`), we made a simple bench measuring the time needed to build `quickest` with `-j2` on a 2.3GHz dual core Macbook Pro. The Make build system took about 22min, while Hadrian took about 27min.

I haven't done much on performance during these months, but it would be important since one of my friends explicitly asked me about the speed of the new build system.

### Community

Currently, the total number of contributors to this project grows to twenty-one. Andrey Mokhov is leading the development, while three others (including me) contributed significant amount of code. I think it is natural for a build system to have less than normal first-time contributors, but on the other hand, the advantage of switching to Hadrian should include its ease of learning -- and we should validate that through observing the number of new contributors, I guess.

In this year, 46 PRs are landed, 301 commits are created, 32 issues are newly opened and still open, and 32 issues are closed.

We set up a new project page (https://github.com/snowleopard/hadrian/projects), which tracks what need to be done before The Merge.

We also updated our CI infra, including a newly added CircleCI for better OS X support, as well as a customized build bot for longer, more complicated CI targets, such as `default` and `validate` (both of them took more than one hour).

## Future of Hadrian

Current goal of Hadrian is getting ready for The Merge with GHC. In fact, some Hadrian-related change has already been shipped in GHC. So a few more major milestones (freeze stage 1 etc.) plus some bugfixes need to be done at least. Below is from this year's HIW slides.

> As soon as important issues are resolved

> - Freeze Stage1> - Fix validation> - Finalise refactoring/documentation> Keep Make until GHC 8.4.1

In future, we should also document more about the usage of Hadrian and how to contribute to it. We should have a larger community, and make sure that old GHC hackers can switch to this system easily. We also want this project to be useful for hackers who want to build GHC in a different way and find some of our general infrastructure reusable.

Beyond that, we also need better CI support, so a complete matrix of build configurations can be tested.

In long term, we should aim at more completed unification of GHC build system, by e.g. dropping `ghc-cabal` entirely.
