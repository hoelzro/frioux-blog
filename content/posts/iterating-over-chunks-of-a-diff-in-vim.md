---
title: Iterating over Chunks of a Diff in Vim
date: 2016-05-25T22:41:14
tags: [ziprecruiter, vim, git, toolsmith]
guid: "https://blog.afoolishmanifesto.com/posts/iterating-of-chunks-of-a-diff-in-vim"
---
Every now and then at work I'll make broad, sweeping changes in the codebase.
The one I did recently was replacing all instances of `print STDERR "foo\n"`
with `warn "foo\n"`.  There were about 160 instances in all that I changed.
After discussing more with my boss, we discussed that instead of blindly
replacing all those print statements with warns (which, for those who don't
know, are easier to intercept and log) we should just log to the right log
level.

<!--more-->

## Enter Quickfix

Quickfix sounds like some kind of bad guy from a slasher movie to me, but it's
actualy a super handy feature in Vim.  Here's what the manual says:

> Vim has a special mode to speedup the edit-compile-edit cycle.  This is
> inspired by the quickfix option of the Manx's Aztec C compiler on the Amiga.
> The idea is to save the error messages from the compiler in a file and use Vim
> to jump to the errors one by one.  You can examine each problem and fix it,
> without having to remember all the error messages.

More concretely, the quickfix commands end up giving the user a list of
locations.  I tend to use the quickfix list most commonly with
[Fugitive](https://github.com/tpope/vim-fugitive).  You can run the command
`:Ggrep foo` and the quickfix list will contain all of the lines that git found
containing `foo`.  Then, to iterate over those locations you can use `:cnext`,
`:cprev`, `:cwindow`, and many others, to interact with the list.

I have wanted a way to populate the quickfix list with the locations of all of
the chunks that are in the current modified files for a long time, and this week
I decided to finally do it.

First off, I wrote a little tool to parse diffs and output locations:

```
#!/usr/bin/env perl

use strict;
use warnings;

my $filename;
my $line;
my $offset = 0;
my $printed = 0;
while (<STDIN>) {
   if (m(^\+\+\+ b/(.*)$)) {
      $printed = 0;
      $filename = $1;
   } elsif (m(^@@ -\d+(?:,\d+)? \+(\d+))) {
      $line = $1;
      $offset = 0;
      $printed = 0;
   } elsif (m(^\+(.*)$)) {
      my $data = $1 || '-';
      print "$filename:" . ($offset + $line) . ":$data\n"
         unless $printed;
      $offset++;
      $printed = 1;
   } elsif (m(^ )) {
      $printed = 0;
      $offset++;
   }
}
```

The general usage is something like: `git diff | diff-hunk-list`, and the output
will be something like:

```
app/lib/ZR/Plack/Middleware/AccessLog.pm:195:  local $SIG{USR1} = sub {
bin/zr-plack-reaper:22:-
bin/zr-plack-reaper:29:sub timeout { 120 }
```

The end result is a location for each new set of lines in a given diff.  That
means that deleted lines will not be included with this tool.  Another tool
or more options for this tool would have to be made for that functionality.

Then, I added the following to my vimrc:

```
command Gdiffs cexpr system('git diff \| diff-hunk-list')
```

So now I can simply run `:Gdiffs` and iterate over all of my changes, possibly
tweaking them along the way!

### Super Secret Bonus Content

The Quickfix is great, but there are a couple other things that I think really
round out the functionality.

First: the Quickfix is global per session, so if you do `:Gdiffs` and then
`:Ggrep` to refer to some other code, you've blown away the original quickfix
list.  There's another list called the location list, which is scoped to a
window.  Also very useful; tends to use commands that start with `l` instead of
`c`.

Second: There is another Tim Pope plugin called
[unimpaired](https://github.com/tpope/vim-unimpaired) which adds a ton of useful
mappings; which includes `[q` and `]q` to go back and forth in the quickfix, and
`[l` and `]l` to go back and forth in the location list.  Please realize that
the plugin does *way* more than just those two things, but I do use it for those
the most.

---

(The following includes affiliate links.)

If you'd like to learn more, I can recommend two excellent books.  I
first learned how to use vi from
<a href="https://www.amazon.com/gp/product/059652983X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=059652983X&linkCode=as2&tag=afoolishmanif-20&linkId=1d3b90d608a023a1dcb898b903b6f6ac">Learning the vi and Vim Editors</a><img src="//ir-na.amazon-adsystem.com/e/ir?t=afoolishmanif-20&l=am2&o=1&a=059652983X" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />.
The new edition has a lot more information and spends more time on Vim specific
features.  It was helpful for me at the time, and the fundamental model of vi is
still well supported in Vim and this book explores that well.

Second, if you really want to up your editing game, check out
<a href="https://www.amazon.com/gp/product/1680501275/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1680501275&linkCode=as2&tag=afoolishmanif-20&linkId=4518880cd2a7fd1333456edcbacc26f6">Practical Vim</a><img src="//ir-na.amazon-adsystem.com/e/ir?t=afoolishmanif-20&l=am2&o=1&a=1680501275" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />.
It's a very approachable book that unpacks some of the lesser used features in
ways that will be clearly and immediately useful.  I periodically review this
book because it's such a treasure trove of clear hints and tips.
