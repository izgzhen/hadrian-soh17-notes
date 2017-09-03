Final note on my Summer of Haskell project
===

In this summer, I helped implemented three major roadblocks before Hadrian can be merged into the GHC master branch. In this process, I also identified and fixed other issues in the Hadrian implementation. What's more, I helped improved the tooling & infra side of the project, by setting up a full-fledged nightly CI, created new ways of tracking project roadmap, and helped make the project more contributor-friendly by documenting and tagging easy issues.

In this note, I will touch on the general ideas behind the three milestones, my experience of fighting with them, and how they fit into the larger picture. After that, I will try to summarize the status of Hadrian project, from my perspective. I will also briefly talk about the future of Hadrian, esp. on how to make it more friendly to the larger community and attract more contributors.

By the way, although this note is called "final", it only means the end of Summer. My plan is to keep contributing to this project, on a more long-term basis. One reason is that this project has very few maintainers, unlike Servo; another reason is that there are just a few more steps to be taken to make it a reality for every GHC developers, so how can I miss that event!

## Understanding the Installation Process

The installation is essentially copying for many other softwares. But for GHC, it is a bit complex since we also need to register packages in the database. So the problem is, identify the built artifact in `_build` directory etc. and try to copy them to the proper place.

First thing to note is that installation tree is different from build tree. It follows UNIX convention, putting them under `prefix`, organized as `lib`, `bin`, `share` etc. A curious bit is that `lib/bin` is where the actual binaries live, while in `bin` are all script wrappers. The main task of wrapper is attaching argument that specifies where the libraries are.

Now, you can invoke `install` target after the build, and use `--install-destdir=<DESTDIR>` to specify a customized installation path.

During implementing this feature, the most painful point is that I have to copy built libraries into a `build` folder at the same level, just because the `ghc-cabal copy` tool hardcoded this. Patching the source of `ghc-cabal` tool is not enough, since there are other parts in the old system that assumes that. From this HACK, you can see that Hadrian is taking a more practical approach towards the final goal of replacing Makefile and unifying all build system relevant bits of GHC. We want a working version first that can be accepted by the wider community and put into production as soon as possible, and improving the system in an incremental way.

## Make it Dynamic

In this part, I made `dynamic` way work. "Way", is like compilation modes, including vanilla, dynamic, profiled ... Each flavor has different set of ways, because sometimes you want a statically linked GHC, but sometimes you don't.

The essence of this part is to pass proper compiler arguments to GHC, GCC etc., like `-shared`, `-fPIC`.

Like what Andrey said, the infrastructure for implementing this is already there, so only thing left is to understand what is missing and add it! Sounds easy enough, but to finish this, I have to come up with my own workflow. We don't always build a new build system -- so some tricks can only be invented.

### First: reverse engineering

Initially, I thought hacking on Hadrian would be translating from Makefiles to `.hs` line by line -- but it turned out to be quite different. You must have a deep understanding of what you are really going to implement. Also, in a lot of times you need to check out what did the Makefile-based system do, compare the verbose output with Hadrian's, and then reflecting the diff back to Haskell code. This lets us skip reading unreadable Makefiles entirely, resulting a more Shake-flavor, clean slate implementation.

### Second: proper way of testing

It is very important to specify clearly how you are going to test this feature, esp. for a build system, since the total effect is unclear only from the code diff.

For this case, we need to compile the program dynamically, and then execute it to see if any error will appear:

```
inplace/bin/ghc-stage2 -dynamic --make ~/Desktop/Playground/Main.hs
```

## Cross compiling GHC

Cross compiling GHC is a GHC that runs on build/host platform (e.g. x86-64), but compiling binaries that run on target platform (e.g. ARM). First, there will be no stage2 compiler (so `Stage1Only` is implied in our case), since we don't need to build a compiler that runs on target platform, but only the necessary stage2 libraries which stage 1 GHC can link binary to. Second, building a cross compiler is basically using right set of toolchain and add proper arguments to `ghc-cabal` etc., although the complex code generation pipeline in GHC build system complicates this.

For me, while the real work is done in one day, exploring the way how to do it effectively and the way to test it conveniently, took me nearly a month.

Change to `src/Settings/Builders/Common.hs` reflects that we might include wrong headers in building cross compiling GHC. This will leads to the size unmatched described in https://github.com/snowleopard/hadrian/issues/349.

Other changes are adding `-f-terminfo` in `ghc-cabal configure` of some packages, so they don't depend on `terminfo` in this mode (we might not readily have curses library for that platform).

The test is fairly complex, involving the use of QEMU, and barely automated. But sake of simple testing it is good enough. In future, we should set up a SSH-able ARM VM and run built binaries on that.

## Status Report

### Performance

For Hadrian (git hash `29046aa`) and GHC (git hash `b2c2e3e`), we made a simple bench measuring the time needed to build `quickest` with `-j2` on a 2.3GHz dual core Macbook Pro.


The Make build system took about 22min, while Hadrian took about 27min.

### Community

Currently, the total number of contributors to this project counts up to 21. Andrey Mokhov is leading the development, while three others (including me) contributed significant amount of code.

In this year, 46 PRs are landed, 301 commits are created, 32 issues are newly opened and still open, and 32 issues are closed.

We set up a new project page (https://github.com/snowleopard/hadrian/projects), which tracks what needs to be done before The Merge.

We also updated our CI infra, including a newly added CircleCI for better OS X support, as well as a customized build bot for longer, more complicated CI targets, such as `default` and `validate` (both of them took more than one hour).

## Future of Hadrian

Current goal of Hadrian is getting ready for The Merge with GHC. In fact, some Hadrian-related change has already been shipped in GHC. So a few more major milestones (freeze stage 1 etc.) plus some bugfixes need to be done at least.

In future, we should also have more text documenting the usage of Hadrian and how to contribute to it. We should have a larger community, and make sure that old GHC hackers can switch to this system as fluent as possible. We also want this project to be useful for hackers who want to build GHC in a different way, but still finding some of our general infrastructure reusable.

Beyond that, we also need a more complete set of CI support, so a matrix of build configurations can be tested.

In long term, we should aim at more completed unification of GHC build system, by e.g. dropping `ghc-cabal` entirely.