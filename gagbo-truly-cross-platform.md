Make your setup truly cross-platform
===============================================================================
Once you have taken the time to setup Vim exactly the way you want, you might
still encounter configuration issues. My whole `~/.vim` folder is under version
control so I can just clone it on any computer I want my familiar Vim
experience.

But I use Windows computers and Linux computers. Even within the
same environment, I might have different dependencies installed for the plugins I try to
import. Also, from time to time I like to use Neovim too, to try out a few
unique features.



environment specific settings
-------------------------------------------------------------------------------
To make environment specific settings we need to know the environment we are
running on.
We can ask vim directly if it was compiled for Windows, otherwise a system call
to `uname` will give the running environment. The function below returns a string
containing the value of the running environment.

```vimscript
function! whichEnv() abort
    if has('win64') || has('win32') || has('win16')
        return 'WINDOWS'
    else
       return toupper(substitute(system('uname'), '\n', '', ''))
    endif
endfunction

" Later use in another file
if (whichEnv() =~# 'WINDOWS')
    " Enable Windows specific settings/plugins
else if (whichEnv() =~# 'LINUX')
    " Enable Linux specific settings/plugins
else if (whichEnv() =~# 'DARWIN')
    " Enable MacOS specific settings/plugins
else
    " Other cases I can't think of like MINGW
endif
```

Intermission : system calls optimizations
-------------------------------------------------------------------------------
System calls are costly
> Make a MWE with or without loading no-stl-wednesdays or any system call in init
> (see the difference in startuptime with or without the
> no-stl-wednesdays plugin which is only a call to `date`)
; caching the results as
much as possible yields good results. Autoloading probably doesn't change
anything about startup time : the file will be loaded on each startup because the
init scripts will call the function. It is just done to namespace my helpers
functions.

```vimscript
" Sets only once the value of g:env to the running environment
" from romainl
" https://gist.github.com/romainl/4df4cde3498fada91032858d7af213c2
function! myHelpers#setEnv() abort
    if exists('g:env')
        return
    endif
    if has('win64') || has('win32') || has('win16')
        let g:env = 'WINDOWS'
    else
       let g:env = toupper(substitute(system('uname'), '\n', '', ''))
    endif
endfunction
```

Host specific settings
-------------------------------------------------------------------------------
We can use exactly the same method for host specific settings.

I only use this
method for Linux, since I have a lot more things installed on my laptop than
there is on the cluster, where my editing doesn't need half the plugins I use
for personal use.

Dependencies specific settings
-------------------------------------------------------------------------------
Even on the same environment, dependencies might not be fulfilled on all the target
machines.

Vim keeps track of its own feature set and you can always use `has()` to check
for features in your scripts. See `:h has()` for all the features you can test
directly within vim.
```vimscript
if has('balloon_eval')
" Do balloon stuff here
endif
```

For external dependencies (like the linters you might want to set as `makeprg`
or the LSP servers you want to start for a project), using system calls is
again the solution to make it work.
> I know about which for Unix, but I actually have not tested that on Windows


Bonus round : Vim 8+ and Neovim compatibility
-------------------------------------------------------------------------------
Vim 8+ is important because I make heavy usage of the package feature for this
adaptation.

First step is to symlink the folders of course. We only want one copy of the
`.vim` folder on the system, Vim does not care about `init.vim` and Neovim does
not care about `vimrc`

After a few updates I made in my plugins and/or colorschemes, I noticed I
always had 2 files to change : `init.vim` and `vimrc`. I still want to keep the
files different because there are a few settings which are actually specific to
one software, but duplicating changes is a code smell. My solution is to use
`runtime` heavily and externalize all the common parts of my old `vimrc`.
> Maybe I should put those files under plugin now, but it's still more a startup
> file than an actual plugin sooo....

`if has('nvim')` is exactly what you want to separate the 2 cases in your
scripts. For example, to load plugins only for Vim or only for Neovim, you can
put your optional plugins in `~/.vim/pack/vim_or_neovim/opt` and then use this
kind of snippets :

```vimscript
if has('nvim')
    " Load Neovim specific plugins
    packadd LanguageClient-neovim " This plugin is not Neovim specific anymore,
                                  " just here for the example
else
    " Load Vim specific plugins
    packadd traces.vim
endif
```


Results
-------------------------------------------------------------------------------
You can see a few of those principles applied on my [current
repo](https://framagit.org/gagbo/vim-setup). Be warned that it is still a little
bit messy, because tidying all the files and plugins is very low priority on my
TODO list.
