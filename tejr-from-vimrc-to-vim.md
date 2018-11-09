From `.vimrc` to `.vim`
=======================

> You can't have everything. Where would you put it?
> –Steven Wright

Attack of the 5,000-line `.vimrc`
---------------------------------

Vim is a tremendously customizable and extensible editor, with a culture of
enthusiastic users eagerly sharing and trying out configuration for their
startup configuration files at [`~/.vimrc` or `~/.vim/vimrc`][rc]. That means
that these configuration files can end up getting pretty large over time as
users add more and more to them over years of use. Users learning how the editor
works and then customizing it to their workflow move from setting a few global
defaults for options—like [`'expandtab'`][et] and [`'wrap'`][wr]—to custom
mappings, functions, filetype-specific logic, and nearly inevitably a litany of
third-party plugins, under the ever-shifting mantle of plugin managers.
Configurations get progressively larger, more intricate, and finely-tuned.

If you're an intermediate to experienced user that has one of these longer
files, you're in good company. Damian Conway, a Vim guru specialising in Perl,
published [a .vimrc with 1,855 lines][dc], and at the time of writing, [Steve
Losh's is a whopping 3,160 lines][sl]. I'm sure you can find vimrc files that
are even longer—perhaps yours already is.

The issue with very long vimrc files isn't so much that the amount of
configuration is in itself excessive—after all, that customizability is there
for a reason. However, if you've been programming for a while, you know from
experience that it's good practice to avoid very large files that do a lot of
disparate things. Since Vim configuration is just code, it's not an exception
to this rule. Instead of a single mammoth configuration file, there's a strong
case to be made for having a larger set of relatively smaller files that each
have a role or responsibility—and those files go in `~/.vim`.

Don't worry, this article isn't going to try to push any Vim minimalism on you.
I'm not going to tell you to junk 90% of your configuration, or that your
choices of options, plugins, and mappings are bad and wrong. Instead, I want to
show how you can leverage the logic of Vim's `'runtimepath'`, startup sequence,
and filetype-switching logic to place your configuration in the *right place*,
so it loads at the *right time*. I'm hoping to help the Vim community along on
its slow realization that we should be sharing our `.vim` directories, and not
single giant `.vimrc` files.

We'll learn how to do this by going through lots of specific examples of the
sort of configuration often found in `.vimrc` files, and show how to place them
in `~/.vim` in a sensible way.

Applying a directory hierarchy to replace your mammoth `.vimrc` will not only
help to keep your configuration manageable, it may also help with keeping Vim
running snappily and smoothly by loading code and configuration only when you
actually need it. It'll also be clearer from its position in the directory
hierarchy what the purpose of any given configuration is, both for you and for
anyone reading your configuration. Finally, this process makes it much easier
to *package* any useful part of your configuration for others to use.

Your own personal `$VIMRUNTIME`
-------------------------------

Most of the benefit of a `~/.vim` configuration directory centers around "going
with the grain" of the way Vim already does things, affording you a lot of
control over in what order and under what circumstances specific configuration
is loaded.

For the uninitiated, let's start by looking at the structure of the runtime
files that come with Vim itself. You can find the location of this directory in
the `$VIMRUNTIME` variable:

    :echo $VIMRUNTIME

If you're using the version of Vim that was packaged with your operating
system, it will very likely be something like `/usr/share/vim/vim80`. Let's
take a look at the contents of that directory:

    $ ls /usr/share/vim/vim80
    autoload/     bugreport.vim      colors/             compiler/
    debian.vim    defaults.vim       delmenu.vim         doc/
    evim.vim      filetype.vim       ftoff.vim           ftplugin/
    ftplugin.vim  ftplugof.vim       gvimrc_example.vim  indent/
    indent.vim    indoff.vim         keymap/             lang/
    macros/       menu.vim           mswin.vim           optwin.vim
    pack/         plugin/            print/              rgb.txt
    scripts.vim   spell/             synmenu.vim         syntax/
    tutor/        vimrc_example.vim

In this directory hierarchy, using a quick-and-dirty count, there are 1,600
files:

    $ find /usr/share/vim/vim80 -type f | wc -l
    1600

Of those, 1,291 are just .vim files:

    $ find /usr/share/vim/vim80 -type f -name \*.vim | wc -l

All of these files are just plain Vim script files, like your vimrc, to be
loaded under some set of conditions, depending on location. Only a small
fraction of them will actually be run on Vim startup. Over a thousand files,
almost all of them ready to be loaded *only when relevant*.

However, the directory named in `$VIMRUNTIME` is only part of the story. If we
look at the value of the `'runtimepath'` option in Vim, we can see mentions of
a few other paths named:

    :set runtimepath?
      runtimepath=~/.vim,/usr/share/vim/vim80,...

The very first entry of `'runtimepath'` is `~/.vim`, and that's where we're
going to build a structure mimicking that of `$VIMRUNTIME`—your personal Vim
runtime directory.

Turn on, `plugin`, drop out
---------------------------

Let's start the process by looking for blocks of self-contained code in your
vimrc that are veering away from simple configuration of the editor's existing
functions, and more into dedicated programs of their own.

If you've been reading others' vimrc files, you will have doubtless seen many
attempts to solve the universal problem of quickly removing spaces at the end
of lines—or "trailing whitespace"—from code. This example is from the Vim Tips
wiki, defining a function for the task:

    function StripTrailingWhitespace()
      if !&binary && &filetype != 'diff'
        normal mz
        normal Hmy
        %s/\s\+$//e
        normal 'yz<CR>
        normal `z
      endif
    endfunction

It's usually followed by a mapping of some sort to actually call the function:

    nnoremap \x :<C-U>call StripTrailingWhitespace()<CR>

Functions like the above don't need to be loaded every time vimrc is sourced:
once defined, they can just sit there, ready for use. Instead of putting this
in `~/.vimrc`, drop it into `~/.vim/plugin`, as
`~/.vim/plugin/strip_trailing_whitespace.vim`.

Not really my `filetype`
------------------------

Don't `:call` us, we'll `:call` you
-----------------------------------

Lose the `:source`, Luke
------------------------

Be water, my friend
-------------------

[dc]: https://github.com/thoughtstream/Damian-Conway-s-Vim-Setup/blob/cbe1fb5b5505e17bd7709669168c367903d94cd4/.vimrc
[sl]: https://bitbucket.org/sjl/dotfiles/src/e2a961f1d037e53ea2809885a65feba66a9aa03e/vim/vimrc?at=default&fileviewer=file-view-default
[rc]: http://vimhelp.appspot.com/usr_05.txt.html#05.1
[et]: http://vimhelp.appspot.com/options.txt.html#%27expandtab%27
[wr]: http://vimhelp.appspot.com/options.txt.html#%27wrap%27
