---
title: Practical usage of nvim-tree in Neovim
description: null
date: 2024-12-27T02:07:50.000Z
image: null
math: null
license: null
hidden: false
comments: true
draft: false
tags:
  - nvim-tree
  - neovim plugin
categories:
  - Linux
  - Neovim
---

Here is a summary of commonly used commands. There are still many features that haven't been convered yet.
I will continue to add them as I use them in the future.

# Basic Features

| Shortcuts | Function                                          |
| --------- | ------------------------------------------------- |
| a         | Create File Or Directory                          |
| d         | Delete                                            |
| c         | Copy                                              |
| x         | Cut                                               |
| p         | Paste                                             |
| o         | Open File in current window Or Expand a Directory |
| q         | Close                                             |
| r         | Rename                                            |
| e         | Rename: Basename                                  |
| y         | Copy Filename                                     |
| Y         | Copy Relative Path                                |
| gy        | Copy Absolute Path                                |

# Quick navigation in the file tree

| Shortcuts | Function                                                                       |
| --------- | ------------------------------------------------------------------------------ |
| <CTRL-]>  | Change current directory                                                       |
| BackSpace | Close floder of current file                                                   |
| -         | Set the working directory to the parent directory of current working directory |
| P         | Move to the parent directory of current file                                   |
| >         | Move to next sibling                                                           |
| <         | Move to previous sibling                                                       |
| J         | Move to the last sibling                                                       |
| K         | Move to the first sibling                                                      |
| E         | Expand All                                                                     |
| W         | Collapse All                                                                   |

# Buffer and Tab management

| Shortcuts | Function                                        |
| --------- | ----------------------------------------------- |
| Ctrl-T    | Open file in new tab                            |
| Ctrl-V    | Open file in current window by vertival split   |
| Ctrl-X    | Open file in current window by horizontal split |
| Tab       | Open preview                                    |

# Git integration

I'm not very familiar with Git-related features currently.

# Other useful key mappings

| Shortcuts | Function                                                         |
| --------- | ---------------------------------------------------------------- |
| B         | Toggle Filter: Display only files currently loaded in the buffer |
| Ctrl-K    | Show information of the file                                     |
| g?        | Help                                                             |
| R         | Refresh                                                          |
