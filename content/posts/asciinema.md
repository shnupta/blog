---
title: "Asciinema"
date: 2020-05-28
tags: ["terminal", "tools"]
categories: ["cool-stuff"]
---

I was looking for a tool to record my terminal output so I could show some example output of programs I had written for some friends.

After searching "record terminal output", the first result to appear was [Asciinema](https://asciinema.org).

This post is just a quick overview of what you can do with Asciinema and why I like it.

# Installation
Installing Asciinema is super easy. 

If you use Homebrew:
```
brew install asciinema
```
Pip:
```
sudo pip3 install asciinema
```
Ubuntu:
```
sudo apt-add-repository ppa:zanchey/asciinema
sudo apt-get update
sudo apt-get install asciinema
```

See their [installation options](https://asciinema.org/docs/installation) page if you use a different distro.

# Basic usage
The best part about Asciinema is the simplicity: no accounts needed, everything can be done via terminal. It just works.

To get recording, type:
```
asciinema rec
```

And to stop, type:
```
exit
```
or simply press Ctrl-D.

As soon as you finish recording you are presented with the option to upload directly to asciinema.org or save the cast locally. If you upload then you are given a link directly to the cast in the terminal - lovely!

There are of course more options that you can do with asciinema suching as using the `-t` option to give your cast a title, specifying a maximum length, and also playing casts at given URL.

I urge you to check them out for yourselves. See the command options with:
```
asciinema -h
```

# Output
The output of Asciinema is also great. Here is an example:
{{< asciinema id="asciicast-0DNPVQOaER9dH2OV11gNwPYPG" src="https://asciinema.org/a/0DNPVQOaER9dH2OV11gNwPYPG.js" rows="25" >}}

As you can see, terminal colours are correctly outputted. You can also change the theme of the terminal and many other things if you connect an account to your recordings!

Something **REALLY** cool that I discovered yesterday is that you can actually copy any of the text present in the cast! Go ahead and try it yourself. Pause the cast above and try copying some of the output for yourself. Awesome right?

There are a few more things you can do with the embedded script I used above such as changing playback speed amongst others but I'll leave all of that fun stuff to you.

I don't know what you think, but I think it's a pretty neat little tool for sharing terminal snippets. So far I've used it to help a friend see the functionality of some Vim commands and also to quickly show ideas of script outputs for a group project I'm working on for uni.

--------------
This is the first post of my [weekly post challenge!]({{< ref "weekly-post-challenge.md" >}})
