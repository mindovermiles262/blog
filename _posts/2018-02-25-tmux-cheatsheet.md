---
title: tmux Cheat Sheet
date: 2018-02-25
layout: post
tags: automation tmux linux
---

## A TMUX Cheat Sheet
### (Because I can never remember the commands)

### Session Management 
`tmux ls` (or `tmux list-sessions`) - List sessions 
`tmux new -s session-name` - Create new session
`Ctrl-b d` - Detach from session 
`tmux attach -t [session name]` - Attach to session
`tmux kill-session -t session-name` - Kill session

`Ctrl-b c` - Create new window 
`Ctrl-b d` - Detach current client 
`Ctrl-b l` - Move to previously selected window 
`Ctrl-b n` - Move to the next window 
`Ctrl-b p` - Move to the previous window 
`Ctrl-b &` - Kill the current window 
`Ctrl-b ,` - Rename the current window 
`Ctrl-b q` - Show pane numbers (used to switch between panes) 
`Ctrl-b o` - Switch to the next pane 
`Ctrl-b ?` - List all keybindings 

### Moving Between Windows
`Ctrl-b n` - Move to the next window
`Ctrl-b p` - Move to the previous window
`Ctrl-b l` - Move to the previously selected window
`Ctrl-b w` - List all windows / window numbers
`Ctrl-b window number` - Move to the specified window number (the default bindings are from 0 -- 9) 

# Tiling Commands 
`Ctrl-b %` - Split the window vertically
`Ctrl-b "` - Split window horizontally
`Ctrl-b o` - Goto next pane
`Ctrl-b q` - Show pane numbers
`Ctrl-b {` - Move the current pane left 
`Ctrl-b }` - Move the current pane right
