---
title: 'Basic Vim Setup'
date: 2021-01-18
permalink: /posts/2021/01/basic-vim-setup/
tags:
  - Vim
  - Static Linting
---

Vim is by no means the only code editing tool out there...it's just the best one :grin:.
It is highly customizable, and if you choose it as your editor you will modify it over time to suit your needs.
For beginners, a very simple setup can go a long way in helping to develop code.

![Display of syntax error feedback in Vim](/images/error-tray.jpg)

## TL/DR
Don't want to read?
Copy/paste the following block in your terminal, if it works (and you don't care why), no need to read on.
```bash
mkdir -p ~/.vim/autoload ~/.vim/bundle && \
curl -LSso ~/.vim/autoload/pathogen.vim https://tpo.pe/pathogen.vim && \
printf '%s\n' '' '"Plugin manager' 'execute pathogen#infect()' \
'"Syntax help' 'syntax on' 'filetype plugin indent on' '' \
'let g:syntastic_cpp_check_header = 1' 'let g:syntastic_auto_loc_list = 1' \
'let g:syntastic_always_populate_loc_list = 1' \
'let g:syntastic_check_on_open = 1' 'let g:syntastic_check_on_wq = 0' '' >> ~/.vimrc && \
git clone --depth=1 https://github.com/vim-syntastic/syntastic.git ~/.vim/bundle/syntastic
```

---

## Pathogen
Like most complex text editors, Vim allows enhancements through plugins.
Pathogen is a great tool that automatically loads all plugins in the ~/.vim/bundle directory.

### Install Pathogen
```bash
mkdir -p ~/.vim/autoload ~/.vim/bundle && \
curl -LSso ~/.vim/autoload/pathogen.vim https://tpo.pe/pathogen.vim
```

**If you're using Windows _(not applicable if using Ubuntu or other Linux systems inside Windows)_, change all occurrences of ~/.vim to ~\vimfiles**

## ~/.vimrc
This is the configuration file for Vim and is likely to grow a lot over time if you use Vim a lot.
Add pathogen loading to your vim config file.
If you aren't sure how to do that, running this command will add the necessary content to your config
```bash
printf '%s\n' '' '"Plugin manager' 'execute pathogen#infect()' \
'"Syntax help' 'syntax on' 'filetype plugin indent on' '' \
'let g:syntastic_cpp_check_header = 1' 'let g:syntastic_auto_loc_list = 1' \
'let g:syntastic_always_populate_loc_list = 1' \
'let g:syntastic_check_on_open = 1' 'let g:syntastic_check_on_wq = 0' '' >> ~/.vimrc && \
>> ~/.vimrc
```

## Syntastic
Syntastic is a syntax static checker that will check for syntax errors when you save a file.
To _install_ Syntastic, just run
```bash
git clone --depth=1 https://github.com/vim-syntastic/syntastic.git ~/.vim/bundle/syntastic
```

That's it.
Pathogen will handle the loading of the plugin for you and nothing more should be needed.
The README files for both Pathogen and Syntastic have other options that you might be interested in, but this should get you up and running.
