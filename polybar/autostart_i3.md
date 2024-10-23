# add to the config of i3(maybe ~/.config/i3/config)
exec_always --no-startup-id $HOME/.config/polybar/launch.sh

# and remove these to disable the i3bar:
bar {
    i3bar_command i3bar
}
