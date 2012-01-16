--- 
wordpress_id: 98
layout: post
title: "Brain Dump: VIM commands"
date: 2011-08-25 17:48:26 -04:00
wordpress_url: http://www.jessesanford.com/?p=98
---
This is the first of a many quick brain dumps I will be doing of helpful keyboard-commands for things like bash, vim, OS X, etc. Check back for more goodies and feel free to comment with your own!

<pre>Miscelaneous:
:set number (:set nu) shows line numbers.
:set nonumber (:set nonu) removes line numbers.

:sh opens shell in current directory
:! [command] executes command and returns output.

Copy and paste:
yy copy 1 line.
5y copy 5 lines.

dd cut 1 line.
5dd cut 5 lines.

p Paste under current line.

m[char] mark spot [char]. *(char can be any alpha-numeric character)
`[char] go to mark [char]. *(alpha-numeric character used as a mark)

y`[char] yank from current cursor to mark [char]

. repeat last command.

Moving:
:n go to line n
gg move to top of file.
G move to end of file.
w move one word forward.
b move one word backward.
} move one paragraph forward.
{ move one paragraph backward.

Line shifting/indenting:
>> shifts a line one shiftwidth to the right.
<< shifts a line one shiftwidth to the left.
== auto indents the current line based on your indent rules.
gg=G auto indents the whole buffer.

Line folding:
:zM folds all lines.
:zm folds current stanza.
:zR unfolds all lines.
:zr unfolds current stanza.

Creating Splits:
:sb horizontally split buffer.
:vsplit virtically split buffer.

CTRL-w s split horizontally
CTRL-w v split vertically
CTRL-w c close current window
CTRL-w o make current window the only window

Switching Splits:
CTRL-w unfocus current split. Then use arrow key or "j"/"k" to move to alternate split.
CTRL-w _ unfocus split and maximize it vertically.
CTRL-w - shrink all vertical splits by 1 line giving more room to prompt.
CTRL-w > to increase split by 1 column.
CTRL-w < to decrease split by 1 column.
CTRL-w + to increase split by 1 line.
CTRL-w - to decrease split by 1 line.
10 CTRL-w + to increase split by 10 lines.
10 CTRL-w - to decrease split by 10 lines.
CTRL-w CTRL-w goto next split

:CTRL-^ switch between buffers in current split.

Opening Buffers:
:e [filename] Open a new buffer with the specified target file.
:open [filename] Opens filename in current split.
:open [directory] Opens netrw directory listing at specified directory
:Explore Opens netrw directory listing in current directory.
P *(when in netrw view) open file in last split

Moving between Buffers:
:ls shows all your open buffers.
:buffer [(partial)filename] goto buffer with specified filename open.
:b [number] goto buffer with number [number]</pre>
