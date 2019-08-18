---
title: VIM split ticks and tips
date: 2019-08-18 15:29:51
tags:
---

VIM is useful and powerful for the generic editor, it has lots of plugins to use for different language and purpose. The vim config file located in `$HOME/.vimrc`. The configuration is not that easy to make as your plugins increasing and its quite tedious to remember and search "How To Use XXX" for each plugs. But https://vim-bootstrap.com/ provide a  handy way to ease the configuration, which saved lots of time.

For me, the split feature is useful, which can display multiple file in a singe window, you don't need to switch from files.

<!--- More--->

The generic tricks for the vim split manipulation

```
 :e filename      - edit another file
 :split filename  - split window and load another file
 ctrl-w up arrow  - move cursor up a window
 ctrl-w ctrl-w    - move cursor to another window (cycle)
 ctrl-w_          - maximize current window
 ctrl-w=          - make all equal size
 10 ctrl-w+       - increase window size by 10 lines
 :vsplit file     - vertical split
 :sview file      - same as split, but readonly
 :hide            - close current window
 :only            - keep only this window open
 :ls              - show current buffers
 :b 2             - open buffer #2 in this window
```

Once generated the `.virmc` file by using the [vim-bootstrap](https://vim-bootstrap.com), the shortcut would be created and you can even create your own shortcut for spcific command.

Assuming we opened file and split that into 3 splits


![vim-split-tutorila](/images/vim-split-tutorial.png)

### Split Navigations

You can use different key mappings for easy navigation between splits to save a keystroke. So instead of ctrl-w then j, it’s just ctrl-j:

```
nnoremap <C-J> <C-W><C-J>
nnoremap <C-K> <C-W><C-K>
nnoremap <C-L> <C-W><C-L>
nnoremap <C-H> <C-W><C-H>
```

### Split Resizing

Vim’s defaults are useful for changing split shapes:
```SHELL
"Max out the height of the current split
ctrl + w _

"Max out the width of the current split
ctrl + w |

"Normalize all split sizes, which is very handy when resizing terminal
ctrl + w =
```

### Split Manipulation

```SHELL
"Swap top/bottom or left/right split
Ctrl+W R

"Break out current window into a new tabview
Ctrl+W T

"Close every window in the current tabview but the current one
Ctrl+W o
```

That's it.
