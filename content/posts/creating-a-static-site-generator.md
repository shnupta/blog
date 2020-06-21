---
title: "Creating a Static Site Generator"
date: 2020-06-21
tags: ["c", "tent", "university"]
categories: ["projects"]
---

The last few weeks I've been completing my summer project for university. This project consisted of 3 parts: an assembler, an emulator (both for the ARM11 architecture) and then a free-choice extension project written in C.

We were put into groups of 4 to complete all of these tasks which was a great experience to work tightly as a group. We shared tasks in a few different ways which was insightful.

We split ourselves into two groups of two at the beginning, and I was working on the assembler. Building the assembler was good fun. Just to see something that you've written, convert source code into an executable file is nice.

Following the completion of both the assembler and emulator we decided as a group to build a static site generator as our extension. I had originally had this idea and was going to build it in Go but I thought it would be a good opportunity to build it now, plus the rest of the group thought it would be a cool project.

So we set out to build it, we called it tent...

The basic requirements were to have some sort of theme mixed with content files written in markdown. We also needed inserts, a way of placing files and content into other files.

By the time we started working on tent we only had around 10 days until the final deadline, and given that none of us were particularly familiar with the code behind static site generators we didn't have much of an idea as to how far we could get. 

We started by splitting the work up into a markdown to html converter, and a templating system. This was slightly naive as we didn't look into the amount of work required for each beforehand, we kind of assumed they carried a similar workload. This was definitley not the case. It turned out that the markdown converter was required considerably more effort and unfortunately the team member that spent a few days on this wasn't rewarded for their work - something we've learnt from.

We took some inspiration for templating and inserts from other static site generators by going with the `{{ insert }}` syntax. We then supported a few insert types: templates, snippets, configs and variables. Templates and snippets are both html style files but templates are full pages that typically have content inserted into them, whereas snippets are typically small bits of html inserted into other files. Configs and variables can be either strings or arrays of strings. Configs are set in the config.tent file at the root of a website, while variables are set in the metadata of each content file and can only be used inside that content file.

We ran into many problems along the way with memory management which was expected as we were using C and handling lots of strings. However we managed to resolve all of these and we've produced a tool we are all really proud of. I think I'm going to rewrite my website and use tent to manage it!

Here are some examples of content files and templates:

Content file:
![example content file](/images/tent/tent-content-file.png)

Template file:
![example template file](/images/tent/tent-template-file.png)

We haven't published tent publically yet but when we do I'll link it here!

--------------
This is the fourth post of my [weekly post challenge!]({{< ref "weekly-post-challenge.md" >}})
