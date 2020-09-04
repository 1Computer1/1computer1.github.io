---
title: "Installing Haskell in Current Year"
date: 2020-07-28
tags: Haskell
---

After seeing first-timers fail to install GHC in various ways, I decided to write a quick guide on how to install Haskell. The point of this post won't be on how to install the best setup or anything like that, but rather for people to choose a method that sounds easy to them so that they can get on with trying out Haskell for the first time.  

For clarification:

- GHC is the Glasgow Haskell Compiler. It is *the* compiler for working with Haskell.
- [cabal](https://cabal.readthedocs.io/) (or cabal-install) is the build tool for building your projects and managing dependencies, among other things. Not to be confused with Cabal, which is the backend library used by cabal.
- [Stack](https://docs.haskellstack.org/en/stable/README/) is an alternative to cabal, created back when cabal used to be much worse at doing its job.

As for whether you should use cabal or Stack, this is a point of contention in the Haskell community. For beginners, it does not matter which you choose, since they do the same thing in the end: building your projects. If you are interested in the differences, take a look [here](https://gist.github.com/merijn/8152d561fb8b011f9313c48d876ceb07).  

I'll also touch on the current IDE integration situation in the Haskell ecosystem, as well as linters and formatters.  

# On Linux/macOS

There are multiple options here, but much of Haskell development is done on Linux, so you should have much less trouble installing GHC here.  

## ghcup

[ghcup](https://gitlab.haskell.org/haskell/ghcup-hs) manages GHC and cabal installations and lets you switch between different versions. You can get it and install GHC and cabal soon after [by following the instructions here](https://www.haskell.org/ghcup/#) (if you are browsing on Windows, click 'display all supported installers').  

## Stack

[Stack](https://docs.haskellstack.org/en/stable/README/) is tool for developing Haskell projects, meant to be an alternative to cabal. It is very simple to start up, simply click on that link and get the installer.  

# On Windows

As we all know, Windows is a pain for development. So, you probably should use [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) if you haven't been using that already. Then, just go to the Linux section and maybe configure your favorite editor like [VSCode to work in WSL](https://code.visualstudio.com/docs/remote/wsl).  

If you're set on sticking to Windows though, here are some ways to get started with Haskell.  

## Chocolatey

The recommended method to install GHC and cabal on Windows at the moment is through [Chocolatey](https://chocolatey.org/). You can simply follow the [instructions on the Haskell website](https://www.haskell.org/platform/windows.html).  

## ghcups

[ghcups](https://github.com/kakkun61/ghcups) manages GHC and cabal installations and lets you switch between different versions. It is similar to [ghcup](https://gitlab.haskell.org/haskell/ghcup-hs).  

## Stack

[Stack](https://docs.haskellstack.org/en/stable/README/) is cross-platform, so you can install it on Windows too.  

# Memory Usage

A known problem with GHC is that it takes up a lot of memory when compiling. If you are working in a low-memory environment (a VPS, a Raspberry PI, an old laptop), then you'll most likely run out of memory compiling large libraries or executables.  

Using cabal, append `--ghc-options="+RTS -M<memory> -RTS"` to your commands. For example, `cabal build --ghc-options="+RTS -M600M -RTS"` to build a project without using more than 600 megabytes.  

Unfortunately, I do not know of a Stack-specific way to limit its memory usage while installing. You can set the maximum number of parallel jobs with `-j<number>` e.g. `stack build -j1`, but this does not limit memory usage.  

Another thing you can do is set the `GHCRTS` environment variable:  

```sh
GHCRTS='-M<memory>'
export GHCRTS
```

This will apply it to all GHC compiled programs. However, if a program was not compiled to be compatible with RTS options, you will get an error and should remove the environment variable.  

# The IDE Situation

## Integration

I'll list some options you have in order of how quickly you can probably get it working:  

- [haskell-language-server](https://github.com/haskell/haskell-language-server) is a high power tool that uses the language server protocol, so it can integrate with any editor that supports LSP. HLS uses a plugin system, which means it includes many different tools within it, such as code diagnostics, linters, and formatters. There are prebuilt binaries available for Windows, Linux, and macOS. The [VSCode extension](https://marketplace.visualstudio.com/items?itemName=haskell.haskell) will also download the binaries automatically for you.

- [Simple GHC (Haskell) Integration](https://marketplace.visualstudio.com/items?itemName=dramforever.vscode-ghc-simple) is a plugin for VSCode. It uses GHCi to do its thing, which means there's no extra tools to install other than the extension, and it should work on all platforms.

- [haskell-mode](https://github.com/haskell/haskell-mode) is an Emacs mode for Haskell. You can take a look at the link for installation instructions.

- [ghcid](https://github.com/ndmitchell/ghcid) is a simple application that uses GHCi to show errors and warnings in your code. There are plugins for various editors.

## Linting and Formatting

Linters:  

- [hlint](https://github.com/ndmitchell/hlint), which works fairly well and has plugins for various editors, such as [VSCode](https://marketplace.visualstudio.com/items?itemName=hoovercj.haskell-linter).
- [Stan](https://github.com/kowainik/stan), a more in-depth static code analyzer. It can generate pretty reports for your project.

Formatters:  

- [ormolu](https://github.com/tweag/ormolu), a formatter using a one "true" formatting style with no configuration.
- [stylish-haskell](https://github.com/jaspervdj/stylish-haskell), a formatter that tries to not get in the way and focuses on tedius things like imports, whitespace, alignment, etc.

# Conclusion

That's pretty much it, enjoy playing around with Haskell! If you spot mistakes or misleading information, please let me know.  
