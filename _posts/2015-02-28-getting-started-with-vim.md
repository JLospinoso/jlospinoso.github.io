---
layout: post
title: Getting started with Vim
date: 2015-02-28 17:37
author: jalospinoso
comments: true
categories: [developing, software]
---
Learning to use Vim can be a great investment. It's installed on many platform by default, highly customizable, and regardless of what you're editing--config files, source from various languages, etc.--it is highly portable. In <span style="text-decoration:underline;">The Pragmatic Programmer</span>, we are encouraged to become masters at text editors like Vim and Emacs just as a woodworker must become a master at hand tools.

Vim proficiency delivers a lot of benefits:
<ul>
	<li>You don't need a mouse, and you use as few keystrokes as possible</li>
	<li>It's a <em>modal editor</em>--you have normal mode, visual mode, insert mode, etc., so you are able to go from editing contents to manipulating the file system without batting an eye</li>
	<li>Vim is available on many more platforms than whatever IDE you are considering</li>
	<li>Once you are proficient with Vim, you don't need to learn a whole new IDE for every language you'd like to learn.</li>
</ul>
<b>Installing Vim</b>

It's as simple as visiting <a href="http://www.vim.org/download.php" target="_blank">http://www.vim.org/download.php</a> and running the installer (at least, on Windows).

<strong>Some Basics</strong>

There are two modes to focus on when first getting started with Vim: Insert mode and normal mode. Insert mode allows you to manipulate the contents of a file just like in another editor. Normal mode allows you to manipulate and navigate the text easily and efficiently. Press <strong>ESC</strong> to navigate to normal mode, and press I to get back to insert mode. It is easy to see which mode you are in by looking at the top of the editor.

I'm using GVIM on Windows, which starts up in normal mode:

<a href="https://jalospinoso.files.wordpress.com/2015/02/normalmodevim.jpg"><img class="alignnone size-medium wp-image-141" src="https://jalospinoso.files.wordpress.com/2015/02/normalmodevim.jpg?w=300" alt="NormalModeVim" width="300" height="150" /></a>

If I press <strong>I,</strong> the prompt at the bottom of the screen tells me that I've switched into insert mode:

<a href="https://jalospinoso.files.wordpress.com/2015/02/insertmodevim.jpg"><img class="alignnone size-medium wp-image-142" src="https://jalospinoso.files.wordpress.com/2015/02/insertmodevim.jpg?w=300" alt="InsertModeVim" width="300" height="152" /></a>

Once in insert mode, you can add and manipulate content just like in other editors. Try writing some text and pressing ESC to get back into normal mode:

<a href="https://jalospinoso.files.wordpress.com/2015/02/textinserted.jpg"><img class="alignnone size-medium wp-image-143" src="https://jalospinoso.files.wordpress.com/2015/02/textinserted.jpg?w=300" alt="TextInserted" width="300" height="153" /></a>

You can navigate around on your document now (without having to use the mouse or arrow keys!). Check out the keys H, J, K, and L for movement from character to character. To move by words, try W, E, and B. There are many, many commands in Normal mode. Check out the cheat sheet at <a href="http://vim.rtorr.com/" target="_blank">http://vim.rtorr.com/</a> and try some of the "Cursor Movement" commands.

To save a new file, type <strong>:sav c:\Users\MyUserName\</strong><strong>hello.txt</strong> and press enter. You can now make modifications to the file and save the file using <strong>:w</strong> (while in normal mode, of course). To exit, use the command <strong>:q</strong>.

<b>Useful Resources</b>

Vim has a notoriously steep learning curve. Well, it's more like a wall (adopted from https://pascalprecht.github.io/2014/03/18/why-i-use-vim/):

<a href="https://jalospinoso.files.wordpress.com/2015/02/vim-learn-curve.jpg"><img class="alignnone size-medium wp-image-137" src="https://jalospinoso.files.wordpress.com/2015/02/vim-learn-curve.jpg?w=300" alt="vim-learn-curve" width="300" height="225" /></a>

Here are some important resources that can help to ease the pain:
<ul>
	<li>Another cheat sheet: <a href="http://www.angelwatt.com/coding/notes/vim-commands.html" target="_blank">http://www.angelwatt.com/coding/notes/vim-commands.html</a></li>
	<li>VimDoc: <a href="http://vimdoc.sourceforge.net/" target="_blank">http://vimdoc.sourceforge.net/</a></li>
	<li>Steve Oualline's Vim book: <a href="ftp://ftp.vim.org/pub/vim/doc/book/vimbook-OPL.pdf" target="_blank">ftp://ftp.vim.org/pub/vim/doc/book/vimbook-OPL.pdf</a></li>
	<li>Interactive Vim tutorial: <a href="http://www.openvim.com/" target="_blank">http://www.openvim.com/</a></li>
	<li>Daniel Miessler's screencast: <a href="https://danielmiessler.com/blog/vim-primer-screencast/" target="_blank">https://danielmiessler.com/blog/vim-primer-screencast/</a></li>
</ul>
