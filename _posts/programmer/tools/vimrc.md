title: VIM 配置文件
date: 2016-05-22 13:03:24
toc: true
tags: [vim]
categories: programmer
keywords: [vim, 配置, vimrc]
description: vim 配置文件备忘
---

这是我的个人 vim 配置。

```vim
" 1. Install bundle
" git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
"
" 2. Powerline font creator
"    a. sudo apt-get install python-fontforge
"    b. /path/to/fontpatcher MyFontFile.ttf
"    c. cp MyFontFile-Powerline.otf ~/.fonts
"    d. sudo fc-cache -vf
"    e. Change your console font. Bingo!

set nocompatible               " be iMproved
filetype off                   " required!

set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

" let Vundle manage Vundle
" required! 
Bundle 'gmarik/vundle'

"Bundle 'Valloric/YouCompleteMe'
"Bundle 'rdnetto/YCM-Generator'
Bundle 'scrooloose/nerdtree'
Bundle 'fholgado/minibufexpl.vim'
Bundle 'vim-scripts/taglist.vim'
Bundle 'kien/ctrlp.vim'
"Bundle 'brookhong/cscope.vim'
"Bundle 'mbbill/echofunc'

Bundle 'vim-scripts/winmanager'
Bundle 'vim-scripts/CRefVim'
"Bundle 'vim-scripts/AutoComplPop'
"Bundle 'vim-scripts/EasyColour'
"Bundle 'vim-scripts/clibs.vim'
"Bundle 'vim-scripts/OmniCppComplete'
Bundle 'plasticboy/vim-markdown'
"Bundle 'ervandew/supertab'
Bundle 'Lokaltog/vim-powerline'

" Search
"Bundle 'ggreer/the_silver_searcher'
"Bundle 'rking/ag.vim'
Bundle 'dyng/ctrlsf.vim'

Bundle 'vim-scripts/a.vim'

"Bundle 'vim-scripts/cream-statusline-prototype'
"Bundle 'xolox/vim-easytags'
"Bundle 'tpope/vim-vividchalk'
"Bundle 'xolox/vim-misc'
"Bundle 'wesleyche/SrcExpl'
"Bundle 'vim-scripts/indexer.tar.gz'
"Bundle 'vim-scripts/vimprj'
"Bundle 'vim-scripts/DfrankUtil'

" My Bundles here:
filetype plugin on
filetype plugin indent on     " required!

colorscheme elflord

"
" Brief help
" :BundleList          - list configured bundles
" :BundleInstall(!)    - install(update) bundles
" :BundleSearch(!) foo - search(or refresh cache first) for foo
" :BundleClean(!)      - confirm(or auto-approve) removal of unused bundles
"
" see :h vundle for more details or wiki for FAQ
" NOTE: comments after Bundle command are not allowed..

let g:syntastic_ignore_files=[".*\.py$"]
let g:ycm_global_ycm_extra_conf = '~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py'

set nocp
" set MiniBufExplorer
let g:miniBufExplMapWindowNavVim = 1 
let g:miniBufExplMapWindowNavArrows = 1 
let g:miniBufExplMapCTabSwitchBufs = 1 
let g:miniBufExplModSelTarget = 1
let g:miniBufExplMoreThanOne=0

" set winmanager
" Caution: current version of winmanager has a problem
" need to modify ~/.vim/plugin/winmanager.vim
" function ToggleWindowsManager()下的call
" s:StartWindowsManager()的下一行加一句exe 'q'
let g:NERDTree_title="[NERDTree]"
let g:winManagerWindowLayout="MiniBufExplorer,NERDTree|TagList"
"let g:winManagerWindowLayout="MiniBufExplorer,TagList"

function! NERDTree_Start()
   exec 'NERDTree'
endfunction

function! NERDTree_IsValid()
    return 1
endfunction

nmap wm :WMToggle<CR>

" set clib.vim
let c_hi_identifiers = 'all'
let c_hi_libs = ['*']

set hlsearch
set nu
set cc=100
set ts=4
set shiftwidth =4
set expandtab
set autoindent
set cindent
set smartindent
set softtabstop=4

" 关闭omnicpp预览窗口
set completeopt=menu
let OmniCpp_ShowPrototypeInAbbr = 1 " show function prototype  in popup window

let g:vim_markdown_folding_disabled=1

" powerline的设置
let g:Powerline_symbols = 'fancy'
set fillchars+=stl:\ ,stlnc:\
set t_Co=256
set laststatus=2   " Always show the statusline

let mapleader=','
" 使用 ctrlsf.vim插件在工程内全局查找光标所在关键字，设置快捷键。快捷键速记法：search in project
nnoremap <Leader>sf :CtrlSF<CR>

" Encoding
" Recognition GBK encoding first
set encoding=utf-8
set fenc=cp936
set fileencodings=cp936,ucs-bom,utf-8
" Avoid menu garbled characters
if v:lang =~? '^\(zh\)\|\(ja\)\|\(ko\)'
    set ambiwidth=double
endif
set nobomb
```
