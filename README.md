 :heart: for my favorite editor

# A Completion Framework for Neovim

This is a **Fast! Extensible! Async! completion framework** for
[neovim](https://github.com/neovim/neovim).  For more information about plugin
implementation, please read the **[Why](#why) section**.

Future updates, announcements, screenshots will be posted
**[here](https://github.com/roxma/nvim-completion-manager/issues/12).
Subscribe it if you are interested.**

![All in one screenshot](https://cloud.githubusercontent.com/assets/4538941/22727187/78f35172-ee12-11e6-95e5-e9c160151f3b.gif)

## Table of Contents

<!-- vim-markdown-toc GFM -->
* [Available Completion Sources](#available-completion-sources)
* [Requirements](#requirements)
* [Installation](#installation)
* [Configuration Tips](#configuration-tips)
* [How to extend this framework?](#how-to-extend-this-framework)
* [Why?](#why)
    * [Async architecture](#async-architecture)
    * [Scoping](#scoping)
    * [Experimental hacking](#experimental-hacking)
* [FAQ](#faq)
    * [Why Python?](#why-python)
* [Related Projects](#related-projects)

<!-- vim-markdown-toc -->

## Available Completion Sources

plugin builtin sources:

- Keyword from current buffer
- Tag completion. (`:help 'tags'`, `:help tagfiles()`)
- Keyword from tmux session
- Ultisnips hint, if you have installed ultisnips
- Neosnippets hint, if you have installed neosnippets
- File path completion
- Python completion via [jedi](https://github.com/davidhalter/jedi)
- Css completion via vim's builtin
  [csscomplete#CompleteCSS](https://github.com/othree/csscomplete.vim)

scoping features:

- Language specific completion for markdown
- Javascript code completion in html script tag
- Css code completion in html style tag

extra sources:

- Language server protocol via
  [LanguageClient-neovim](https://github.com/autozimu/LanguageClient-neovim)
    - [LanguageServer-php-neovim](https://github.com/roxma/LanguageServer-php-neovim)
- PHP completion via
  [nvim-cm-php-language-server](https://github.com/roxma/nvim-cm-php-language-server)
  (deprecated)
- C/C++ completion via
  [clang_complete](https://github.com/roxma/clang_complete). This plugin
  [requires
  python2](https://github.com/llvm-mirror/clang/commit/abdad67b94ad4dad2d655d48ff5f81d6ccf3852e)
  support for neovim.
- Javascript completion via
  [nvim-cm-tern](https://github.com/roxma/nvim-cm-tern)
- Golang completion via [gocode](https://github.com/nsf/gocode)

## Requirements

- Neovim.
- Or vim8 with `has("python")` or `has("python3")`
- `python3` found in your `$PATH` env variable or setting
  `g:python3_host_prog` to the full path of your python3 executable.

## Installation

- Assumming you're using [vim-plug](https://github.com/junegunn/vim-plug)

```vim
" the framework
Plug 'roxma/nvim-completion-manager'
```

- If you are **vim8 user**, you'll need
  [vim-hug-neovim-rpc](https://github.com/roxma/vim-hug-neovim-rpc). The vim8
  support layer is still experimental, please 'upgrade' to
  [neovim](https://github.com/neovim/neovim) if it's possible.

```vim
" Requires vim8 with has('python') or has('python3')
" Requires the installation of msgpack-python. (pip install msgpack-python)
if !has('nvim')
    Plug 'roxma/vim-hug-neovim-rpc'
endif
```

- Install pip modules for your neovim python3:

```sh
# neovim is the required pip module
# jedi for python completion
# mistune for markdown completion (optional)
# psutil (optional)
# setproctitle (optional)
pip3 --user install neovim jedi mistune psutil setproctitle
```

(Optional) It's easier to use
[python-support.nvim](https://github.com/roxma/python-support.nvim) to help
manage your pip modules for neovim:

```vim
Plug 'roxma/python-support.nvim'
" for python completions
let g:python_support_python3_requirements = add(get(g:,'python_support_python3_requirements',[]),'jedi')
" language specific completions on markdown file
let g:python_support_python3_requirements = add(get(g:,'python_support_python3_requirements',[]),'mistune')

" utils, optional
let g:python_support_python3_requirements = add(get(g:,'python_support_python3_requirements',[]),'psutil')
let g:python_support_python3_requirements = add(get(g:,'python_support_python3_requirements',[]),'setproctitle')

```

- (Optional) Install typical completion sources
```vim
" (optional) javascript completion
Plug 'roxma/nvim-cm-tern',  {'do': 'npm install'}
" (optional) language server protocol framework
Plug 'autozimu/LanguageClient-neovim', { 'do': ':UpdateRemotePlugins' }
" (optional) php completion via LanguageClient-neovim
Plug 'roxma/LanguageServer-php-neovim',  {'do': 'composer install && composer run-script parse-stubs'}
autocmd FileType php LanguageClientStart
```

## Configuration Tips

- Supress the annoying completion messages:

```vim
" don't give |ins-completion-menu| messages.  For example,
" '-- XXX completion (YYY)', 'match 1 of 2', 'The only match',
set shortmess+=c
```

- Use tab to select the popup menu:

```vim
inoremap <expr> <Tab> pumvisible() ? "\<C-n>" : "\<Tab>"
inoremap <expr> <S-Tab> pumvisible() ? "\<C-p>" : "\<S-Tab>"
inoremap <expr> <buffer> <CR> (pumvisible() ? "\<c-y>\<cr>" : "\<CR>")
```

- Trigger Ultisnips or show popup hints [with the same
  key](https://github.com/roxma/nvim-completion-manager/issues/12#issuecomment-278605326)
  `<c-u>`

```vim
let g:UltiSnipsExpandTrigger = "<Plug>(ultisnips_expand)"
inoremap <silent> <c-u> <c-r>=cm#sources#ultisnips#trigger_or_popup("\<Plug>(ultisnips_expand)")<cr>
```

- If you have only `omnifunc` available, you may register it as a source to
  the framework.
 
 
```vim
" css completion via `csscomplete#CompleteCSS`
" The `'cm_refresh_patterns'` is PCRE.
" Be careful with `'scoping': 1` here, not all sources, especially omnifunc,
" can handle this feature properly.
au User CmSetup call cm#register_source({'name' : 'cm-css',
		\ 'priority': 9, 
		\ 'scoping': 1,
		\ 'scopes': ['css','scss'],
		\ 'abbreviation': 'css',
		\ 'cm_refresh_patterns':[':\s+\w*$'],
		\ 'cm_refresh': {'omnifunc': 'csscomplete#CompleteCSS'},
		\ })
```

**Warning:** `omnifunc` is implemented in a synchronouse style, and
vim-vimscript is single threaded, it would potentially block the ui with the
introduction of a heavy weight `omnifunc`, for example the builtin
phpcomplete. If you get some time, please try implementing a source for NCM as
a replacement for the old style omnifunc.


- There's no guarantee that this plugin will be compatible with other
  completion plugin in the same buffer. Use `let g:cm_enable_for_all=0` and
  `call cm#enable_for_buffer()` to use this plugin for specific buffer.

- This example shows how to disable NCM's builtin tag completion. It's also
  possible to use `g:cm_sources_override` to override other default options of
  a completion source.

```vim
let g:cm_sources_override = {
    \ 'cm-tags': {'enable':0}
    \ }
```

## How to extend this framework?

- For really simple, light weight completion candidate calculation, or
  avoiding python, refer to
  [autoload/cm/sources/ultisnips.vim](autoload/cm/sources/ultisnips.vim)
- For really async completion source (strongly encoraged), refer to the gocode
  completion:
  [pythonx/cm_sources/cm_gocode.py](pythonx/cm_sources/cm_gocode.py)

Please upload your screenshot
[here](https://github.com/roxma/nvim-completion-manager/issues/12) after you
created the extension.


## Why?

This project was started just for fun, and it's working pleasingly for me now.
However, it seems there's lots of differences between deoplete, YCM, and
nvim-completion-manager, by implementation.

I haven't read the source of YCM yet. So here I'm describing the basic
implementation of NCM (short for nvim-completion-manager) and some of the
differences between deoplete and this plugin.

### Async architecture

Each completion source should be a standalone process, the manager notifies
the completion source for any text changing, even when popup menu is visible.
The completion source notifies the manager if there's any complete matches
available. After some basic priority sorting between completion sources, and
some simple filtering, the completion popup menu will be triggered with the
`complete()` function by the completion manager.

If some of the completion source is calculating matches for a long long time,
the popup menu will still be shown quickly if other completion sources work
properly. And if the user hasn't changed anything, the popup menu will be
updated after the slow completion source finishes the work.

As the time as of this plugin being created, the completion sources of
deoplete are gathered with `gather_candidates()` of the `Source` object,
inside a for loop, in deoplete's process. A slow completion source may defer
the display of popup menu. Of course it will not block the ui.

IMHO, NCM is potentially faster because all completion sources run in
parallel.

### Scoping

I write markdown files with code blocks quite often, so I've also implemented
language specific completion for markdown file. This is a framework feature,
which is called scoping. It should work for any markdown code block whose
language completion source is avaible to NCM. I've also added support for
javascript completion in script tag of html files, and css completion in style
tag.

The idea was originated in
[vim-syntax-compl-pop](https://github.com/roxma/vim-syntax-compl-pop). Since
it's pure vimscript implementation, and there are some limitations currently
with neovim's syntax api. It's very likely that vim-syntax-compl-pop doesn't
work, for example, javascript completion in markdown or html script tag.  So I
use custom parser in NCM to implement the scoping features.

### Experimental hacking

Note that there's some hacking done in NCM. It uses a per 30ms timer to detect
changes even popup menu is visible, as well as using the `TextChangedI` event,
which only triggers when no popup menu is visible. This is important for
implementing the async architecture. I'm hoping one day neovim will offer
better option rather than a timer or the limited `TextChangedI`.

## FAQ

### Why Python?

YouCompleteMe has [good
explanation](https://github.com/Valloric/YouCompleteMe#why-isnt-ycm-just-written-in-plain-vimscript-ffs).

## Related Projects

[asyncomplete.vim](https://github.com/prabirshrestha/asyncomplete.vim)

