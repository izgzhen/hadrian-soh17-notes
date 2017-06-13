Summer of Haskell 2017: Improve the Shake-Based Hadrian Build System for GHC
====

## Links

- [Original idea](https://summer.haskell.org/ideas.html#hadrian-ghc)
- [Project homepage](https://github.com/snowleopard/hadrian/)
- [Proposal draft](proposal.md)

## Status

### Opened Issues

- [Document on debugging Hadrian](https://github.com/snowleopard/hadrian/issues/308)
- [Fix dynamic way](https://github.com/snowleopard/hadrian/issues/4)
- [Implement installation and binary/source distribution rules](https://github.com/snowleopard/hadrian/issues/219)
- [Support building cross compilers](https://github.com/snowleopard/hadrian/issues/177)
- [ghc-cabal copy assumes library artifacts in build subpath](https://github.com/snowleopard/hadrian/issues/318)

### Pending PRs

- [Add Install Rules](https://github.com/snowleopard/hadrian/pull/312)
- [Fix documentation rules](https://github.com/snowleopard/hadrian/pull/324)

### Merged PRs

- [Add copyFileUntracked](https://github.com/snowleopard/hadrian/pull/313)
- [Add Werror to CC and HC](https://github.com/snowleopard/hadrian/pull/309)
- [Disable some warnings](https://github.com/snowleopard/hadrian/pull/307)
- [Add wrapper for Runhaskell, Fix #304](https://github.com/snowleopard/hadrian/pull/305)
- [Fix implicit assumption about inplace installation etc.](https://github.com/snowleopard/hadrian/pull/315)
- [Fix CABAL_VERSION argument in building ghc-cabal](https://github.com/snowleopard/hadrian/pull/319)
- [Add binary wrappers for hp2ps, hpc, hsc2hs](https://github.com/snowleopard/hadrian/pull/321)
- [Add more utilities including install and symbolic link](https://github.com/snowleopard/hadrian/pull/316)


## This Week in Hadrian

- [Week 1](weekly-notes/week1.md)
- [Week 2](weekly-notes/week2.md)
- [Week 3](weekly-notes/week3.md)
- [Week 4](weekly-notes/week4.md)
