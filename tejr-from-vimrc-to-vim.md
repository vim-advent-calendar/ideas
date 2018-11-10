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
choices of options, plugins, and mappings are bad and wrong. Not in this
article, anyway. Instead, I want to show how you can leverage the logic of
Vim's `'runtimepath'`, startup sequence, and filetype-switching logic to place
your configuration in the *right place*, so it loads at the *right time*. I'm
hoping to help the Vim community along on its slow realization that we should
be sharing our `.vim` directories, and not single giant `.vimrc` files.

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
once defined, they can just sit there, ready for use. Instead of putting the
function definition in `~/.vimrc`, drop it into `~/.vim/plugin`, as
`~/.vim/plugin/strip_trailing_whitespace.vim`.

Why `~/.vim/plugin`? Our code isn't a "real" plugin, after all, it's just a
single function. However, to Vim, that's not a meaningful distinction; per
`:help load-plugins`, it loads all the `*.vim` files it can in the `plugins`
subdirectory of each of the directories in `'runtimepath'`. It makes no
difference to Vim what those files actually are.

The files are also only loaded during Vim's actual startup, not when your vimrc
is sourced. If you have a mapping to reload your vimrc, putting heavier blocks
of code in `~/.vim/plugin` can avoid re-running them unnecessarily.

It doesn't have to be functions, either. A set of related abbreviations, or
custom commands, or a block of code dependent on a single operating system? Why
not?

Putting blocks of code like this in distinct files in `~/.vim/plugin` has
some other advantages. For one thing, you can use Vim's [script-variable][sv]
*scoping* for internal functions and variables:

    let s:myvar = 'value'
    function s:Myfunc()
      ...
    endfunction

This means you don't have to worry about trampling on any other variables
defined in your vimrc file.

Another advantage is the ability to short-circuit the script; you can check at
the top of the file whether the rest of the file should be loaded at all, and
use [`:finish`][fn] if it shouldn't; this is the basis of a lot of guards that
check for the `'compatible'` option, a minimum version of Vim, the availability
of a feature, or whether the plugin has already been loaded:

    if &compatible
          \ || v:version < 700
          \ || has('folding')
          \ || exists('g:loaded_myplugin')
      finish
    endif
    let g:loaded_myplugin = 1

Should you include the mapping that uses the defined function in the plugin
itself? Good question; it's up to you, but I like to think of vimrc as where
the user-level choices go, and mappings would fall in that category. If you
want to be thorough, and to keep some abstraction between what the plugin does
and how it's called, you can use [`<Plug>` prefix][pp] mappings to expose an
interface from the plugin file:

    function s:StripTrailingWhitespace()
      ...
    endfunction
    nnoremap <Plug>StripTrailingWhitespace
          \ :<C-O>:call <SID>StripTrailingWhitespace()<CR>

And then put your choice of mapping to it in your .vimrc:

    nmap \x <Plug>StripTrailingWhitespace

If someone else wants to use your plugin, this makes choosing their own
mappings for it a little more straightforward if they don't agree with yours.
There's more general advice about writing fully-fledged plugin files in [:help
write-plugin`][wp].

Not really my `filetype`
------------------------

Here's another common pattern often seen in vimrc files; this line of code is
intended to set the [`'spell'`][sp] option on for buffers of the [`mail` filetype][mf]:

    autocmd Filetype mail setlocal spell

The first thing to note here is that this should be surrounded in a
self-clearing [`augroup`][ag], so that re-running it doesn't leave multiple
definitions for the same hook:

    augroup ftmail
      autocmd!
      autocmd mail setlocal spell
    augroup END

The second is that this hook is set up every time vimrc is loaded, regardless
of whether a mail file is being edited. It makes more sense to put this into a
[filetype plugin][fp].

A file placed in `~/.vim/ftplugin/mail.vim` will be detected and run whenever a
new buffer's filetype is set to `mail`. This is set up by the long
`filetype.vim` file in the Vim runtime when activated by `filetype plugin on`.
The hooks do a lot of work to detect the type of any given file, and to run the
appropriate filetype plugins, so there's no need for the `autocmd` hooks here;
we already have hooks loaded, and a place to put the definition, so we can just
put this there:

    setlocal spell

To be a little bit neater still, we'll change a couple more things.

Firstly, rather than `~/.vim/ftplugin`, we'll put it in
`~/.vim/after/ftplugin`. The [after directory][ad] is run *after* the runtime
files included in Vim, so with this path, we can ensure that our option is set
*after* the main mail filetype plugin at `$VIMRUNTIME/ftplugin/mail.vim` has
completed whatever it does. This makes it more straightforward to *override*
what a filetype plugin configures if you disagree with it.

Secondly, if the filetype of the buffer changes, ideally we want to *reverse*
all the local filetype settings. We do this with the [`b:undo_ftplugin`][uf]
variable, which is run first whenever the filetype changes. After each option
we change, we should add code to this variable to *undo* what we set:

    setlocal spell
    let b:undo_ftplugin .= '|setlocal spell<'

The `spell&gt;` syntax used here specifies that the local value of `'spell'`
should be returned to match its global value at the time the filetype is
unloaded.

Don't forget that code related to indentation, such as `autoindent` or
`indentexpr` settings, goes in a different location again:
`~/.vim/after/indent`. Those are called if you include the word `indent` in
your `:filetype` call. You can put indent settings in your ftplugin if you want
to, but remember here we're trying to find the *right place* for things.

And Hell follow'd `after`
-------------------------

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
[pp]: http://vimhelp.appspot.com/usr_41.txt.html#using-%3CPlug%3E
[sv]: http://vimhelp.appspot.com/eval.txt.html#script-variable
[fn]: http://vimhelp.appspot.com/repeat.txt.html#%3Afinish
[wp]: http://vimhelp.appspot.com/usr_41.txt.html#write-plugin
[mf]: http://vimhelp.appspot.com/syntax.txt.html#mail%2Evim
[ag]: http://vimhelp.appspot.com/autocmd.txt.html#%3Aaugroup
[fp]: http://vimhelp.appspot.com/usr_05.txt.html#ftplugins
[ad]: http://vimhelp.appspot.com/options.txt.html#after-directory
[uf]: http://vimhelp.appspot.com/usr_41.txt.html#undo_ftplugin
