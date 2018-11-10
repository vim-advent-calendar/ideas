The Road to Integration
=======================================================================

Topic thesis (WIP, from topics.md):

Dev workflows vary from person to person, although they do share some
common tasks such as kicking off build processes, linting, formatting,
testing, running code, etc. This article will talk about integrating
these tasks into Vim.

Personal notes:

- Re: title: obviously still WIP since I still have to write the whole
  article, but I like how "Road to Integration" implies that instead of
  supplying pre-packaged solutions (which is what IDEs do better), the
  article will instead help the reader build his own tailor-fit
  customizations to match his unique workflow better.
- What to cover -- the major points:
  - Build processes, so `:make` and related settings
  - Linting. Should fall under `:make`, but it might do well to point
    out that the feature can be used for more than just literally `make`
    and other build runners. Might still need new ideas, since there's
    already material out there, e.g. [vimgor's why you don't need
    Syntastic][1]
  - Formatting, so `gq`, `=`, and related settings (Update: taken by a different
    topic)
  - Running code. Quickly running through an interpreter is fine. Might
    even be able to still take advantage of `:make` for this.
    Interacting with REPLs is trickier. Might merit plugin suggestions
    here since I use one to interact with tmux panes (as well as taking
    advantage of file watchers). I'll also need to do research re: how
    to take advantage of `:terminal` for this.
  - Debugging? I'll need to do a *lot* of research if I include this,
    since I don't debug from within Vim myself. `:terminal` was followed
    closely by a `termdebug` package, so this merits some thought.
  - Completion? Still unsure if this should be included, but I'm
    currently leaning towards "No". Some other topics like LSP and such
    are more suited, and leaving this out conveniently gives me a more
    focused "`:make` and quickfix to the fullest" theme.
  - Search? Same boat as completion above
  - Re: "Jump to definition". I'd definitely leave this one out, since a
    whole article is already being devoted to tags.
- Bonus minor ideas:
  - Fugitive's `:Gmerge`, which I find extra useful after having `git
    merge` result in conflicts, since it loads up the conflicts in the
    quickfix.
  - Client/server feature. A combination of this feature, a short
    script, and some cool tooling bundled with React Native allows me to
    tap on errors in the stack trace on my test device and have the
    relevant file pop up in my current running Vim on my dev machine.
    Definitely bumped down to "minor" status for being very specific in
    its use case, but might make for a great point to discuss,
    especially in an "integrating Vim with the dev environment" article.

# Draft

## Introduction

> The Fifth is lacking in the monk Wangohan. 
> If asked to hunt a deer for supper, 
> he will choose the shuriken against all objections, 
> and chase a bleeding deer for thirty days.
>
> - [Inaction Hero, The Codeless Code][case206-quote]
 
How many times have you seen Vim compared to an IDE? For the practical software
engineer, it's a comparison that definitely merits some thought. After all,
tools should be chosen with a critical mindset. ((FIXME: Empty trash. This
should never hit master. What point am I *really* trying to make?))

((FIXME: For the life of me I can't figure out a good intro flow to segue into
"Road to Integration" and what it stands for.))

## The build process

Vim has a feature expressly made for kicking off build processes: `:make`. At
the very least, you can use it to run make:

```vim
" run make without arguments
:make
" build a specific target
:make server
```

Very straightforward. But the name can be a bit misleading; you're not limited
to make for your build process. The `makeprg` setting controls what program
`:make` starts.

For example, in the frontend Javascript projects I work on, build processes are
run with NPM scripts:

```vim
:set makeprg=npm\ run

" run the main build chain
:make build
" run a script, e.g. a specific build target
:make build:docs
```

Convenient! Though, not *that* convenient right now; it's still not much of a
departure from just running `npm run build` in your shell. You can map some keys
to make it faster:

```vim
nnoremap <F5> :make build<CR>
```

But by far, the most convincing argument for doing this from Vim is the quickfix
list.

## Build errors and the quickfix list

## Linting, and beyond

## Running code

## Integration in its natural habitat

[1]: https://gist.github.com/ajh17/a8f5f194079818b99199
[case206-quote]: http://thecodelesscode.com/case/206
