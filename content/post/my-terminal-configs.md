---
title: "My Terminal Configs"
date: 2017-12-17T14:15:50-07:00
draft: false
tags: ["environment", "configs"]
summary: "Here's some information about my system config files."
---
It's starting to become more popular for some hiring companies to request your
"dot" files to show how you configure your environment.
I figure this is probably intended to be something akin to a signalling mechanism,
meaning that you're into the weeds enough to know that on \*NIX systems you have 
configuration files for each user on the system, typically prefixed with a "dot".
Hence we have files like `.bashrc` that gets executed when you open a new bash
session and sets up the environment for you.
Same for vim, where you have the `.vimrc` file.
You can potentially have many more (including "dot" folders, like `.ssh`), just
depending on what software you use on a regular basis on your \*NIX system.

So I want to show and explain a few of my "dot" files here, but first, I have a gripe.

The point of having systems that, to whatever extent they may, conform to POSIX
standards is to facilitate interoperability.
Not only of systems, but users also.
That's why things like `vi`, `nano`, and `gnu-utils` are pretty uniformly auto-installed
on \*NIX systems.
That makes it easy for me, Sue the Admin, or Joe the Developer to log in to a
system they never have and at least know the tools available to them.
For that reason, I'm not a huge fan of doing a lot of customization of my "dot" files
and I try to encourage people to know the defaults quite well prior to customizing.

End of gripe.

{{% toc %}}

# Justification
So I just finished up my gripe about not using the standard tools and I'm about to start
talking about `vim` and `tmux`, neither of which fall into that "ubiquitously installed"
category.
Well, there's a reason for that.

I like `vi` well enough and can definitely use it without any troubles.
However, `vim` gives me some added functionality that is really useful for doing
development work, particularly `syntastic` for syntax checking/linting, `EDITORCONFIG`,
and some additional keybind functionalities that are just missing from `vi`.

Then there's `tmux`.
Again, `screen` is perfectly good and I can use it well enough, but there were just
some things that didn't work the way I was hoping.
For instance, `screen` requires a lot more combination keybinds to do things that
I would hope would be much simpler, such as splitting panes.
Also, `tmux`'s configurability is quite a bit stronger.

# .bashrc
My `.bashrc` is fairly plain-jane.
The only things that I've added are to put a few things in `$PATH` that are not
enabled by default, like snaps (I'm on BunsenLabs Helium/Debian Stretch).
I don't have any special aliases for things that I use a lot because I value
that muscle memory of being able to quickly use the standard built-in tools.

So, my `.bashrc` is the standard `.bashrc` that you get when creating a new user
on a Debian system, with the following two lines added at the end.

```
export PATH=$PATH:/snap/bin
export EDITOR='vim'
```

# .vimrc
My `.vimrc` has a bit more customization for things like `EDITORCONFIG`, `syntastic`,
and the `solarized` color scheme.
Still no special aliases for doing special things.
You'll notice nothing about `EDITORCONFIG` here, but it's being brought in by `pathogen`.

```
set nocompatible
syntax on
set autoindent		" always set autoindenting on
set smartindent
set nowrap		" turns off text wrapping
set number 		" turns line numbering on
colorscheme solarized
set background=dark
set ruler		" show the cursor position all the time

set smarttab
set softtabstop=2
set expandtab
set shiftwidth=2
set tabstop=2
filetype indent plugin on
set showcmd
set hlsearch
set ignorecase
set smartcase
set ruler
set laststatus=2
set confirm

set cmdheight=2

" Disable python-mode in favor of syntastic
let g:pymode_lint = 0
let g:pymode_lint_on_write = 0

" Syntastic setup
let g:syntastic_aggregate_errors = 1
let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_auto_jump = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0
let g:syntastic_javascript_checkers = ['eslint']
let g:syntastic_python_checkers = ['pep8', 'python']
let g:syntastic_tex_checkers = ['lacheck']

let Vimplate="/usr/bin/vimplate"

" Pathogen config
execute pathogen#infect()
```

# .tmux.conf
Again, not doing much special with tmux.
I've considered setting up my usual pane layout to automatically be used when I
start a new `tmux` session, but I just haven't gotten around to it.
Most of these are the defaults.
The only ones I've changed are to have a 256 color terminal (because baller),
change the pane index to be 1-based instead of 0-based because it makes it easier
to switch panes one-handed, letting tmux now that I use vim, and making the pane
numbers display a bit longer on screen to give me a second to decide which pane
I want to switch to.

My standard setup is 3-panes, with my main pane vertical on the right, and the other
two panes split horizontally on the left.
I like this better since it puts the vertical center line between the panes near the
middle of my screen, and therefore the left edge of my text editor is pretty close
to the middle of the screen as well.
`tmux` actually has a layout preset that's pretty similar to this called `main-vertical`,
however it puts the large "main" vertical pane on the left instead of the right.
Putting my main pane closer to the middle of the screen is definitely a bit more
natural since that's where my attention will naturally be drawn.

```
set -g default-terminal "tmux-256color"
set -g pane-base-index 1
set-window-option -g mode-keys vi
set -g display-panes-time 3000
set -g display-time 3000
set -g status-fg cyan # ThG original was: white
set -g status-bg default
set -g status-attr default
set -g status-left ""
if '[ -z "$DISPLAY" ]' 'set -g status-left "[#[fg=green] #H #[default]]"'
if '[ -z "$DISPLAY" ]' 'set -g status-right "[ #[fg=magenta]#(cat /proc/loadavg | cut -d \" \" -f 1,2,3)#[default] ][ #[fg=cyan,bright]%a %Y-%m-%d %H:%M #[default]]"'
if '[ -z "$DISPLAY" ]' 'set -g status-right-length 50'
```
