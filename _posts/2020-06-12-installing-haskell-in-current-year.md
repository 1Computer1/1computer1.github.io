---
title: "Installing Haskell in Current Year"
date: 2020-06-12
tags: Haskell
---

After seeing first-timers fail to install GHC in various ways, I decided to write a quick guide on how to install Haskell.  

# Installation Woes

I bet you've seen [this post](https://reddit.com/r/haskell/comments/guecax/confused_by_the_haskell_installation_options/) or something similar. Or maybe you're someone who ran into similar issues. Either way, after seeing first-timers fail to install GHC in various ways, I decided to write a quick guide on how to install Haskell. The point of this post won't be on how to install the best setup or anything like that, but rather for people to choose a method that sounds easy to them so that they can get on with trying out Haskell for the first time.  

For clarification:

- GHC is the Glasgow Haskell Compiler. It is *the* compiler for working with Haskell.
- [cabal](https://www.haskell.org/cabal/users-guide/intro.html) (or cabal-install) is the build tool for building your projects and managing dependencies, among other things. Not to be confused with Cabal, which is the backend library used by cabal.
- [Stack](https://docs.haskellstack.org/en/stable/README/) is an alternative to cabal, created back when cabal used to be much worse at doing its job.

As for whether you should use cabal or Stack, this is a point of contention in the Haskell community. But, if you are starting out in Haskell, it really does not matter which you choose, since they do the same thing in the end: compiling your code. You can always try the other later, or even use both depending on your needs (I do this!). If you are interested in the differences, take a look [here](https://gist.github.com/merijn/8152d561fb8b011f9313c48d876ceb07).  

I'll also touch on the current IDE integration situation in the Haskell ecosystem, as well as linters and formatters.  

# On Windows

As we all know, Windows is a pain for development. In fact, at the time of this writing, I've had to guide a friend through several methods to install GHC and each one of them came up with an error of some kind. So, you probably should use [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) if you haven't been using that already. Then, just skip right down to the Linux section and maybe configure your favorite editor like [VSCode to work in WSL](https://code.visualstudio.com/docs/remote/wsl).  

If you're set on sticking to Windows though, here are two ways to get started with Haskell.  


## Chocolatey

The recommended method to install GHC and cabal on Windows at the moment is through [Chocolatey](https://chocolatey.org/). You can simply follow the [instructions on the Haskell website](https://www.haskell.org/platform/windows.html).  

## Stack

[Stack](https://docs.haskellstack.org/en/stable/README/) is tool for developing Haskell projects, meant to be an alternative to cabal. It is extremely simple to start up, simply click on that link and get the installer. Stack is quite useful on Windows since it manages multiple versions of GHC automatically as a standalone application.  

# On Linux/macOS

There are multiple options here, but much of Haskell development is done on Linux, so you should have much less trouble installing GHC here.  

## ghcup

[ghcup](https://gitlab.haskell.org/haskell/ghcup-hs) manages GHC and cabal installations and lets you switch between different versions. You can get it and install GHC and cabal soon after [by following the instructions here](https://www.haskell.org/ghcup/#) (if you are browsing on Windows, click 'display all supported installers').  

## Stack

[Stack](https://docs.haskellstack.org/en/stable/README/) is cross-platform, so you can install it on Linux/macOS too. I recommend it less if you can get ghcup working, but if you can't, then go ahead and try Stack.  

## PPA

If you are on Ubuntu and prefer a PPA instead, you can try the [ghc ppa](https://launchpad.net/~hvr/+archive/ubuntu/ghc) to install GHC and cabal. Simply run the following commands, replacing `XXX` and `YYY` with your desired versions.  

```sh
sudo add-apt-repository -y ppa:hvr/ghc
sudo apt-get update
sudo apt-get install -y cabal-install-XXX ghc-YYY
```

# Regarding cabal's Documentation

[cabal](https://www.haskell.org/cabal/users-guide/intro.html)'s documentation is a bit long-winded, and it has a long history, so much of what you might find online might be outdated. So, I'll direct you to the important parts of the documentation:

- [Section 3.1](https://www.haskell.org/cabal/users-guide/developing-packages.html) tells you how to start a project.
- [Section 5.1](https://www.haskell.org/cabal/users-guide/nix-local-build.html) tells you how to build and run a project.
- [Section 5.4](https://www.haskell.org/cabal/users-guide/nix-local-build.html#commands) lists many of the commands you can use.  

Note that many commands are prefixed with `v2-` or `new-`. In cabal versions 3.0.0.0 and above, the prefix is no longer necessary.  

# The IDE Situation

## Integration

Currently, the IDE situation in Haskell is a little bit disappointing, with things being difficult to get working. I'll list some options you have in order of how quickly you can probably get it working:

- [Simple GHC (Haskell) Integration](https://marketplace.visualstudio.com/items?itemName=dramforever.vscode-ghc-simple) is a plugin for VSCode. It uses GHCi to do its thing, which means there's no extra tools to install other than the extension, and it should work on all platforms. I highly recommend this for most people.

- [haskell-mode](https://github.com/haskell/haskell-mode) is an Emacs mode for Haskell. If you're using Emacs, you can install this, I'd hope.

- [ghcid](https://github.com/ndmitchell/ghcid) is a simple application that uses GHCi to show errors and warnings in your code. There are plugins for various editors.

- [ghcide](https://github.com/digital-asset/ghcide), [haskell-ide-engine](https://github.com/haskell/haskell-ide-engine) (HIE), and [haskell-language-server](https://github.com/haskell/haskell-language-server) (HLS). The history of these three, you can read [here](https://neilmitchell.blogspot.com/2020/01/one-haskell-ide-to-rule-them-all.html). The gist of it is that ghcide and HIE has been consolidated into HLS. I do not recommend using it at the moment because it is still in early stages, but once its mature, it will probably be the standard IDE integration tool, so keep an eye on it! Various editors are supported.

## Linting and Formatting

There's really only one linter that I know of, which is [hlint](https://github.com/ndmitchell/hlint). It works fairly well and has plugins for various editors, such as [VSCode](https://marketplace.visualstudio.com/items?itemName=hoovercj.haskell-linter). It should be also integrated in HIE and HLS eventually.  

As for formatting, I do not use a formatter myself, but here are some that I've seen being used:

- [brittany](https://github.com/lspitzner/brittany/)
- [floskell](https://github.com/ennocramer/floskell)
- [ormolu](https://github.com/tweag/ormolu) which uses a one "true" style.
- [stylish-haskell](https://github.com/jaspervdj/stylish-haskell) which formats tedious things like import statements, pragmas, alignment, etc.

# Conclusion

That's pretty much it, enjoy playing around with Haskell!  

If you're reading this and spot mistakes or misleading information, please let me know.  
