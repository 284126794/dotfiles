# tree
~/dotfiles/
└── nvim/
    └── .config/
        └── nvim/
            ├── init.vim
            └── lua/
                └── (various Lua config files)

# usage
# goto the dotfiles,list all the applications there,which all contain the  <name1>/.config/<name2>/...
# where name1 used in the stow,name2 used becasue the ~/.config/<name2> is the config route.
cd ~/dotfiles
# -v means details; -t for setting the location,here now is ~;nvim is the folder name.

stow -v -t ~ nvim

# actually is :
~/.config/nvim/init.vim -> ~/dotfiles/nvim/.config/nvim/init.vim。
