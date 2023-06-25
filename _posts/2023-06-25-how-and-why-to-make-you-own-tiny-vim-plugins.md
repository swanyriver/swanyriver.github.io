---
layout: post
title: how_and_why_to_make_you_own_tiny_vim_plugins
date: 2023-06-25 12:47 -0700
---

This post is for anyone who is using vim/nvim and wants to make the leap into some light scripting to improve their editing experience.
It is easy to get started on configuring the appearance and behavior of vim with some tweaks to the `.vimrc` but the next step of creating plugins can look intimidating; but it doesn't need to be.
Open source available Vim plugins deal with loading their code into the runtime, script-local namespacing, and handling all expected use-cases and edge-cases.
But scripting in Vim can be as simple as a single short file and with that you can solve your own personal annoyances and focus on the use-cases that matter to you and be prudent about addressing edge-cases.

## Example: copy using/includes to left-buffer

When coding in C++ I often found myself repeating a previously done operation and then being immediately greeted with dozens of `W`s from my LSP.
I would then open a file where I had done this similar operation already and then one by one resolve the `undefined symbol` and `missing include` errors by copying over includes and usings.
I found myself doing this over and over pressing the same window-movement and copy-paste keys,  so I made the following vimscript to automatically copy them from the right hand window (where v-splits are opened for me) to the left hand window.

https://github.com/swanyriver/BASH/blob/master/vim/copyovercpp.vim

```
function CopyOverCpp()
  let l:saved_unnamed_register = @@
  let copy_from_save_cursor = getcurpos()
  normal ^Y
  wincmd h
  let copy_to_save_cursor = getcurpos()

  if @@ =~ "\^using "
    normal G
    let l:foundline = search("^using ", "b")
    call append(l:foundline, @@)
    write
  elseif @@ =~ "\^#include "
    normal G
    let l:foundline = search("^#include ", "b")
    call append(l:foundline, @@)
    write
  else
    echom "not a using or include"
  endif

  call setpos('.', copy_to_save_cursor)
  normal j
  wincmd l
  call setpos('.', copy_from_save_cursor)
  let @@ = l:saved_unnamed_register
endfunction

nnoremap <space>c :<c-u>call CopyOverCpp()<cr>
```

In `.vimrc` load above function and remap with `source copyovercpp.vim`

## Function, a Keymap, and Source:  SO EASY!!

As stated above creating your own editor actions can be as simple as
  * define a `function` 
  * map that function to a key in normal-mode with `nnoremap`
    *  `:<c-u>` opens the command-prompt or clears any existing input
    *  `call ***()` invokes your function
    *  `<cr>` simulates the Enter press to run the mapped command
  * source the function and keymapping in your `.vimrc`

That's it, bish bash bosh, you have your own tiny vim plugin.

## State Save In, State Restore Out

The vimscript above has a filo/stack order for saving a state before modifying and then restoring that state.
In doing so the cursor position is unchanged in each window after the operation is done and the unnamed register (referred to as @@, the most recently yanked contents) is undisturbed.
The getpos and setpos functions are used for capturing and restoring the cursor position.

The above function can be described thusly:
```
save yank-register
save R-window cursor
* yank *
move to L-window
save L-window cursor

* find and paste *

restore L-window cursor
move to R-window
restore R-window cursor
restore yank-register
```

Following this pattern of **Save Before Modify** and **Restore in Reverse Order** is an easy way to perform complex actions without causing side-effects.

## Plumbing vs Porcelain

## Iterations

