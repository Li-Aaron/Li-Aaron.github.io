---
layout: post
title: windows下 gvim 配置成 Source Insight
date: 2018-04-30 17:51:00.000000000 +08:00
---
移动自cnblogs: [windows下 gvim8.0 编译器配置](https://www.cnblogs.com/ac2sherry/p/8973649.html)

最近由于各种原因，IDE从source insight换成了vim，参考了诸多博客的文章，折腾了好久折腾了个大概的样子，现在总结一下经验：  
主要参考：  
[改造vim变成source insight](http://qiweiyou1982.blog.163.com/blog/static/14164218620132112548476/)  
[Winxp下 gvim 编程环境搭建](https://blog.csdn.net/minico/article/details/1938050)  
[VIM学习笔记 折叠 (Fold)](https://zhuanlan.zhihu.com/p/27473875)  

效果图（新：Taglist+NERD_Tree+SrcExpl），图为自动补全状态：  
![](/assets/images/2018-04-30-windows-gvim/1146999-20180505011713950-2038894543.png)

效果图（旧：WinManager+SrcExpl）：  
![](/assets/images/2018-04-30-windows-gvim/1146999-20180430173055152-253098202.png)





## 1. 安装：

下载gvim8.0安装就可以了

刚装完是这个样子的：  
![](/assets/images/2018-04-30-windows-gvim/1146999-20180430173011463-1517323194.png)

## 2. 插件安装：

选用的插件如下表 （尝试后发现WinManager和SrcExpl冲突，改用Trinity）

| 插件名（点击下载） | 作用 | 安装方法（vimrc在后面统一配置）|
| ---------------- | ---- | -------------------------- |
| [taglist](http://www.vim.org/scripts/script.php?script_id=273) | 基于ctags的taglist | 解压到.\vim80目录下面 |
| [WinManager](http://www.vim.org/scripts/script.php?script_id=95) | 将FileExplore和Taglist整合 | 解压到.\vim80目录下面 |
| [Ctags](http://ctags.sourceforge.net) | ctags，用于生成tag文件（符号链接）| 将ctags.exe放到.\vim80路径下，并将vim80添加到环境变量/ctags.exe放到system32路径下 |
| [Snipmate](http://www.vim.org/scripts/script.php?script_id=2540) | 提供常用代码快速输入（Tab补齐）| 解压到.\vimfiles目录下面 |
| [Supertab](http://www.vim.org/scripts/script.php?script_id=1643) | 用Tab键自动补齐 | Open the file in vim ($ vim supertab.vmb) and Source the file (:so %) |
| [SrcExpl](http://www.vim.org/scripts/script.php?script_id=2179) | 实现source insight的预览框的功能 | 解压到.\vimfiles目录下面 |
| [Cscope](http://sourceforge.net/projects/mslk/files/) | ctags的强化版，不仅可以生成源tag还能生成调用tag | 将压缩包解压并将目录加入环境变量path中 |
| [Trinity](http://www.vim.org/scripts/script.php?script_id=2347) | NERD_Tree+taglist+SrcExpl的组合版 | 解压到.\vimfiles目录下面 |

## 3. 主题安装：

vim主题选用monokai，字体选用consolas

主题地址：[https://github.com/sickill/vim-monokai](https://github.com/sickill/vim-monokai)

更改文件名为`monokai.vim`后放到`vim80\colors\`里

个人喜好的一些修改：（修改到`monokai.vim`对应的行）
```vim
hi Search term=reverse cterm=NONE ctermfg=231 ctermbg=24 gui=NONE guifg=#f8f8f2 guibg=#AA0000
hi Folded ctermfg=242 ctermbg=235 cterm=NONE guifg=#75715e guibg=#272822 gui=NONE
hi FoldColumn ctermfg=242 ctermbg=235 cterm=NONE guifg=#75715e guibg=#272822 gui=NONE
```

## 4.vimrc更改：

在`_vimrc`文件后增加如下：

```vim
"设置Taglist
let Tlist_Show_One_File=1
let Tlist_Exit_OnlyWindow=1
 
"使用F8打开Taglist
"nmap <silent> <F8> :TlistToggle<CR>
"通过WinManager插件来将TagList窗口和netrw窗口整合起来
let g:winManagerWindowLayout='FileExplorer|TagList'
"nmap <F8> :WMToggle<cr>
"使用F9打开SrcExpl
"nmap <F9> :SrcExplToggle<CR>
 
"Trinity 设置
" Open and close all the three plugins on the same time
nmap <F8>   :TrinityToggleAll<CR>
" Open and close the srcexpl.vim separately
nmap <F9>   :TrinityToggleSourceExplorer<CR>
let g:SrcExpl_jumpKey = "<ENTER>"
let g:SrcExpl_gobackKey = "<SPACE>"
let g:SrcExpl_prevDefKey = "<F3>"
let g:SrcExpl_nextDefKey = "<F4>"
 
"设置SuperTab，用tab键打开cppcomplet的自动补全功能。
let g:SuperTabRetainCompletionType=2
let g:SuperTabDefaultCompletionType="<C-X><C-O>"
"显示行号
set number
"设置主题颜字体
colorscheme monokai
set guifont=Consolas:h12
"为了使用智能补全，打开文件类型检测,关闭VI兼容模式
filetype plugin indent on
set nocp
"字符匹配单词
set incsearch
"代码折叠
set fdm=syntax
set foldlevel=1
set foldcolumn=2
"不换行
set nowrap
"缩进设置
set smartindent
set tabstop=4
set shiftwidth=4
set expandtab
set softtabstop=4
```

## 5. 一些其他设置：

tags生成（虽然Source Explore支持每次打开调用时更新，但如果首次打开没有tags会在当前目录生成，如果打开的是工程内部文件就会导致tags不全，所以首次运行最好生成一下）

```bash
ctags -R ./Drvlib ./Source ./Include
```

cscope数据库生成（路径更改为自己的）

```bash
find -P ./Drvlib ./Source ./Include > cscope.files
cscope -bq
```

cscope数据库包含（路径更改为自己的）

```bash
cscope add ../cscope.out
```
