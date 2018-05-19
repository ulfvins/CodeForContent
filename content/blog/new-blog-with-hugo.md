+++
author = "Richard Ulfvin"
date = "2018-01-22"
description = "Selecting a solution to run this blog"
linktitle = ""
title = "The new blog is up"
type = "post"

+++

So, I finally stated a new blog to share my adventures in code and other related topics. It wasn’t a simple task to select a blogging engine.

I really like the simplicity with static website generators, and initially I started out with Jekyll hosted on Github Pages. This was a suitable candidate for an engine, but I didn’t like the dependencies I ended up with and all the hassle with gems and gem configuration management (kind of reminded me of NPM and all the jokes about massive package libraries). But then a co-worker showed me Hugo, which we also evaluated and now use for a documentation wiki, and I loved it.

Why use a static website generator in the first place you might ask? Well, many people who want to start a blog have a significant caveat: lack of knowledge about a content management system (CMS) or time to learn. In my case, we have used WordPress in a lot of places, and It is super adopted, but I don’t like to maintain an infrastructure (read database and updates from a security perspective). And I also seem to write and edit information in just txt files nowadays, even as markdown format. You see, I tend to spend all my time in VS Code and that is also a nifty tool for simple documentation.

There are many advantages on running static site generators (and Hugo in particular).

* No database, no logins no plugins that require permissions. A dream from a security perspective.
* The site is built with static files, which means lightning-fast serve time. All pages are rendered at deploy time, so your sites footprint on a server is minimal.
* Version control is easy - just use Git, the same tooling you use for your other daily work.
* Everything you write is just Markdown (text files).
* Download a single binary for Hugo and you are off to a flying start.
* Hugo is written in GO and is superfast at building content. 
* It is easy to use the built-in server in Hugo for testing and developing. 
* …Hugo makes writing a website simple and fun again!

## The Initial Setup
This is what I did to setup the solution.

## Step 1
First install Chocolatey for Windows as a package manager, this is easy with powershell.

{{< highlight powershell >}}
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
{{< /highlight >}}

## Step 2
Download and install Hugo with choco command

{{< highlight powershell >}}
choco install hugo -confirm
{{< /highlight >}}

Now all you need to do is setup a location where you want the site to reside on you machine.

{{< highlight powershell >}}
# Create a new directory
New-Item -Type Directory "c:\Sites"

# Change the location
Set-Location "c:\Sites"

# Create the first site
hugo new site myblog

{{< /highlight >}}

The bummer is, right now the site is completely empty. So we need to check out a Theme to download and configure it to be used when we invoke the hugo commands for building the actual site. But first, if you are going to use Git, do a init of the repo before you start with all the templating parts.

## Features for a blog
When I started to look at all the templates that are provide in the themes section of the Hugo homepage, it hit on me that I really needed to have a feature wishlist at hand. So I started to compose a list with requests:

* ```Syntax highlighting``` for code
* ```RSS feed``` for subscription services
* ```Share links``` for social media
* ```Read time``` estimate for posts
* ```About``` page
* ```List item``` pages, for talks etc
* ```Tags and Categories``` for search
* ```Analytics``` for views
* ```Dark theme``` layout (for readers like night cats like me)

Well, no one theme got all of the features but [] ...

## Setup the theme
{{< highlight powershell >}}
git init;\
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke;\
{{< /highlight >}}

# Summary
We have arrived to the end of the first post. If you read this, the blog is alive and this post probably use a lot of the features that I wanted to test out.
So now my blog have a new technology stack up and running with the following components involved:

* VS Code - for editing
* Hugo - for building 
* Github - for source hosting
* Netify - for continuos integration and web hosting