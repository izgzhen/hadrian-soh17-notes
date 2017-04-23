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

### Fix dynamic way

By insert the following `buildDynLib` rule into the `packageRules`, I retrieved the names of `*.so`
to build correctly.

```
buildDynLib :: Rules ()
buildDynLib =
  "_build//*.so" %> so -> error $ show so
```

## Workflow

We sure need two pieces of build, one standard, one hadrian, so we know the difference and can compare side by side

Hadrian team is smaller and the project is not as big and mature as my GSoC project, but I think it is a chance to promote better community working style such as through email/IRC/slack etc.

I will keep a weekly report on my completed tasks and planned ones.

During the proposing period, I will actively use GitHub as a venue of discussion, since we are in different timezones and we want a open discussion so everyone can participate even though we have a relatively small team.

We should try to set up a real-time discussion each week though, preferably closely after my weekly report is read. We can discussion about them in a much faster pace, through either IRC or slack. I think each real time discussion would take around half an hour to one hour.

## Project Structure

### Major Priority

- Dynamic way
- Source and binary distribution generation
- Cross compilation

### Other Issues

- Test suite support (Rewrite `make`-based testsuite with Haskell?)
- Validation support (https://github.com/snowleopard/hadrian/issues/187)
- Provide legacy interface
- iserv-bin requires a wrapper

### Cross Compilation

#### Overview

We want to support cross-compilation in Hadrian, that user can specify a different target
and build a Stage1 compiler that compiles code for that target.

**FIXME**: needs more investigation into what current system does.

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

### Dynamic way

#### Overview

The dynamic way is a flag, which when turned on, will instruct the build system to create shared libs for packages, such as `base`. So when the dependents are built with dynamic flag on, they will be linked to these shared libraries, rather than static ones.

The key is to make rules for `*.so` objects.

#### References

Issue: https://github.com/snowleopard/hadrian/issues/4

GHC doc:

- https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/shared_libs.html#using-shared-libs
- https://ghc.haskell.org/trac/ghc/wiki/Building/Architecture/Idiom/VanillaWay

Old build system:

- `mk/ways.mk`
- `rules/build-package-way.mk`.

### Source and Binary Distribution Generation

#### Overview

Implement `install` and `binary-dist` rules in makefiles.

They are basically just moving files around and packaging stuff together, which should not be very hard.

#### References

Issue: https://github.com/snowleopard/hadrian/issues/219

Relevant bits in Hadrian:

- [PR #265: Implement 'sdist-ghc' rule](https://github.com/snowleopard/hadrian/pull/265)

Old build system:

- `rules/bindist.mk`: A rule to add file to binary dist list
- `mk/install.mk.in`: Sets up the installation directories
- install: `ghc.mk:872`: figure out what kinds of files need to be installed, from where, and to where.
- binary-dist: `Makefile`, dispatches to `unix-binary-dist-prep` or `windows-...` (in `ghc.mk`)

### Test and Validate Support

#### Overview

The testsuite (XXX: link here) are a completely separate part, written in `make`, and currently called by hadrian. However, getting rid of these part of `Makefile`s is also part of the project goal, even if not that urgent (or standing in the way of merging into the GHC master).

#### References

Issue: https://github.com/snowleopard/hadrian/issues/197

GHC doc:

- https://ghc.haskell.org/trac/ghc/wiki/Building/RunningTests/Running
- https://ghc.haskell.org/trac/ghc/wiki/TestingPatches

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
