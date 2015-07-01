---
layout: post
title: "Vim Tricks: Fixing Indentation"
date: 2015-01-02 11:34:59 -0500
comments: true
categories: vim
---

One of the annoying things about working with Vim is pasting from the clipboard directly into insert mode. Your indentation can easily get all screwy:

For example:

```vim
fu! SaveSess()
    execute 'call mkdir(%:p:h/.vim)'
        execute 'mksession! %:p:h/.vim/session.vim'
        endfunction

        fu! RestoreSess()
        execute 'so %:p:h/.vim/session.vim'
        if bufexists(1)
            for l in range(1, bufnr('$'))
                    if bufwinnr(l) == -1
                                exec 'sbuffer ' . l
                                        endif
                                            endfor
                                            endif
                                            endfunction

                                            autocmd VimLeave * call SaveSess()
                                            autocmd VimEnter * call RestoreSess()

```

Thankfully Vim has an awesome autoindent feature which is bound to `=` by default. Two useful commands are `gg=G` to reident the whole file and using visual mode then `=`.

```vim
fu! SaveSess()
  execute 'call mkdir(%:p:h/.vim)'
  execute 'mksession! %:p:h/.vim/session.vim'
endfunction

fu! RestoreSess()
  execute 'so %:p:h/.vim/session.vim'
  if bufexists(1)
    for l in range(1, bufnr('$'))
      if bufwinnr(l) == -1
        exec 'sbuffer ' . l
      endif
    endfor
  endif
endfunction

autocmd VimLeave * call SaveSess()
autocmd VimEnter * call RestoreSess()
```
