From `.vimrc` to `.vim`
=======================

> You can’t have everything. Where would you put it?
> –Steven Wright

Attack of the 5,000-line vimrc
------------------------------

Vim is a tremendously configurable and extensible editor, with a culture of
enthusiastic users eagerly sharing and trying out configuration for their
startup configuration files in [`~/.vimrc` or `~/.vim/vimrc`][rc]. That means
that these configuration files can end up getting pretty large over time, as
users add more and more to them over the years. Users learning how the editor
works and then customizing it to their workflow move from setting a few global
defaults for options—like [`'expandtab'`][et] and [`'wrap'`][wr]—to custom
mappings, functions, filetype-specific logic, and nearly inevitably a litany of
third-party plugins, under the ever-shifting mantle of plugin managers.
Configurations get progressively larger, more intricate, and finely-tuned.

If you’re an intermediate to experienced user that has one of these longer
files, you’re in good company. Damian Conway, a Vim guru specializing in Perl,
published [a vimrc with 1,855 lines][dc], and at the time of writing, [Steve
Losh’s is a whopping 3,160 lines][sl]. I’m sure you can find vimrc files that
are even longer—perhaps yours already is.

The issue with very long vimrc files isn’t so much that the amount of
configuration is in itself excessive—after all, all of that power is there for
a reason. If you’ve been programming for a while, you know from experience that
it’s good practice to avoid very large files that do a lot of disparate things.
Since Vim configuration is just code, it’s no exception to this rule. Instead
of a single mammoth configuration file, there’s a strong case to be made for
having a larger set of relatively smaller files that each have a role or
responsibility—and those files go in `~/.vim`.

Applying a directory hierarchy to replace your mammoth vimrc will not only help
to keep your configuration manageable, it may also help with keeping Vim
running snappily and smoothly by loading code and configuration only when you
actually need it. It’ll also be clearer from its position in the directory
hierarchy what the purpose of any given configuration is, both for you and for
anyone reading your configuration. Finally, this process makes it much easier
to *package* any useful part of your configuration for others to use.

Your own personal `$VIMRUNTIME`
-------------------------------

Most of the benefit of a `~/.vim` configuration directory centers around “going
with the grain” of the way Vim already does things, affording you a lot of
control over in what order and under what circumstances specific configuration
is loaded.

Let’s start by looking at the structure of the runtime files that come with Vim
itself. You can find the path for this directory in the `$VIMRUNTIME` variable:

    :echo $VIMRUNTIME

If you’re using the version of Vim that was packaged with your operating
system, it will very likely be something like `/usr/share/vim/vim80`.

Let’s take a look at the contents of that directory:

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

Of those, 1,291 are `.vim` files:

    $ find /usr/share/vim/vim80 -type f -name \*.vim | wc -l
    1291

All of these are just plain Vim script files, like your vimrc, to be loaded
under some set of conditions, depending on location within this directory. Only
a small fraction of them will actually be run on Vim startup. That’s over a
thousand files, almost all of them ready to be loaded *only when relevant*. We
should take a hint from Bram on that!

The directory named in `$VIMRUNTIME` is only the first part of the story. If we
look at the value of the `'runtimepath'` option in Vim, we can see mentions of
a few other paths named:

    :set runtimepath?
      runtimepath=~/.vim,/usr/share/vim/vim80,...

The very first entry of `'runtimepath'` is `~/.vim`, and that’s where we’re
going to build a structure mimicking that of `$VIMRUNTIME`—your personal Vim
runtime directory.

Lose the `:source`, Luke
---------------------------

If you’ve worked with Vim script for a while, you probably know how to use the
[`:source`][sc] command to read in script from a file. You might even have
some lines like this in your vimrc, to load a separate file with something like
mapping definitions in it:

    source ~/.vim/mappings.vim

Vim offers another method for loading these files that works with the file
layout of the `'runtimepath'` we just inspected, named [`:runtime`][rc]. Used
without an exclamation mark, it reads Vim script commands from the first path
it finds in any of its `'runtimepath'` entries. With an exclamation mark, it
reads *all* of them, including pattern matching with `?` and `*` globs.

    runtime syntax/c.vim
    runtime! syntax/c.vim
    runtime! */maps.vim
    runtime! **/maps.vim

The [double asterisk][ss] in the last example here can be taken as a shorthand
for a directory hierarchy, which can be up to 100 levels deep. This means that
a file in `foo/bar/baz/quux/maps.vim` would still be found and loaded.

Unlike `:source`, `:runtime` also doesn’t raise errors if it can’t find the
file, avoiding a lot of boilerplate if a file’s absence in a particular
directory may be normal.

A few of Vim’s startup processes—and some of its commands, like
[`:filetype`][ft]—are in fact just thin wrappers around `:runtime` commands.
Leveraging this allows a lot of flexibility in choosing what files to run and
where, based on what’s available in these paths; with a little care, we can run
code before, instead of, or after other code, in order to disable, replace, or
extend it.

Turn on, `plugin`, drop out
---------------------------

You can start the process of breaking up the code in your vimrc file by looking
for blocks of self-contained code that are veering away from simple
configuration of the editor’s existing functions, and more into dedicated
programs of their own, so we can put them into self-contained file in the
`plugin` subdirectory.

As a common example, if you’ve been reading others’ vimrc files, you will have
doubtless seen many attempts to solve the universal problem of quickly removing
spaces at the end of lines—or “trailing whitespace”—from code. This function to
do so is from the [Vim Tips wiki][vs]:

    function StripTrailingWhitespace()
      if !&binary && &filetype != 'diff'
        normal mz
        normal Hmy
        %s/\s\+$//e
        normal 'yz<CR>
        normal `z
      endif
    endfunction

It’s usually followed by a mapping of some sort to actually call the function:

    nnoremap <Leader>x :<C-U>call StripTrailingWhitespace()<CR>

Functions like the one above don’t need to be loaded every time vimrc is
sourced. Once defined, they can just sit there, ready for invocation when
appropriate. In fact, for the function above, Vim throws an error if we attempt
to reload our vimrc, because it doesn’t like the re-definition. We could fix
that by declaring the function with `function!`, but there’s another way:
instead of putting the function definition in `~/.vimrc`, drop it into
`~/.vim/plugin` with a `.vim` extension, for example as
`~/.vim/plugin/strip_trailing_whitespace.vim`.

Once installed and Vim is restarted, we can confirm our plugin was loaded by
checking it’s included in the output of [`:scriptnames`][sn]:

    :scriptnames
    ...
    10: ~/.vim/plugin/strip_trailing_whitespace.vim
    ...

Note that the `<Leader>x` mapping left in your vimrc still works just fine,
too, despite being loaded *before* the function was defined.

### What’s a plugin, anyway?

Why `~/.vim/plugin`? You might object that our code isn’t a “real” plugin; it’s
just a single function. To Vim, that’s not a meaningful distinction—as
described in [`:help load-plugins`][lp], it loads all the `*.vim` files it can
in the `plugins` subdirectory of each of the directories in `'runtimepath'`. It
makes no difference to Vim what Vim script those files actually contain.

A plugin file doesn’t have to be functions, either. A set of related
abbreviations? Custom commands? Code dependent on one particular machine or
operating system? Hey, why not?

### Plugin subdirectories

Similarly, because `*.vim` files are loaded from the `plugin` directory
recursively, you can organize them in subdirectories if you want to:

    ~/.vim/plugin/insert/cancel.vim
    ~/.vim/plugin/insert/suspend.vim
    ~/.vim/plugin/visual/region.vim
    ~/.vim/plugin/whitespace/squeeze.vim
    ~/.vim/plugin/whitespace/trim.vim

Remember how we mentioned Vim’s thinly-veiled `:runtime` wrappers? This is one
of them. The clue here to how this works is in the command that `:help
load-plugins` suggests as an analogue to what it does internally:

    :runtime! plugin/**/*.vim

### Local script scope

Putting blocks of code like this in distinct files in `~/.vim/plugin` has
some other advantages. For one thing, you can use Vim’s [script-variable][sv]
*scoping* for internal functions and variables:

    let s:myvar = 'value'
    function s:Myfunc()
      ...
    endfunction

This apples a unique numeric prefix to all of your function names and variable
names, which means you don’t have to worry about trampling on any other
variables defined in your vimrc file; the code is better scoped and
self-contained.

### Short-circuiting and load guards

Another advantage of separate plugin files is the ability to **short-circuit**
any given script—to prevent it from loading if it’s not appropriate to do so.
You can check at the top of the file whether the rest of the file should be
loaded at all, and use [`:finish`][fn] if it shouldn’t.

For Vim plugin distributions, this is the basis of a lot of guards that check
for the `'compatible'` option, or a minimum version of Vim, or the availability
of a feature, or whether the plugin has already been loaded:

    if &compatible
          \ || v:version < 700
          \ || has('folding')
          \ || exists('g:loaded_myplugin')
      finish
    endif
    let g:loaded_myplugin = 1

Now you don’t have to wrap the whole thing in a single `:if` block in your
vimrc.

### The question of mappings

Should you include a mapping that uses a defined function in the plugin itself?
Good question; it’s up to you, and your plans for the plugin, but I like to
think of vimrc as where the user-level choices go, and plugins where the code
they call should go, and mappings would fall into the former category.

If you want to be thorough, and to keep some abstraction between what the
plugin does and how it’s called, you can use [`<Plug>` prefix][pp] mappings to
expose an interface from the plugin file:

    function s:StripTrailingWhitespace()
      ...
    endfunction
    nnoremap <Plug>StripTrailingWhitespace
          \ :<C-U>call <SID>StripTrailingWhitespace()<CR>

And then put your choice of mapping to it in your vimrc:

    nmap <Leader>x <Plug>StripTrailingWhitespace

If someone else wants to use your plugin, this makes choosing their own
mappings for it a little more straightforward if they don’t agree with yours.
There’s more general advice about good mapping practices in writing
fully-fledged plugin files in [`:help write-plugin`][wp].

Not really my `filetype`
------------------------

Here’s another common pattern often seen in vimrc files; this line of code is
intended to set the [`'spell'`][sp] option on, but only for buffers of the
[`mail` filetype][mf], to highlight possible spelling errors as an email
message is edited:

    autocmd Filetype mail setlocal spell

The first thing to note here is that this should be surrounded in a
self-clearing [`augroup`][ag], so that re-running it doesn’t leave multiple
definitions for the same hook:

    augroup ftmail
      autocmd!
      autocmd mail setlocal spell
    augroup END

However, we can avoid this boilerplate. The second thing to note is that this
hook is set up every time vimrc is loaded, *regardless* of whether a mail file
is actually edited in that session. It makes more sense to put this into a
[filetype plugin][fp] or **ftplugin**.

A file placed in `~/.vim/ftplugin/mail.vim` will be detected and run whenever a
new buffer’s filetype is set to `mail`. The hooks that do this are set up by
the long `filetype.vim` file in your `$VIMRUNTIME` Vim runtime when activated
by `filetype plugin on`. They do a lot of work to detect the type of any given
file, and to run the appropriate filetype plugins, so there’s no need for the
`autocmd` hooks here; we already have hooks loaded, and a place to put the
definition, so we can just put this single line in there:

    setlocal spell

Now, when we edit a new `mail` buffer, we can check `:scriptnames` and confirm
that our file was loaded when the filetype was chosen:

    :set filetype=mail
    :scriptnames
    ...
    20: ~/.vim/ftplugin/mail.vim
    ...
    :set spell?
      spell

We can improve this still further.

### Loading filetype code afterwards

Rather than putting our option setting in `~/.vim/ftplugin/mail.vim`, we’ll put
it in `~/.vim/after/ftplugin/mail.vim`—note the extra `after` level. The [after
directory][ad] is run *after* the runtime files included in Vim, so with this
path, we can ensure that our option is set *after* any main mail filetype
plugin at `$VIMRUNTIME/ftplugin/mail.vim` has completed whatever it does. This
makes it more straightforward to *override* what a filetype plugin configures
if you happen to disagree with it.

If you want to make this even more granular, you can also put files in
*subdirectories* named after the filetype:

    ~/.vim/after/ftplugin/mail/spell.vim
    ~/.vim/after/ftplugin/mail/quote.vim

The filetype followed by an underscore and then a script name works, too:

    ~/.vim/after/ftplugin/mail_spell.vim
    ~/.vim/after/ftplugin/mail_quote.vim

This is another `:runtime` analogue on Vim’s part; it’s like we ran this:

    :runtime! ftplugin/mail.vim ftplugin/mail_*.vim ftplugin/mail/*.vim

### Undoing filetype settings

If the filetype of the buffer changes, ideally we want to *reverse* all the
local filetype settings we made. We can do this with the
[`b:undo_ftplugin`][uf] variable, which is run first whenever the filetype
changes. After each option we change, we should add code to this variable to
*undo* what we set if it needs to be reversed:

    setlocal spell
    let b:undo_ftplugin .= '|setlocal spell<'

The `spell<` syntax used here specifies that the local value of `'spell'`
should be restored to match the global value of `'spell'`, at the time the
filetype is unloaded.

We can check its value with `:let`:

    :set ft=mail
    :let b:undo_ftplugin
    b:undo_ftplugin        setl modeline< tw< fo< comments<|setlocal spell<

### The difference with indent

Don’t forget that code related to indentation, such as `autoindent` or
`indentexpr` settings, goes in a different location again: `~/.vim/indent` or
`~/.vim/after/indent`. Those files are called if you include the word `indent`
in your `:filetype` call.

You can put indent settings in your filetype plugin if you want to—Vim won’t
protest, or even notice—but remember that ideally here we’re trying to find the
*right place* for things.

### Detecting filetypes

As a final note for filetype-dependent logic, don’t forget that hooks to set a
buffer’s filetype have their own subdirectory too, in `ftdetect`:

    autocmd BufNewFile,BufRead */irc/*.log setfiletype irssilog

Putting the hooks in the `ftdetect` directory means they are loaded as part of
the `filetypedetect` `augroup` defined in `filetype.vim`. This is therefore one
place in which you do *not* have to surround `autocmd` definitions in a
self-clearing `augroup`—because if you put the definitions in the right place,
it’s already done for you.

Be water, my friend
-------------------

Note that all of the above is just the beginning: we haven’t even touched on
[lazy-loading functions][al] for speed with definitions in `~/.vim/autoload`,
or custom [`:compiler`][cm] definitions for setting `'makeprg'` and
`'errorformat'` in `~/.vim/compiler`: yet more examples of Vim functionality
that wraps around `:runtime` loading.

While Vim does afford the user tremendous power in configuring and customizing,
there is definitely a **Way of Vim** for the timely loading of relevant
configuration, and if you learn a little about how it works, you’ll fight with
your editor that much less. When you first learned Vim, do you remember how
strange using the HJKL keys for movement seemed, before it made sense? Do you
remember how you wanted to stay in insert mode all the time, before normal mode
made sense?

Working within the Vim runtime file structure instead of ignoring it, fighting
with it, or trying to reshape it can make your `~/.vim` directory into a
refined toolbox, with a place for everything, and everything in its place. It’s
well worth the effort.

If you’d like to see an example of how this layout can end up looking when you
make it work for you, [the author’s personal `~/.vim` directory is available
for download][pv].

[ad]: http://vimhelp.appspot.com/options.txt.html#after-directory
[ag]: http://vimhelp.appspot.com/autocmd.txt.html#%3Aaugroup
[al]: http://vimhelp.appspot.com/eval.txt.html#autoload
[cm]: http://vimhelp.appspot.com/quickfix.txt.html#%3Acompiler
[dc]: https://github.com/thoughtstream/Damian-Conway-s-Vim-Setup/blob/cbe1fb5b5505e17bd7709669168c367903d94cd4/.vimrc
[et]: http://vimhelp.appspot.com/options.txt.html#%27expandtab%27
[fn]: http://vimhelp.appspot.com/repeat.txt.html#%3Afinish
[fp]: http://vimhelp.appspot.com/usr_05.txt.html#ftplugins
[ft]: http://vimhelp.appspot.com/filetype.txt.html#%3Afiletype
[lp]: http://vimhelp.appspot.com/starting.txt.html#load-plugins
[mf]: http://vimhelp.appspot.com/syntax.txt.html#mail%2Evim
[pp]: http://vimhelp.appspot.com/usr_41.txt.html#using-%3CPlug%3E
[pv]: tejr-from-vimrc-to-vim.zip
[rc]: http://vimhelp.appspot.com/usr_05.txt.html#05.1
[rt]: http://vimhelp.appspot.com/repeat.txt.html#%3Aruntime
[sc]: http://vimhelp.appspot.com/repeat.txt.html#%3Asource
[sl]: https://bitbucket.org/sjl/dotfiles/src/e2a961f1d037e53ea2809885a65feba66a9aa03e/vim/vimrc?at=default&fileviewer=file-view-default
[sn]: http://vimhelp.appspot.com/repeat.txt.html#%3Ascriptnames
[sp]: http://vimhelp.appspot.com/options.txt.html#%27spell%27
[ss]: http://vimhelp.appspot.com/editing.txt.html#starstar-wildcard
[st]: http://vim.wikia.com/wiki/Remove_unwanted_spaces
[sv]: http://vimhelp.appspot.com/eval.txt.html#script-variable
[uf]: http://vimhelp.appspot.com/usr_41.txt.html#undo_ftplugin
[wp]: http://vimhelp.appspot.com/usr_41.txt.html#write-plugin
[wr]: http://vimhelp.appspot.com/options.txt.html#%27wrap%27
[vs]: http://vim.wikia.com/wiki/Remove_unwanted_spaces
