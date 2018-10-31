Make your setup truly cross-platform
===============================================================================
Once you have taken the time to setup Vim exactly the way you want, you might
still encounter configuration issues. My whole `~/.vim` folder is under version
control so I can just clone it on any computer I want my familiar Vim
experience.

But I use Windows computers and Linux computers. Even within the same
environment, I might have different dependencies installed for the plugins I
try to import. Also, from time to time I like to use Neovim too, to try out a
few unique features.

All the tips in this article gravitate around being able to use if statements in
your startup scripts.


Environment specific settings
-------------------------------------------------------------------------------
To make environment specific settings we need to know the environment we are
running on.
We can ask vim directly if it was compiled for Windows, otherwise a system call
to `uname` will give the running environment. The function below returns a string
containing the value of the running environment.

```vim
function! whichEnv() abort
    if has('win64') || has('win32') || has('win16')
        return 'WINDOWS'
    else
       return toupper(substitute(system('uname'), '\n', '', ''))
    endif
endfunction

"""""""""""""""""""""""""""""
" Later use in another file "
"""""""""""""""""""""""""""""
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

Intermission : external shell calls optimizations
-------------------------------------------------------------------------------
External shell calls are costly, mostly because of the context switching they
require. They are not very long, but one of the big advantages of vim is the
very quick startup time, and having too many external calls in your startup
will definitely have consequences on your experience.

To illustrate this, let's compare the startup time of Vim with a one-liner
vimrc, where :
- one case has `let g:dummy_string = 'dummy string'` (no external shell call),
- the other case has `let g:dummy_string = system('date')` (one external shell
  call)

The command I used to compare the startup times was `vim --noplugin -u
one_liner_vimrc --startuptime startup.txt`, and here is the final result :
- no external call : `002.369  000.003: --- VIM STARTED ---`
- external call : `093.497  000.002: --- VIM STARTED ---`

The 90 ms difference in startup time is entirely due to the external system call
(see `:h startuptime` for help on the syntax) :
```
# No external call
001.246  000.021  000.021: sourcing no_externalcall_vimrc

# External call
092.540  088.869  088.869: sourcing externalcall_vimrc
```

Caching the results from any external system call as
much as possible is important when crafting flexible `.vim/` startup scripts.
Here is the final function I use (and call in my other scripts) to know my
environment. The result of `system` is stored in a global variable and used
in all scripts.

```vim
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

"""""""""""""""""""""""""""""
" Later use in another file "
"""""""""""""""""""""""""""""

" I can call this function before every environment specific block with the
" early return branch.
call myHelpers#setEnv()
if (g:Env =~# 'WINDOWS')
    " Enable Windows specific settings/plugins
else if (g:Env =~# 'LINUX')
    " Enable Linux specific settings/plugins
else if (g:Env =~# 'DARWIN')
    " Enable MacOS specific settings/plugins
else
    " Other cases I can't think of like MINGW
endif
```

> For romainl : vim has "has(osx)" and even "has(osx_darwin)", why not
> use those ?

also,
> Should I speak about autoload or just leave it as an advanced feature ?
> Basically the solution boils down to autoloading or leaking the function
> in the global namespace.
Autoloading here doesn't change
anything about startup time : the file will be loaded on each startup because the
init scripts will call the function. It is just done to namespace my helper
functions.

Host specific settings
-------------------------------------------------------------------------------
We can use exactly the same method for host specific settings.

I only use this
method for Linux, since I have a lot more things installed on my laptop than
there is on the cluster, where my editing doesn't need half the plugins I use
for personal use.

Linux provides [hostname](https://linux.die.net/man/1/hostname) so we can use
the same function as before, replacing only the `toupper...` line with
`system('hostname')`, and storing it in another `g:` variable like `g:Hostname`.

_*This method may also work on Windows, but not sure*_

Host specific settings are good when you know you're only cloning your `~/.vim`
directory in a few computers on which you know what is installed. Using this to
differentiate between 50 hosts means you will need very long if statements which
get quickly hard to read.

I still find this useful for clipboard
handling or other purely host-specific settings.

> I made this a big heading because it felt like a good thing to keep in the
> TOC of the article, but it's really short in content (it boils down to
> s/uname/hostname/g && s/g:Env/g:Hostname/g). Should I fuse ?

Dependencies specific settings
-------------------------------------------------------------------------------
Even on the same environment, dependencies might not be fulfilled on all the
target machines. These dependencies can be separated into 2 categories, Vim's
*features* and external *dependencies*. The difference I make between these 2
is that *features* are defined at compilation time in Vim and that
*dependencies* are external to Vim's compilation.

Vim keeps track of its own feature set, defined at compile time. Therefore you
can directly use vimscript to know if Vim has a feature or not, using the
`has()` function.
See `:h has()` for all the features you can test for
directly within vim.
```vim
if has('cscope')
" Enable all the plugins or change settings to use cscope support
endif
```

For external dependencies (like the linters you might want to set as `makeprg`
or the LSP servers you want to start for a project), using `executable()`
instead of using a call to [which](https://linux.die.net/man/1/which) is very
important, as it is way faster than `system`. I ran the same test case as
before with a vimrc which only include an `executable()` call :
```vim
if executable('rg')
    let g:string_date = 'dummy string'
    " usually the line here is set grepprg=rg\ --vimgrep
endif
```

and the results are almost the same as the *no external call* case :
```
# Important lines only
001.991  000.136  000.136: sourcing exec_vimrc
003.383  000.005: --- VIM STARTED ---
```


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
one software, but duplicating changes is a code smell.

My solution is to use
`runtime` heavily and externalize all the common parts of my old `vimrc`.
`runtime` will look for files to source with the given name in your
`runtimepath` and will source the first one it finds. Choosing a "unique"
folder for the sourced files (like `settings`) allows to store them all with
any name as long as they are in the `runtimepath`. I choose to leave them
directly in the `~/.vim` folder to have them under version control of course :
```
$ ls ~/.vim
... some files
init.vim
vimrc
settings/
... other folders

$ ls ~/.vim/settings
colors.vim
ale.vim
... other files

```

```vim
" In vimrc
set autoindent " 'vim-specific' setting
runtime settings/fold_fillchars.vim

```


`if has('nvim')` is exactly what you want to separate the 2 cases in your
scripts. For example, to load plugins only for Vim or only for Neovim, you can
put your optional plugins in `~/.vim/pack/vim_or_neovim/opt` and then use this
kind of snippets :

```vim
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
TODO list. And also because writing this post made me verify and learn new
things about how to further smooth my truly cross platform setup.


Gerry

This article is licensed under
[Creative Commons v4.0](https://creativecommons.org/licenses/by/4.0/)
