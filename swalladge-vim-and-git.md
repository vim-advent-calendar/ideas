# Vim ∪ Git


Vim and Git are both highly complex, configurable developer tools. Developers
who use Vim are likely to also need to use Git frequently. This article attempts
to explore how these two tools can interact in many ways.

First off, I'm not going to prescribe any particular workflows, or argue for or
against a particular method. There are just too many options and you are
encouraged to develop your own workflow.

Vim and Git are two separate tools and can of course be used that way. However,
it can be useful, time-saving, and convenient to integrate the two.


## The Git perspective

From the perspective of Git there are several opportunities to utilize Vim:

- editing commit/tag messages
- resolving merge conflicts (yay!)
- interactive rebasing

### Editing commit messages

If you have the `EDITOR` environment variable already set to Vim, Git should
automatically use Vim to edit messages. If environment variables aren't to be
relied on, the `core.editor` git config can be set:

```
[core]
    editor = "vim"
```

When editing a commit message in Vim and wish to abort, you can use the `:cq`
command to exit with a non-zero status. This will cause Git to abort the commit,
even if you had begun writing a message.

Additionally, be sure to use `git commit -v` or set `commit.verbose = true` in
your gitconfig file to have the full patch shown in Vim when editing the commit
message.


### Resolving merge conflicts

Before resorting to external diff tools to merge, consider using Vim's excellent
diff mode to help! Vim can be configured to be used as Git's `mergetool`, so
it can be automatically launched with the correct configuration and files ready
to perform the merge when you run `git mergetool`. There are two main methods
which I've used: vanilla vim (or neovim) in diffmode, and vim-fugitive's
`Gdiff`. Example configuration below (my setup, neovim):

```
[merge]
    tool = nvimdiff4
    # if not using a tool name with builtin support, must supply mergetool cmd
    # as below

[mergetool "nvimdiff4"]
    cmd = nvim -d $LOCAL $BASE $REMOTE $MERGED -c '$wincmd w' -c 'wincmd J'

[mergetool "nfugitive"]
    cmd = nvim -f -c "Gdiff" "$MERGED"
```

The first method requires no plugins. It is simply Vim (Neovim) launched in
diffmode with the 4 files Git provides in environment variables. The wincmds
executed move the working file to full-width along the lower half of the window.

The second method uses the vim-fugitive plugin to set up the layout. It still
uses Vim's diffmode, but only shows 3 files (local, remote, and index). I think
it provides some other niceties too (TODO: I've forgotten what because i usually
use vanilla).

To help with choosing which buffer to `do` or `dp` from/to, I like to include
the buffer number in the status line. The item `%n` will do that. See `:h 'statusline'` for more.


Likewise, git `difftool` can be set to use Vim too:

```
[diff]
    tool = nvimdiff2

[difftool "nvimdiff2"]
    cmd = nvim -d $LOCAL $REMOTE
```


### Interactive rebasing

TODO


## The Vim perspective

Ok, so the flip side: how can Git be integrated into Vim?

Note: I personally use Neovim, but practically everything will work the same in
Vim. Assume "Vim" refers to both Neovim and Vim unless a distinction is
explicitly made.

### Vanilla

Let's begin with useful things you can do with no plugins required.

#### `:!git`

One way is to shell out directly to Git to run a command. Be aware that
interactive stdin is impossible currently in Neovim so any commands that prompt
for input will fail.

```
:!git stash
```

#### `:terminal`

Neovim, and more recently Vim, have embedded terminals that can be launched with
the `:terminal` command. This enables having a complete shell (and Git command
line tools) within Vim. Very useful if using Gvim, or don't want to suspend/quit
Vim to get back to a shell.

####

TODO others


### Configuration

`autoread` and `checktime` are useful if you leave Vim open while checking out
different commits and don't want to deal with Vim complaining that a file has
changed on disk.

```
set autoread
set checktime
```



### Plugins


#### Vim-Fugitive

**The** Git plugin. There are many articles about Fugitive so I won't go into it
in much detail. Basically, it provides many Vim commands for working with the
repo, a diffing setup for staging changes, interactive status window, etc. I
recommend the 


#### GitGutter/Signify

[GitGutter][gitgutter] and [Signify][signify] are both plugins whose job is to
show diff information in the sign column. GitGutter is tailored for Git, and
provides some useful extras, such as undoing hunks. Signify has historically
been faster, but recent refactoring efforts in GitGutter has levelled the
playing field. I'd recommend GitGutter for its extra features if you only use
Git, or Signify if you also use other VCSs.


#### GV

[GV][gv] is an excellent Git browser for Vim. TODO: more info


#### Vimagit

[Vimagit][vimagit] is an attempt to bring some of Emacs' famous [Magit][magit]
to Vim.  It provides a single buffer where you can launch Git commands, stage
hunks/files, and commit.  Its interface is very nice (imho), but unfortunately
it suffers from low performance. Worth checking out but is perhaps not mature
enough to support efficient workflows yet.


#### Committia

[Committia][committia] Committia is a simple plugin to help make editing commit
messages smoother.

This is separate to other plugins like fugitive. If you are already in vim, This
is separate to other plugins like fugitive. If you are already in vim, then
fugitive will give a more advanced then fugitive will give a more advanced
commit workflow. However, committia is useful when you are working on the
command line and just want to commit with `git commit ...`. In this case, your
configured editor will launch with the `COMMIT_EDITMSG` file to edit the commit
message. If you `git commit -v ...`, then you already get the diff view, but
committia adds some niceties:
TODO: clean up this section


### Tig

[Tig][tig] is an awesome cli Git browser. [tig-explorer][tig-explorer] is a neat
plugin that opens Tig in a Vim terminal buffer.

TODO more explanation


### Branch manager

I recently discovered some cool plugins for managing Git branches (and related
tasks). [Twiggy][twiggy] and [Merginal][merginal]




## Github specific goodies

Github is one of the most popular Git repository hosting services, and it
follows that there are some helpful Vim plugins for further Git repository
integration if hosted on Github.

[rhubarb][rhubarb] is a Vim plugin by tpope that enables opening a Git object on
Github in the browser. It's very convenient when you are working in Vim on a
local repository and wish to refer someone to a particular line or file.

To open the current file:

```
:Gbrowse
```

To link to a range of lines, supply a range for the function. For example, visual
select some lines and run:

```
:'<,'>Gbrowse
```

Finally, any git object can be given as an argument to the function:

```
:Gbrowse a75830f
```

Rhubarb also contains an autocompleter for Github issues when editing commit/tag
messages.



## My workflow

So what about my workflow, you may ask. Well I cnsider my workflow to be very
scattered. I use whichever method of performing a Git action as happesn to be
most convenient at the time. Generally I use Tmux and have a pane/window open
that I can switch to and run Git commands. I find GitGutter great in vim to see
which lines have been changed and not staged. I do have a lot of Git plugins
installed, and attempting to transition to using them more often. When I say I
use the most convenient command, sometimes the most convenient is a command that
is slower but less taxing on the memory. For example, often I will open a new
tmux window and  use `git add --patch` from the cli rather than Fugitive's
`:Gdiff` simply because I'm more familiar with the former.

Something I can't stress enough is the importance of learning a tool and forcing
yourself to use that tool everywhere possible until it reaches muscle memory if
it's to actually be an improvement to your workflow. I've found that it's no
point having convenient plugins installed if I end up forgetting about them and
using the vanilla methods instead.


## Conclusion

TODO

---

Contact details:

- name: Samuel Walladge
- email: samuel@swalladge.id.au
- GitHub: swalladge
- Twitter: @srwalladge
- irc (freenode): swalladge
- website: swalladge.id.au


[committia]: https://github.com/rhysd/committia.vim
[gitgutter]: https://github.com/airblade/vim-gitgutter
[gv]: https://github.com/junegunn/gv.vim
[magit]: https://magit.vc/
[merginal]: https://github.com/idanarye/vim-merginal
[rhubarb]: https://github.com/tpop/vim-rhubarb
[signify]: https://github.com/mhinz/vim-signify
[tig-exploror]: https://github.com/iberianpig/tig-explorer.vim
[tig]: https://github.com/jonas/tig
[twiggy]: https://github.com/sodapopcan/vim-twiggy
[vimagit]: https://github.com/jreybert/vimagit
