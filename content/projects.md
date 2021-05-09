+++
title = "Projects"
menu = "main"
+++

Here is a quick showcase of some of the personal projects I've developed.

## Contents
- [bric](#bric)
- [Chip-Emul8](#chip-emul8)
- [tent](#tent)
- [WACC](#wacc)

## bric
bric is a text editor based on kilo.

bric does not depend on any library (not even curses). It uses fairly standard VT100 (and similar terminals) escape sequences to write and read to and from the terminal.

Read the blog post [here](https://caseywilliams.me/2020/bric-text-editor).

## Chip-Emul8
Chip-Emul8 was the first emulator I ever wrote. It is written in C and emulates the [CHIP-8](https://en.wikipedia.org/wiki/CHIP-8) virtual machine from the 70s.

I actually livestreamed this however the site I did it on is no longer around so the recordings are lost. It was a lot of fun and taught me the basics of computer processors and the execution cycle etc.

I was originally inspired by [Ferris](http://iamferris.com/).

## tent
tent is a simple static site generator written in C. It was our extension for the first year C project at Imperial. Check it out on [github](https://github.com/julesdehon/tent).

## WACC
For our spring term project we were tasked with writing a compiler from scratch for a basic language called WACC. We decided to write the project in C++ (while at the time we all had very little to no experience with it) in order to practice for summer internships. 

We used ANTLR4 as the parser and lexer and then designed and coded the intermediate representations and code generation as well as implement optimisations such as register allocation by graph colouring, constant folding, side-affecting expressions and classes. 

This was one of the projects I've had the most fun doing so far, I learnt so much about C++ and its different features as well as finally creating a compiler after so many years of "just finding them interesting".

## SeeMyFeels
In March 2021 we competed in the DoCSoc IC Hello World hackathon. It was my first hackathon ever and I had a great time competing with everyone. We stayed up for a full 30 hours working on SeeMyFeels, a generative art project where you feed in some audio and a unique piece of digital art is generated using features from a Tensorflow model. [Github](https://github.com/shnupta/SeeMyFeels).
