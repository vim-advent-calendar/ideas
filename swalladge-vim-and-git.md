# Vim and Git


- intro
- relationship between vim and git
- this no emacs/magit


## Workflows


### Vanilla


`:h autoread` for automatically updating buffers when files changed by git.

`<c-z>` to suspend, git in shell, `$ fg` back to editing

`!git <args>` from inside vim

newer vim/nvim have `:terminal`



### Tmux

- popular
- switch between tmux panes/windows; one for vim, another for shell/git
- no suspend/fg required


### Plugins

- plugins can augment the above
- plugins can replace the above


## Let's talk about plugins


### Fugitive

**The** git plugin.


### Committia

for editing commit messages. more about this later.

### signify/gitgutter

diff status in the gutter


### :GV

tig-style browser in vim


### tig-explorer

newer plugin for integrating tig with vim

### vimagit

attempt to bring some magit style to vim

- slow?


### others?

add more lesser-known niche git plugins (there are several)



## Config


vim config things here?


## Git config

We can't discuss using vim with git without covering how to work with git's
configuration to help integrate it better!

- `$HOME/.gitconfig`
- global and project-local
- mergetool setup for vim and nvim
- vim-fugitive with mergetool (should this be here or under fugitive section? so
  many cross-cutting concerns I'm not sure how to structure the article...)
- diff - from git's side


## Neovim vs Vim

- differences that affect working with git
- no interactive stdin with `:!command` in neovim
- less differences now both have async and :terminal



## Tips and tricks (aka I don't know where to put these)

### Editing commit messages

`:cq` in vim to exit with a non-zero status. This will abort the commit even if
you had begun writing a message. If the editor is exited normally without
writing any message, the commit will be aborted anyway.


#### [committia.vim](https://github.com/rhysd/committia.vim)

Committia is a simple plugin to help make editing commit messages smoother.
This is separate to other plugins like fugitive. If you are already in vim, then fugitive will give a more advanced
commit workflow. However, committia is useful when you are working on the
command line and just want to commit with `git commit ...`. In this case, your
configured editor will launch with the `COMMIT_EDITMSG` file to edit the commit
message. If you `git commit -v ...`, then you already get the diff view, but
committia adds some niceties:

- shows diff and status sections in separate buffers, split so everything is
  visible.
- `<Plug>` mappings for scrolling the diff buffer from the edit buffer.
- hook functions for opening windows event.

The workflow can then be along these lines:

1. Run `git commit` from the shell.
2. Vim opens and committia sorts out the buffers.
3. Begin typing the commit message.
4. Press your keybinds to scroll the diff to view more to help draft the commit
   message.
5. Close vim to finalize the commit.


### Tig and friends?

???

### github-specific

- autocompletion for github issues
- hub
- `:Gbrowse` with fugitive and rhubarb

