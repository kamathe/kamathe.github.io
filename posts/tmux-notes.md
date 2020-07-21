# Tmux notes
### `21 July 2020`


## Some random tmux related notes


- [Start a new tmux session](#start-a-new-tmux-session)
- [Split screens](#split-screens)
- [Switch between panes](#switch-between-panes)
- [Detach session](#detach-session)
- [List attach session](#list-attach-session)
- [Scroll within session](#scroll-within-session)
- [Kill session](#kill-session)
- [Run script within tmux in detached mode](#run-script-within-tmux-in-detached-mode)



## Start a new tmux session


```
# without any name

tmux

# with a name

tmux new -s "name"

# rename an old session

tmux rename-session -t 0 "newname"

# rename current window

Ctrl-b + \,
```


## Split screens

```
# side by side

Ctrl-b + %

# up and down

Ctrl-b + \"

```


## Switch between panes


```
# move to next pane

Ctrl-b + arrow-key

# switch between full screen pane and normal

Ctrl-b + z

# resize custom pane

Ctrl-b C arrow-key
```


## Detach session


```
# detach from session

Ctrl-b + d
```


## List attach session

```
# list sessions

tmux ls

# re-attach single and only session

tmux a or tmux attach


# re-attach specific session

tmux attach -t 0

```


## Scroll within session

```

Ctrl-b PageUp
Ctrl-b [ - then UP arrow

# scroll with mouse

Ctrl-b [

```


## Kill session

```

# kill specific session

tmux kill-session -t 0


# kill all sessions

tmux kill-server

# kill all sessions except one we are currently in

tmux kill-session -a


# set scroll limit via conf file .tmux.conf

set-option -g history-limit 3000

# or on command line

tmux set-option history-limit 5000

```



## Run script within tmux in detached mode


```

# run a command or script within tmux without attaching to it
# good for scripting


tmux new-session -d -s my_session 'bash test.sh'
```
