# Mappings

[NOTE]

Late af, sorry about that.

Just pushing to repo to show progress, there is still a lot to fix:
- typos
- *much* better titles
- add all links to vim documentation

Also, the article might be spit into two parts, depending on what is left ot
add.

Won't be able to work on it this week-end, will try and finish it by next
Wednesday.

[/NOTE]

## On Mappings And Vim Design

What makes Vim so different from other editors is, arguably, its rich set of
motions commands. Once you can move around with precision, you not only gain
time, as you don't need to reach for the mouse, but you can also turn pretty
much every modification into 'programs', either to accomplish ad hoc tasks via
recorded macros, or to address more general cases, thanks to commands,
functions--and mappings.

A mapping is the binding of a key or a sequence of keys, called the *left-hand
side (LHS)*, to a series of keystrokes, the *right-hand side (RHS)*. The RHS is
similar to a macro recorded with `q`; contrary to some other tools or editors,
mappings don't reach for some 'core' functions or commands, like hypothetical
`next-word` or `delete-char`, that the default bindings would just call. In Vim,
motions and modification commands are so central that it would not make much
sense to decouple them from their standard keys, since they are the building
blocks of what we could call the "Vim editing programming language". For
instance, `dw` is like a tiny program that deletes chars from cursor to the
beginning of next word; it's a 'program' every vim user will understand, like an
'if' statement in C or in another programming language.

Obviously, since alphanumric keys are used to write 'editing programs', there is
a need to separate 'programming mode' where the user hit `dw` to delete a word,
from 'insertion mode', where the user just want to type text and not execute
programs. That's why Vim is a *modal editor*, contrary to, say, Emacs. Once you
go the modal road, it makes sense to define extra modes for things like visual
selection or the command line. Again, mappings make it possible to automate
tasks for each of these modes.

So, if Vim is programming, mappings are no less than its user-defined functions.

## A Whirlwind Tour Of Mappings

The general syntax of map-definining commands is:

    {map-cmd} [modifiers] {lhs} {rhs}

'{map-cmd}' is one of the several mapping-defining commands we will examine
shortly. '[modifiers]' is an optional list of modifiers. '{lhs}' is the key
sequence that will trigger the mapping: if it contains a literal blank (space,
tab or linefeed), it must be escaped. '{rhs}' is the key sequence that will be
triggered, as though the user typed them directly (for the most part -- there
are a few differences).

### Modes

Each Vim mode got its own map-defining commands:

* `nmap` for normal mode
* `imap` for insert mode
* `vmap` for visual mode and select mode
* `xmap` for visual mode only
* `cmap` for command-line mode
* `omap` for operator-pending mode
* `tmap` for terminal mode

Most are self-explanatory. `vmap` defines mappings both for visual mode and
select mode; if you don't know what select mode is, it's similar to visual mode,
but hitting a printable key like alphanumerics will replace the visual selection
with that key and switch to insert mode. Select mode is not often used, so many
vim users do as though it didn't exist and use `vmap` to define visual mode
mappings. Some plugins can take advantage of it though, for instance snippet
plugins that expand shortcuts to text or code with parts that the user may want
to change, like some variable name. Selecting the variable in select mode
enables the user to type the new name 'over' the visual selection, or to hit
some control char (eg. `<Tab>`) to go to the next occurrence. Unfortunately, if
the user made some mappings with `vmap`, this can interfere with that process,
if there are mappings starting with a printable char like `<Space>` or `,`.
Thus, it's best to use `xmap` to define mappings for visual mode (and `smap` for
select mode if you ever need it).

`omap` is for operator-pending mode, ie. the mode where extra key(s) are
expected, typically to define the selection where the current command will
operate. This is the mode you get when you hit `y`, `c` or `d` and Vim waits for
the user to define what will be yanked, changed or deleted. This command
enables the user to create custom text objects of sorts.

`tmap` lets users define mappings for the new *terminal mode* of Vim, availble
inside a buffer containing a terminal. This requires Vim 8 or the backported
patches.

Finally, `map` is the original mapping command from vi, and in Vim it
simultaneously defines a mapping for normal, visual and operator-pending mode.
`map!` (also from vi) defines a mapping for insert and command-line mode. The
mode-specific versions (`nmap`, `imap` etc.) are generally prefered in Vim.

To be exhaustive, there is also `lmap`, for locale-specific mappings, which we
won't address in this article.

### Noremap

Say you want to extend the `<Return>` key in normal mode, so that in addition to
its standard function (going to the first non-blank of the next line), it also
temporarily turns off search highlighting. You come up with the following
command:

    nmap <Return> :nohls<Return><Return>

It calls the `nohls` Ex command on the command line, runs it, and then finally
sends a `<Return>` to go to the next line. You execute this nmap command, hit
`<Return>`, and... Vim seems to hang (hit `<C-c>` to interrupt). What gives?

The `:nohls<Return>` part runs just fine, but the issue is the last `<Return>`:
remember, the RHS of a mapping runs as though the user typed it, like a macro.
So, when when you hit `<Return>` to run the mapping, the mapping will also "hit"
`<Return>` at the end--and you get an infinite recursion!

What you meant, of course, was to execute the core function of the unmapped
`<Return>`, not to run the mapping again. In other words, you don't want
mappings to apply in the RHS. That's exactly what the 'noremap' version of
map-defining commands do. Try:

    nnoremap <Return> :nohls<Return><Return>

Bingo, everything works as intended.

Each map-creating command has its noremap version:

* nnoremap
* inoremap
* xnoremap
* cnoremap
* onoremap
* etc.

As a rule of thumb, use the noremap versions unless you actually need to run
mappings in the RHS.

### Modifiers

A few modifiers are available to tweak the created mapping; they all have the
form `<modifier>`, between angle brackets. Here are the main ones; consult the
documentation for the whole list.

#### Silent

`<silent>` is one of the most used modifiers. As its name suggests, it turns
off visual feedback for the duration of the mapping execution. This is
especially useful if you run Ex commands in the RHS, and you don't want to
expose it to the user. For instance:

    xnoremap <silent> p p:if v:register == '"'<Bar>let @@=@0<Bar>endif<cr>

This extends `p` in visual mode (note the noremap), so that if you paste from
the unnamed (ie. default) register, that register content is not replaced by
the visual selection you just pasted over--which is the default behavior. This
enables the user to yank some text and paste it over several places in a row,
without specifying a named register (eg. `"ay`, `"ap` etc.).

In the previous mapping, it would be annoying to see the `:if v:register...`
command line each time you pasted in visual mode; thus, the `<silent>` modifier
was added, and nothing is displayed when the mapping run. As for the rest of the
mapping, the `v:register` special variable contains the name of the register, if
any, that was specified to the current mapping when the user types something
like `"ap`. The statement `let @@=@0` re-assigns the value of the `@0` register,
which contains the last yanked text, to the unnamed register (`@@`) once the
paste is done. `<Bar>` stands for the `|` character that separates Ex commands
on the command line, and `<cr>` is another way to get a carriage return, ie.
`<Enter>` or `<Return>`.

#### Buffer

`<buffer>` makes the new mapping buffer-local, ie. it will be defined only for
the buffer that was current when the mapping was created. It is useful in
filetype-specific settings, eg. in `/.vim/after/ftplugin/help.vim`:

    nnoremap <silent><buffer> zl
     \:call search('<Bar>[^ <Bar>]\+<Bar>\<Bar>''[A-Za-z0-9_-]\{2,}''')<cr>

A buffer-local mapping on `zl` is created on buffers with the `help` filetype,
ie. help buffers created with `:help`. The mapping jumps to the next tag in the
current buffer. Obviously, this only makes sense in help buffers, so we should
not make this mapping global. Again, note the `<silent>` modifier, sicne we
don't want to show this on the command line each type we use the mapping.

Here is the backward-jumping version of the mapping:

    nnoremap <silent><buffer> zh
     \:call search('<Bar>[^ <Bar>]\+<Bar>\<Bar>''[A-Za-z0-9_-]\{2,}''','b')<cr>

#### Unique

`<unique>` prevents the clobbering of an existing mapping on the same LHS. This
can be useful for plugin authors, who want to offer default mappings but are
still careful not to override users' own mappings:

    nnoremap <unique><silent> <LocalLeader>a :call ThisPluginFunction()<cr>

If there already is a mapping on `<LocalLeader>a`, then the `nnoremap` above
will fail (the error can be silenced with `silent! nnoremap <unique> ...`).
`<LocalLeader>` stands for some user-defined key, typically used for
buffer-local settings.

#### Expr

`<expr>` is a very useful modifier, perhaps a bit more difficult to grasp than
those above. It changes the meaning of the RHS: it is no longer a sequence of
keys to run like a macro, but an *expression* that will be evaluated each time
the mapping is triggered, and the result of that evaluation will be a string
containing the key sequence to run. So this is a level of indirection to make
things more dynamic; typically, it contains some conditional to either run one
sequence or the other. Here's an example:

    inoremap <expr> jk pumvisible() ? "<C-e>" : "<Esc>"

This is the "classic" `jk` to exit insert mode, with a twist: if a pop-up menu
is visible (this is the menu with completion candidates), then it will close it
with `<C-e>` instead of exiting insert mode with `<Esc>`. The whole RHS is a
single expression, here, a ternary conditional, and when you hit `jk`, the
expression is evaluated: the pumvisible() function is called, returning true or
false depending on whether a pop-up menu is visible, and if it is then the
expression evaluates to `<C-e>`, otherwise it evaluates to `<Esc>`. The result
of that evaluation becomes the final RHS. 

Here's another example:

    onoremap <expr> il ':<C-u>norm! `['.strpart(getregtype(), 0, 1).'`]<cr>'

This is an operator-pending mapping, that selects the last piece of changed text
(most often, some pasted text), in the same visual mode (char, line or block) as
that of the used register. It can be used to indent some pasted lines, with
`>il`. The `<expr>` modifier is necessary to evaluate the `strpart(...)` part of
the RHS; it is then concatenated to the leading and trailing literal strings, to
form the final RHS.

As for the rest of the mapping: the `<C-u>` clears the command-line, since in
some circumstances Vim can fill it automatically with a line range, for instance
if `:` is hit from visual mode, or if a count is given to it (either directly,
or from a mapping, here if we hit something like `3il`). `norm` will execute
normal mode commands, and the `!` says not to use mappings in those, like
`noremap`. The `[` and `]` marks are automatically set on both ends of the last
changed text, so the first backtick goes to one end, then the right visual mode
is set by the `strpart(...)` part, and the second backtick goes to the other end
of the changed text. Making a visual selection from an operator-pending map
defines the object of the current command, eg. `y`, `c` or `d`.

### Escaping and Notation

#### Bars

Mapping-defining commands are standard Ex commands, so they can be separated by
the `|` character. What do you think the following command will do?

    nnoremap <F7> :echo "foo"|echo "bar"

This command actually contains two parts: first, the `nnoremap` Ex command, then
the `echo "bar"` one. So, it will map `<F7>` to `:echo "foo"`, and it will run
`echo "bar"` just once, when the `<F7>` mapping is defined. This is probably not
what the user intended!

In order to map `<F7>` to `echo "foo"|echo "bar"`, the `|` character must be
escaped, either with `\|` or with the Vim Notation `<Bar>`. The following two
commands do the same thing:

    nnoremap <F7> :echo "foo"\|echo "bar"
    nnoremap <F7> :echo "foo"<Bar>echo "bar"

Pick your favorite method. Also, don't forget to add a trailing `<cr>` to those
mappings if you want to run the command lines when calling the mappings;
otherwise, they will stay there on the command line, for the user to modify, run
or cancel.

#### Control Chars

You can use literal control chars in your mappings, by hitting `<C-v>` followed
by the control char in insert or command-line mode, eg. pressing `<C-v>` then
the tab key will insert a literal tab char (you can use `:set list'` to show
them). However, it is not very convenient to work with literal control chars:
the graphic representation can be confusing (eg. "^[" for the Escape key), and
it can insert the terminal-dependent sequence of the control char, like "^[OP",
instead of the generic character (here, the F1 key).

The best practice is to use the Vim notation like we did in this article, eg.
`<C-x>`, in five chars (`<` + `C` + `-` + `x` + `>`), for Control+x. All keys
have a notation, eg. `<Tab>`, `<Return>` or `<Esc>`; check out the
documentation for the complete list.

Vim will expand Vim notation in mappings commands as long as the `<` flag does
not appear in `'cpoptions'` (it does not by default). You can also include the
Vim notation in strings by prepending the notation with a backslash, eg.
`"\<Tab>"` is a string that will contain a single literal tab when the code is
evaluated.

## Toolbox

### Leader and LocalLeader

As a convenience, Vim provides two customizable Vim notation expansions that you
can use in your mappings: `<Leader>` and `<LocalLeader>`. You can set their
value via the `mapleader` and `maplocalleader` variables, eg.:

    let mapleader = "\<Space>"
    let maplocalleader = "&"

You can then use it like this:

    nnoremap <silent> <Leader>w :w<cr>

Now hitting `<Space>w` will save the current buffer.

It is called "leader" as it is generally the first key of a group of mappings
all starting with it. A common setting is to undefine the original behavior of
that key, so that it does not trigger inadvertently:

    nnoremap <Space> <Nop>

Leader and LocalLeader let the user change several mappings in one shot just by
changing the value of the `mapleader` and `maplocalleader` variables, but this
is not something that occurs very often. Also, note that the mechanism is a mere
convenience: `<Leader>` is expanded in mapping commands, but not elsewhere. This
does NOT echo a space:

    :echo "\<Leader>"

And if you show a mapping using Leader, you will see its expansion (`<Space>`):

    :nmap <Leader>w

So all in all, there is not a whole lot to gain with Leader and LocalLeader,
but they at least show intent, and it can make it easier to search through your
vim files for mappings.

### Plug

In a nutshell, `<Plug>` is a Vim notation for some special key sequence _that
the user cannot type_. What use is it?

Imagine you are a plugin author and you have a complicated mapping whose LHS
must be customizable by the user. The first option is to instruct them in the
documentation to copy that complicated mapping in their vimrc and just change
the LHS to their liking. This will work, but the user will have to deal with the
internals of your pluging, which is not ideal. And if you want to change the RHS
in a later version, your users will have to update their own version of the
mapping.

The second option is to make an indirection: create a mapping to your
internal RHS from a simple, intermediate LHS, and expose that LHS in your
documentation. The user will then be able to map his own LHS to your simple LHS,
and your implementation details won't be exposed. Eg.:

    nnoremap <silent> ]&MyPluginIndentLine ...internal RHS...
    nmap <silent> <LocalLeader>f ]&MyPluginIndentLine

Now, if the user want to change the default mapping from `<LocalLeader>i` to
something else, say, `<Leader><Tab>`, they will just to have this in their vimrc:

    nmap <Leader><Tab> ]&MyPluginIndentLine

Then, if you later change the internal RHS part, your users won't have to change
anything. Note that all mappings except the one to the internal RHS are _not_ of
the noremap family, since we do want all mappings to chain together; `nnoremap`
in any of those would break the chain.

This setting would work, but there is a flaw: the intermediate LHS, namely
`]&MyPluginIndentLine`, interfer with normal usage. We paid attention to use a
leading sequence of `]&`, that is not mapped by default, but the user might well
have created a mapping on that very sequence -- and now, each time they will hit
`]&`, there will be a slight delay when Vim waits for the duration of the
timeout to see if it should run `]&` or `]&MyPluginIndentLine`.

That is where `<Plug>` comes in handy: since it is not a sequence made from
normal keys, it gets totally out of the way of other mappings that don't use
`<Plug>` themselves. To use it, just replace the arbitrary `]&` prefix above
with `<Plug>`:

    nnoremap <silent> <Plug>MyPluginIndentLine ...internal RHS...
    nmap <silent> <LocalLeader>f <Plug>MyPluginIndentLine

And in the user's vimrc:

    nmap <Leader><Tab> <Plug>MyPluginIndentLine

Note that the RHS is just the special sequence `<Plug>` expands to (we don't
need to know what it is exactly, though you can get an idea with `:echo
"\<Plug>"`), followed by all the individual letters of "MyPluginIndentLine";
this is not some special command-line or function-invoking mode! Therefore,
conflicts could theoretically arise; suppose we write a snippet plugin with
these two mappings:

    nnoremap <silent> <Plug>MyPluginFor ...internal RHS 1...
    nnoremap <silent> <Plug>MyPluginFori ...internal RHS 2...

The first mapping might insert some for-loop snippet, and the second one could
insert a for-loop using a variable called 'i'.

So far so good, but now a user finds it convenient to go into insert mode right
after inserting the first for-loop variant, and they want to make a mapping for
it:

    " Intent: call <Plug>MyPluginFor and hit 'i' to go into insert mode
    nmap <Leader>i <Plug>MyPluginFori

You see the problem: the mapping will inadvertently call the wrong mapping from
our snippet plugin!

Even though those cases are rare, it is common practice to avoid them by
surrounding the part after `<Plug>` in braces:

    nmap <silent> <LocalLeader>f <Plug>(MyPluginFor)
    nmap <Leader><Tab> <Plug>(MyPluginFori)

In user's vimec:

    " Intent: call <Plug>MyPluginFor and hit 'i' to go into insert mode
    nmap <Leader>i <Plug>(MyPluginFor)i

Problem solved.

### Control-R

`<C-r>` inserts the content of a register from insert and command-line mode. It
is notably useful in visual mode, when the mapping needs to work with the
selection eg.:

    xnoremap <silent> <Leader>gf y:pedit <C-r><C-r>"<cr>

This will open the filename expected in the visual selection into the preview
window. Doubling the `<C-r>` inserts the content literally, in case there were
some control characters in the filename that might be interpreted by Vim.

The expression register can also be used, opening some interesting
possibilities:

    inoremap <C-g><C-t> [<C-r>=strftime("%d/%b/%y %H:%M")<cr>]

That mapping will insert the current date an time between brackets.

Another example, from command-line mode:

    cnoremap <C-x>_ <C-r>=split(histget('cmd', -1))[-1]<cr>

This will insert the last space-separated word from the last command-line, as 
`<M-_>` in Bash.

`<C-r>` can also insert text present under the current cursor, when followed by
some control characters. Here is an example with `<C-r><C-f>`, which inserts the
filename under the cursor;

    nnoremap <silent> <Leader>gf :pedit <C-r><C-f><cr>

This is the normal mode version of the preview mapping we saw above (the
filename recognition will depend on the `'isfname'` option). Note that on the
command-line, for a command where a filename is expected, you can also use a few
special Vim notations to similar effects, eg. `<cfile>` will insert on the
command-line the filename under the cursor, and `<cword>` will insert the
current word. If a filename is not expected, you can always use `expand()` like
this:

    nnoremap <silent> c<Tab> :let @/=expand('<cword>')<cr>cgn

This mapping sets the last search pattern to the word under the cursor, and
change it with the `cgn` sequence -- conveniently repeatable with `.` to apply
the same replacement to some following occurrences.

### @=

The `@` key executes the content of a register, and once again the expression
register offers a good deal of flexibility. As an example, consider the `<C-a>`
normal mode command, that increases the closest number on the right of the
cursor, if any. A common annoyance is words like `file-1.txt`: hitting `<C-a>`
on it will turn it to `file0.txt`, to the surprise of many users, as Vim assumes
the next number is '-1', not '1'. Let's write a mapping to change this behavior.

    function! Increment() abort
      call search('\d\@<!\d\+\%#\d', 'b')
      call search('\d', 'c')
      norm! v
      call search('\d\+', 'ce')
      exe "norm!" "\<C-a>"
      return ''
    endfun

The `Increment()` function finds the sequence of digits under the cursor or
following it, then selects it in visual mode, and finally runs `<C-a>` on it.
The visual mode version of `<C-a>` is relatively recent, so vim8 or a late vim7
version is required. Now, let's remap the normal mode `<C-a>` to our function:

    nnoremap <silent> <C-a> @=Increment()<cr>

The effect is to execute the `Increment()` function in the expression register,
which as we saw increases the next number ignoring leading minuses and returns
the empty string -- leaving nothing to do the `@` command, since the job is
already done.

At first glance, this might just look like a fancy alternative to
`:call Increment()<cr>`, but there's a nice bonus to it: our mapping now accepts
a _count_, so that we can type `3<C-a>` to add three to the next number.

### feedkeys()

The built-in `feedkeys()` function inserts keys into the internal Vim buffer
containing all keys left to execute, either typed by the user or coming from
mappings. This can sound confusing and low-level, but it is a very useful tool.

    nnoremap <silent> <C-g> :call feedkeys(nr2char(getchar()),'nt')<cr>

This mapping executes the next key hit after `<C-g>` ignoring any mapping for
that key -- a kind of "just-once-noremap". `getchar()` is first executed: it
waits for the user to hit a key, and returns its keycode. `nr2char()` converts
that keycode into a character, and `feedkeys()` puts that key into the Vim
internal buffer; the 'nt' options says not to use mappings, and process the key
as though the user typed it. Even though it remaps the useful `<C-g>` built-in,
it also makes it available for free on `<C-g><C-g>`.

Here's a longer example, inspired from igemnace on #vim:

    function! QuickBuffer(pattern) abort
      if empty(a:pattern)
        call feedkeys(":B \<C-d>")
        return
      elseif a:pattern is '*'
        call feedkeys(":ls!\<cr>:B ")
        return
      elseif a:pattern =~ '^\d\+$'
        execute 'buffer' a:pattern
        return
      endif
      let l:globbed = '*' . join(split(a:pattern, ' '), '*') . '*'
      try
        execute 'buffer' l:globbed
      catch
        call feedkeys(':B ' . l:globbed . "\<C-d>\<C-u>B " . a:pattern)
      endtry
    endfun

    command! -nargs=* -complete=buffer B call QuickBuffer(<q-args>)

    nnoremap <Leader>b :B<cr>

Hitting `<Leader>b` will run the user-defined ex command `B`, which will in turn
call the `QuickBuffer()` function. When the latter is called without argument,
it will run `feedkeys(":B \<C-d>")`, with the effect of listing the completion
options of the `B` command -- that is, showing the list of buffers, thanks to
the `-complete=buffer` option of `B`. The `:B` is still on the command-line, so
now the user can pick its choice by entering a part of the wanted buffer. All
the conditionals of the `QuickBuffer()` function will be skipped, and the
`buffer` ex command will be run on the typed argument with leading and trailing
wildcards automatically added. If there is a single match, the buffer will be
displayed and the function ends. If there is no match or more than one match,
the choices will be shown and the `:B` will be put back on the commnd-line (in
the 'catch' part of the function).

The first `elseif` allows for `:B *` to show a full `:ls!` listing, with hidden
buffers. The second `elseif` lets the user select a buffer by number, eg.
`:B 2`, skipping all wildcards addition.


## Use Case -- Lazy loading of a group of mappings

