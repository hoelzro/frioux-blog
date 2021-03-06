---
aliases: ["/archives/1616"]
title: "You should be using git grep"
date: "2011-08-14T06:59:05-05:00"
tags: [ack, git, grep]
guid: "http://blog.afoolishmanifesto.com/?p=1616"
---
Usually when searching through files I use [ack](http://betterthangrep.com/)
which is an awesome tool indeed.

Unfortunately, though ack does indeed work on windows, using it on windows is a
painful experience. The main two problems are that it's slow and the color
coding doesn't work.

I figured I'd try out [git
grep](https://www.kernel.org/pub/software/scm/git/docs/git-grep.html), with the
hope that it might be marginally better. I try my best to at least be familiar
with all the git commands, so this is one of those things I had been meaning to
do anyway.

I was in for a pleasant surprise when I found that git-grep's color coding
worked and it is extremely fast. I profiled it on linux and I get about an order
of magnitude difference for a simple repo. If you use a repo with submodules the
difference is even more visible, though possibly you would actually want to
search submodules by default.

There are two features of ack that I use regularly. The first is that it doesn't
search files it shouldn't, like .svn and .git. With git grep that is also the
case, though it only works with git checkouts (which is all I have at this
point.) The second feature is the ability to do --perl or --js or --java or
whatever else. This can also be done easily with git grep, and I may make a bash
wrapper called git-ack that just gives the interface of ack with the backend of
git grep.

      ack frew --js
      git grep frew -- '*.js'

Sure, it's marginally longer, but if I do make the wrapper it will just be 'git
ack frew --js' and given that I have cool [shell settings based on the directory
I'm in](https://github.com/cxreg/smartcd) I will set it up such that 'ack frew
--js' just runs 'git ack frew --js'

Also, check out the amazing -p argument for git grep; it uses git's code parsing
to include the function that matches are in.

---

(The following includes affiliate links.)

If you're interested in learning more about Git, I cannot recommend
<a  href="https://www.amazon.com/gp/product/1484200772/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1484200772&linkCode=as2&tag=afoolishmanif-20&linkId=73f85964b6ab98ea870583701b7e77aa">Pro Git</a><img src="//ir-na.amazon-adsystem.com/e/ir?t=afoolishmanif-20&l=am2&o=1&a=1484200772" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
enough.  It's an excellent book that will explain how to use Git day-to-day, how
to do more unusual things like set up Git hosting, and underlying data
structures that will make the model that is Git make more sense.
