---
title: Linux vim常用配置
date: 2019-04-12 20:42:06
categories:  vim
tags: [ Linux, vim ]

---

列举一些有用的vim小功能, 能有效的避免重复造轮子.

<!-- more -->


# vim新建文件时, 按F4既可以添加作者信息

~/.vimrc中追加如下内容

```
"进行版权声明的设置
""添加或更新头
map <F4> :call TitleDet() <cr>'s
function AddTitle()
    call append(0,"/********************************************************")
    call append(1,"* Author        : ×××")
    call append(2,"* Email         : ×××@×××.com")
    call append(3,"* Last modified : ".strftime("%Y-%m-%d %H:%M"))
    call append(4,"* Filename      : ".expand("%:t"))
    call append(5,"* Description   : ")
    call append(6,"*********************************************************/")
    echohl WarningMsg | echo "Successful in adding the copyright." | echohl None
endf
"更新最近修改时间和文件名
function UpdateTitle()
    normal m'
    execute '/# *Last modified:/s@:.*$@\=strftime(":\t%Y-%m-%d %H:%M")@'
    normal ''
    normal mk                                         
    execute '/# *Filename:/s@:.*$@\=":\t\t".expand("%:t")@'
    execute "noh"                               
    normal 'k
    echohl WarningMsg | echo "Successful in updating the copy right | echohl None
endfunction
"判断前10行代码里面，是否有Last modified这个单词，
"如果没有的话，代表没有添加过作者信息，需要新添加；
"如果有的话，那么只需要更新即可
function TitleDet()     
    let n = 1
    "默认为添加
    while n < 7
        let line = getline(n)
        if line =~ '^\#\s*\S*Last\smodified:\S*.*$'
            call UpdateTitle()
            return
        endif
        let n = n+1
    endwhile
    call AddTitle()
endfunction
```


# vim新建python或者bash脚本添加固定内容

~/.vimrc中追加如下内容

```
autocmd BufNewFile *.py,*.sh, exec ":call SetTitle()"
let $author_name = "taot.jin"
let $author_email = "taot.jin@q.com"

func SetTitle()
    if &filetype == 'sh'
    call setline(1,"\###################################################################")
    call append(line("."), "\# File Name: ".expand("%"))
    call append(line(".")+1, "\# Author: ".$author_name)
    call append(line(".")+2, "\# mail: ".$author_email)
    call append(line(".")+3, "\# Created Time: ".strftime("%c"))
    call append(line(".")+4, "\#=============================================================")
    call append(line(".")+5, "\#!/bin/bash")
    call append(line(".")+6, "")
    else
    call setline(1,"\###################################################################")
    call append(line("."), "\# File Name: ".expand("%"))
    call append(line(".")+1, "\# Author: ".$author_name)
    call append(line(".")+2, "\# mail: ".$author_email)
    call append(line(".")+3, "\# Created Time: ".strftime("%c"))
    call append(line(".")+4, "\#=============================================================")
    call append(line(".")+5, "\#!/usr/bin/python")
    call append(line(".")+5, "\# -*- coding: utf-8 -*-")
    "call append(line(".")+6, "")
    endif
endfunc
```

# vim新建markdown文件时添加固定信息

~/.vimrc中追加如下内容

```
autocmd BufNewFile *.md, exec ":call SetTitle1()"
let $author_name = "taot.jin"
let $author_email = "taot.jin@q.com"

func SetTitle1()
    call setline(1,"---")
    call append(line("."), "title: ")
    call append(line(".")+1, "date: ".strftime("%Y-%m-%d"))
    call append(line(".")+2, "tags: ")
    call append(line(".")+3, "categories: ")
    call append(line(".")+4, "top: ")
    call append(line(".")+5, "description: ")
    call append(line(".")+6, "password: ")
    call append(line(".")+7, "")
    call append(line(".")+8, "---")
endfunc
```
