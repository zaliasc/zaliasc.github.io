---
title: "vim settings"
date: 2021-11-02
draft: false
tags: ["tips"]
# categories: ["ca1"]

---



.vimrc

```
" base settings {{{
"""""""""""""""""""""""""""""""""按键映射(just for colemak user)"""""""""""""""""""""""""""""""

"split navigations
nnoremap <C-J> <C-W><C-J>
nnoremap <C-K> <C-W><C-K>
nnoremap <C-H> <C-W><C-H>
nnoremap <C-L> <C-W><C-L>

noremap qq <ESC>:q!<CR>
noremap tsq <ESC>:wq<CR>

"imap jk <ESC>

imap vv <ESC>:w<cr>
map vv <ESC>:w<cr>
" 空格键向上滚屏 光标不变
nnoremap <SPACE> 2<C-e>
" noremap <C-j> 3<C-e>
" noremap <C-k> 3<C-y>

"""""""""""""""""""""""""""""""""""""""""""基本设置"""""""""""""""""""""""""""""""""""""""""""""
filetype on	"开启文件类型侦测
filetype indent on	"适应不同语言的缩进
syntax enable	"开启语法高亮功能
syntax on 	"允许使用用户配色

"""""""""""""""""""""""""""""""""""""""""""显示设置"""""""""""""""""""""""""""""""""""""""""""""
set laststatus=2        	"总是显示状态栏
set ruler               	"显示光标位置
set number              	"显示行号
set cursorline          	"高亮显示当前行
"set cursorcolumn            "高亮显示当前列
set hlsearch                " 高亮搜索结果
exec "nohlsearch"
set incsearch               "边输边高亮
set ignorecase              "搜索时忽略大小写
set smartcase
set relativenumber          "其他行显示相对行号
"set scrolloff=5            "垂直滚动时光标距底部位置

"""""""""""""""""""""""""""""""""""""""""""编码设置"""""""""""""""""""""""""""""""""""""""""""""
set fileencodings=utf-8,gb2312,gbk,gb18030,cp936    " 检测文件编码,将fileencoding设置为最终编码
set fileencoding=utf-8                              " 保存时的文件编码
set termencoding=utf-8                              " 终端的输出字符编码
set encoding=utf-8                                  " VIM打开文件使用的内部编码

"""""""""""""""""""""""""""""""""""""""""""编辑设置"""""""""""""""""""""""""""""""""""""""""""""
set expandtab   	"扩展制表符为空格
set tabstop=2   	"制表符占空格数
set softtabstop=4	"将连续数量的空格视为一个制表符
set shiftwidth=2	"自动缩进所使用的空格数
set textwidth=80	"编辑器每行字符数
set wrap            "设置自动折行
set linebreak       "防止单词内部折行
set wrapmargin=5      "指定折行处与右边缘空格数
set autoindent  	"打开自动缩进
set wildmenu    	"vim命令自动补全
set paste 				"粘贴格式
set mouse=a
set nu " 搜索不区分大小写，但键入了大写则自动区分大小写
set noswapfile " 不产生swp文件
"""""""""""""""""""""""""""""""""""""""""""其他设置"""""""""""""""""""""""""""""""""""""""""""""
set showcmd
set t_Co=256

" set cuc
set showmatch
set undofile
set noerrorbells
set history=2000
set autoread
set nolist
set autochdir
set wildmode=longest:list,full
set laststatus=2
set selection=exclusive
set selectmode=mouse,key
" }}}


""""""""""""""""""""""""""""""""""""""""""""插件设置""""""""""""""""""""""""""""""""""""""""""""
set nocompatible    "关闭兼容模式
filetype off    "文件类型侦测关闭

call plug#begin('~/.vim/plugged')
Plug 'scrooloose/nerdcommenter' "多行注释
Plug 'jiangmiao/auto-pairs'     "括号、引号自动补全
Plug 'scrooloose/nerdtree'      "树形目录
Plug 'Yggdroot/indentLine'      "自动缩进插件
Plug 'vim-airline/vim-airline'  "状态栏主题
Plug 'vim-scripts/Solarized'    "主题
Plug 'honza/vim-snippets'       "代码片段补全
Plug 'SirVer/ultisnips'

Plug 'mhinz/vim-startify'         "vim开始界面最近文件
Plug 'connorholyday/vim-snazzy'   "主题方案
Plug 'tpope/vim-commentary'       "代码注释
Plug 'ryanoasis/vim-devicons'     "文件图标
Plug 'Lokaltog/vim-powerline'     "状态栏主题

call plug#end()
filetype plugin indent on

"solarized
set background=dark
colorscheme solarized   "素雅
"hi Normal ctermbg=none "设置背景透明

"vim-airline
let g:airline_powerline_fonts=1


"Lokaltog
"let g:Powerline_colorscheme='solarized256'  "设置状态栏主题风格

"nerdtree
"autocmd vimenter * NERDTree
map <F2> :NERDTreeToggle<CR>
let NERDTreeWinSize=40
let NERDTreeWinPos="left"

"indentLine
let g:indentLine_char='┆'       "缩进指示线符       
let g:indentLine_enabled = 1    "开启缩进指示

"Snippet
let g:UltiSnipsSnippetDirectories=[$HOME.'/.vim/plugged/vim-snippets/UltiSnips']
let g:UltiSnipsExpandTrigger = "<tab>"
let g:UltiSnipsListSnippets = "<c-tab>"
let g:UltiSnipsJumpForwardTrigger = "<tab>"
let g:UltiSnipsJumpBackwardTrigger = "<s-tab>"
let g:UltiSnipsEditSplit="vertical"
```