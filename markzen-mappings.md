# Mappings

Note:
  - Title TBD
  - Number of bullets in each chapter doesn't correlate with word count

* Intro
  * Types of mappings
    * map vs noremap
    * nmap/imap: pretty obvious
    * cmap types - getcmdtype()
    * vmap vs xmap
    * omap
    * others (lmap, tmap) => not much addressed in this article
  * What keys to remap
    * the obvious (function keys, spacebar, some control keys)
    * redundant and semi-redundant keys eg. `s`
    * rarely used keys eg. `,`, `&`, some `g`+letter, some `z`+letter
    * keys used in sequences: X+letter eg. `c`/`d`/`y`/`q`/`m`
      * remap some, let others fall back on original mapping
    * more aggressive/experimental:
      * imaps starting with letters (famous `jk`, + others)
  * What to map
    * new features
    * easier access to frequently used features
      * eg. window-related commands
      * repeated keys for very frequent features eg. `vv` for V
  * On <Leader>
  * RHS how-to (<Bar>)
* Tools
  * <expr>
  * <C-r>
  * @=
  * <Plug>
  * <Buffer>
  * feedkeys()
* Use cases
  * lazy loading group of mappings with same prefix
  * using feedkeys() to unsilent a silent mapping
  * [more]
