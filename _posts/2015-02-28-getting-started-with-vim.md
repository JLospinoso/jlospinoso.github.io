---
layout: post
title: Getting started with Vim
date: 2015-02-28 17:37
author: jalospinoso
comments: true
categories: [developing, software, vim]
---
[1]: http://en.wikipedia.org/wiki/Vi
[2]: http://www.amazon.com/The-Pragmatic-Programmer-Journeyman-Master/dp/020161622X
[3]: http://www.vim.org/download.php
[4]: http://vim.rtorr.com/
[5]:  https://pascalprecht.github.io/2014/03/18/why-i-use-vim/

[vimlogo]: {{site.url}}/images/vim.jpg "Vim"
[vim1]: {{site.url}}/images/2015-02-28_1.jpg "Normal Mode on Vim"
[vim2]: {{site.url}}/images/2015-02-28_2.jpg "Insert Mode on Vim"
[vim3]: {{site.url}}/images/2015-02-28_3.jpg "Text Inserted"
[vim4]: {{site.url}}/images/2015-02-28_4.jpg "Learning Curve"

![vimlogo]

Learning to use Vim can be a great investment. It's installed on many platform by default (well, at least its ancestor [Vi][1]), highly customizable, and regardless of what you're editing--config files, source from various languages, etc.--it is highly portable. In [The Pragmatic Programmer][2], we are encouraged to become masters at editors like Vim and Emacs just as a woodworker must become a master at hand tools.

Vim proficiency delivers a lot of benefits:

* You don't need a mouse, and you use as few keystrokes as possible
* It's a *modal editor*--you have normal mode, visual mode, insert mode, etc., so you are able to go from editing contents to manipulating the file system without batting an eye
* Vim is available on many more platforms than whatever IDE you are considering
* Once you are proficient with Vim, you don't need to learn a whole new IDE for every language you'd like to learn

### Installing Vim

For a Windows install, it's as simple as visiting [Vim.org][3] and running the installer.

### Some Basics

There are two modes to focus on when first getting started with Vim: Insert mode and normal mode. Insert mode allows you to manipulate the contents of a file just like in another editor. Normal mode allows you to manipulate and navigate the text easily and efficiently. Press `ESC` to navigate to normal mode, and press `I` to get back to insert mode. It is easy to see which mode you are in by looking at the top of the editor.

I'm using GVIM on Windows, which starts up in normal mode:

![vim1]

If I press <strong>I,</strong> the prompt at the bottom of the screen tells me that I've switched into insert mode:

![vim2]

Once in insert mode, you can add and manipulate content just like in other editors. Try writing some text and pressing ESC to get back into normal mode:

![vim3]

You can navigate around on your document now (without having to use the mouse or arrow keys!). Check out the keys `H`, `J`, `K`, and `L` for movement from character to character. To move by words, try `W`, `E`, and `B`. There are many, many commands in *normal mode*. Check out this [cheat sheet][4] and try some of the *Cursor Movement* commands.

To save a new file, type `:sav c:\Users\MyUserName\hello.txt` and press `Enter`. You can now make modifications to the file and save the file using `:w` (while in *normal mode*, of course). To exit, use the command `:q`.

### Useful Resources

Vim has a notoriously steep learning curve. Well, it's more like a wall (from [pascalprecht][5]):

![vim4]

Here are some important resources that can help to ease the pain:

* [Another cheat sheet](http://www.angelwatt.com/coding/notes/vim-commands.html)
* [VimDoc](http://vimdoc.sourceforge.net/)
* [Steve Oualline's Vim book](ftp://ftp.vim.org/pub/vim/doc/book/vimbook-OPL.pdf)
* [Interactive Vim tutorial](http://www.openvim.com/" target="_blank">http://www.openvim.com/)
* [Daniel Miessler's screencast](https://danielmiessler.com/blog/vim-primer-screencast/)
