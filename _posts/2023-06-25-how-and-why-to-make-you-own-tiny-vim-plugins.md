---
layout: post
date: 2023-06-25 12:47 -0700
---

This post is for anyone who is using vim/nvim and wants to make the leap into some light scripting to improve their editing experience.
It is easy to get started on configuring the appearance and behavior of vim with some tweaks to the `.vimrc` but the next step of creating plugins can look intimidating; but it doesn't need to be.
Open source Vim plugins deal with loading their code into the runtime, script-local namespacing, and handling all expected use-cases and edge-cases.
But scripting in Vim can be as simple as a single short file and with that you can solve your own personal annoyances and focus on the use-cases that matter to you and be prudent about addressing edge-cases.

## Example: copy using/includes to the left window

<script async id="asciicast-593182" src="https://asciinema.org/a/593182.js"></script>

When coding in C++ I often found myself repeating a previously done operation and then being immediately greeted with dozens of `W`s from my LSP.
I would then open a file where I had done this similar operation already and then one by one resolve the `undefined symbol` and `missing include` errors by copying over includes and usings.
I found myself doing this over and over pressing the same window-movement and copy-paste keys,  so I made the following vimscript to automatically copy them from the right hand window (where v-splits are opened for me) to the left hand window.

[copyovercpp.vim](https://github.com/swanyriver/BASH/blob/master/vim/copyovercpp.vim)

```vimscript
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
  * source the function and keymapping in your `.vimrc`

That's it, bish bash bosh, you have your own tiny vim plugin.

Note about the keymap: the `:<c-u>` before and the `<cr>` after the function
call are there to open the command prompt or clear any existing input, and then
to simulate the Enter key to run the given command.

## State Save In, State Restore Out

The vimscript above has a filo/stack order for saving a state before modifying and then restoring that state.
In doing so the cursor position is unchanged in each window after the operation and the unnamed register (referred to as @@, the most recently yanked contents) is undisturbed.
The `getpos` and `setpos` functions are used for capturing and restoring the cursor position.

The above function can be described thusly:
```text
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

There are often two ways to do something in a vimscript:
  * using the key shortcuts you are used to via the `normal` command
    * `normal yy` would copy a given line
  * using a named command
    * in the script above `wincmd <arg>` instead of `normal <C-w> <arg>`

The named commands can be easier to read and to understand the arguments being
passed to them, or find then help document for.  But when writing your own
plugins you can feel free to use whatever mix of them you prefer.

## Iterations

If you were making a plugin to share with the world you would need to reach a
reasonable 1.0 before releasing and then make appropriate dot releases.  But
when making your own tiny plugins you can start using it right away as soon as
it almost works the way you want.  And then make small changes as needed, such
as [this one](https://github.com/swanyriver/BASH/commit/27722e5b6d5de5fc34a6ebdf77cec243cd7ff885#diff-9c721aeab9a456f3b990a1e4182566017e5458753322d14f9135576882e2a2bb) that fixed a small bug and changed it to work when more than 2 windows were open.

