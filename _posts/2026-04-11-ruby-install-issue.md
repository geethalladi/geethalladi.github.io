---
title: "RVM / ruby install with OpenSSL 3.0 in mac"
date: 2026-04-11
layout: post
categories:
- debugging
tags:
- debugging
- installation
- ruby
- rvm
- mac
- openssl
---

# Table of Contents

1.  [Introduction](#org080f5bf)
2.  [The Goal](#orgbb7bf79)
3.  [My Environment](#org164ca45)
4.  [Initial Installation](#orgcea33da)
5.  [Roadblock](#orgc2fd175)
6.  [Understanding the Problem](#org8b549e8)
7.  [Steps I Tried (That Didn’t Work)](#org918dd9c)
8.  [The Breakthrough](#orgf7de78c)
9.  [The Fix](#org6956684)
10. [Key Takeaways](#orgc99f31d)
11. [When You Still Need OpenSSL 1.1](#orgbca1b33)
12. [Final Thoughts](#org0856407)


<a id="org080f5bf"></a>

## Introduction

Setting up Ruby with RVM on modern macOS feels deceptively simple until it isn’t. I ran into an issue involving OpenSSL while trying to
install the latest Ruby. This post documents the problem, failed attempts, and the eventual fix. I hope this saves you a few hours of debugging.


<a id="orgbb7bf79"></a>

## The Goal

-   Install the latest version of RVM
-   Install the latest Ruby (3.4.x at the time of writing)


<a id="org164ca45"></a>

## My Environment

    $ brew config
    HOMEBREW_VERSION: 5.1.5
    Branch: stable
    HOMEBREW_EDITOR: emacsclient
    HOMEBREW_PREFIX: /opt/homebrew
    macOS: # macOS Tahoe
    Rosetta 2: false


<a id="orgcea33da"></a>

## Initial Installation

Following the official [RVM Installation Guide](https://rvm.io/rvm/install)

    # Import GPG keys
    $ curl -sSL https://rvm.io/mpapis.asc | gpg --import -
    $ curl -sSL https://rvm.io/pkuczynski.asc | gpg --import -

    # Install RVM + latest Ruby
    $ \curl -sSL https://get.rvm.io | bash -s stable --ruby


<a id="orgc2fd175"></a>

## Roadblock

The installation failed with an unexpected dependency on openssl@1.1:

    $ brew install automake libtool pkg-config coreutils zlib openssl@1.1 --force

    Warning: No available formula with the name "openssl@1.1".
    Did you mean openssl@3?


<a id="org8b549e8"></a>

## Understanding the Problem


### How RVM Works

-   Checks for precompiled Ruby binaries:

        $ rvm list known
-   Falls back to compiling from source if needed
-   Installs dependencies via Homebrew


### Ruby ↔ OpenSSL Compatibility

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Ruby Version</th>
<th scope="col" class="org-left">OpenSSL Dependency</th>
</tr>
</thead>
<tbody>
<tr>
<td class="org-left">&lt; 3.0</td>
<td class="org-left">openssl@1.0</td>
</tr>

<tr>
<td class="org-left">&gt;= 3.1</td>
<td class="org-left">Works with openssl@3.x</td>
</tr>
</tbody>
</table>


<a id="org918dd9c"></a>

## Steps I Tried (That Didn’t Work)


### Install latest OpenSSL

    $ brew install openssl


### Explicit OpenSSL path

    $ rvm install 3.4.9 --with-openssl-dir=$(brew --prefix openssl)


### Configure PKG CONFIG PATH

    $ brew install pkg-config
    $ export PKG_CONFIG_PATH="$(brew --prefix openssl)/lib/pkgconfig"
    $ rvm install 3.4.9 --with-openssl-dir=$(brew --prefix openssl)


### Try reinstalling RVM

    $ rm -rf ~/.rvm

    $ \curl -sSL https://get.rvm.io | bash -s stable --ruby
    rvm install 3.4.9


### Cleanup Environment Variables

-   Removed stale PATH
-   Cleaned PKG_CONFIG_PATH
-   Cleaned LD_LIBRARY_PATH

None of these approaches resolved the issue.


<a id="orgf7de78c"></a>

## The Breakthrough

    requirements_osx_brew_libs_install ... openssl@1.1


### Root Cause

-   RVM autolibs tries to manage dependencies automatically
-   Misidentifies newer macOS versions
-   Falls back to legacy rules requiring openssl@1.1


<a id="org6956684"></a>

## The Fix

Disable autolibs and install dependencies manually

    $ rvm autolibs disable

    $ brew install automake libtool pkg-config coreutils zlib openssl

    $ rvm install 3.4.9 --with-openssl-dir=$(brew --prefix openssl)

This resolves the issue successfully.


<a id="orgc99f31d"></a>

## Key Takeaways

-   Debugging with LLM is fun. They are highly useful for exploration but still requires manual validation
-   The devil is in the details. Good log messages are still the fastest path to the root cause
-   Timeboxing debugging sessions helps avoid fatigue
-   RVM autolibs can misbehave on newer macOS versions
-   Ruby >= 3.1 works well with OpenSSL 3


<a id="orgbca1b33"></a>

## When You Still Need OpenSSL 1.1

-   Legacy Ruby versions (< 3.0)
-   Older applications with strict dependencies

These setups require manual installation and linking of older OpenSSL versions.


<a id="org0856407"></a>

## Final Thoughts

This was a great reminder:

> The problem is rarely where you first look.

Understanding and questioning system assumptions is often more valuable than repeatedly retrying commands.
