---
layout   : post
title    : "Highlight current word on Vim"
date     : 2020-03-11
excerpt  : "Reproduce a VSCode feature in a few lines of Vimscript"
---

I'm a Vim user, but I sometimes wonder if some features from other editors, like VSCode would suits my workflow.
Highlighting the current word is one of them.<br />
I set my objective to reproduce it on Vim without becoming to heavy.
<center><i>VSCode highlight on <code>pub</code> word</i></center>
![test](/images/2020-03-11-vim-highlight/vscode.png)

First step is to define the file structure:
- **Autoload** file, where the code resides. Autoload files allow Vim to load the function only when needed, not before.
- **Plugin** file to handle the call to the autoload function.
```
│
└── .vim
      ├── plugin
      │   └── highlight_current_word.vim
      └── autoload
          └── highlight_current_word.vim
```

The plugin file is the easiest, it needs to have a **autocommand group**, and an **autocommand** to call the highlight function (from *autoload*) when the `CursorMoved` event is dispatched.<br />
Additionally, we will also create a **highlight group** for the word, defining background and foreground.
```vim
" File: .vim/plugin/highlight_current_word.vim
highlight HighlightCurrentWord guibg=#463626 ctermbg=94

augroup highlight_current_word
    au!
    au CursorMoved * call highlight_current_word#fn()
augroup END
```

For the main function, the idea is to leverage on `matchadd()` and `matchdelete()`.<br />
`matchadd()` assign a highlight group according to a given regular expression. An **id** can be provided to identify and cancel the assignment with `matchdelete()`

The complete logic will be the following:
1. Select a random ID
2. Match delete past highlight group matching the ID
3. Find the word under the cursor
4. Assign the highlight group to the current word, with the same random ID

```vim
" File: .vim/autoload/highlight_current_word.vim
function! highlight_current_word#fn()
  let exclude_ft = ["nerdtree", "fugitive", "fzf"]
    if index(exclude_ft, &filetype) == -1
        try
            call matchdelete(481516)
        catch
        endtry
        let current_word = escape(expand("<cword>"), "\\/[]*~")
        call matchadd("HighlightCurrentWord", "\\<" . current_word ."\\>", 0, 481516)
    endif
endfunction
```

Additionally, it is possible to check that the **filetype** of the current buffer is not part of a blacklist, to avoid unnecessary highlights.<br />
Also, for `current_word`, we are using `escape()` in order to prevent special characters to be matched with `matchadd()`

Result is the following:
<center><i>Vim highlight on <code>pub</code> word</i></center>
![test](/images/2020-03-11-vim-highlight/vim.png)

You can find the source of this files on my [dotfiles on Github](https://github.com/Baspar/dotfiles/tree/master/vim/.vim)
