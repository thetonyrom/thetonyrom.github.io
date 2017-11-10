---
layout: post
title:  "VIM in a week!"
date:   2017-11-10 13:28:11 +0300
categories: vim
---

Hi there! My name is Tony and I am going to show you the way how to conquer VIM in a week!

>And remember, VIM is a process.

## Installation

If you don't have it installed yet.

### OSX

In case you haven't heard about **homebrew** follow the [link][homebrew]{:target="blank"}.

{% highlight bash %}
  $ brew install vim
{% endhighlight %}

### \*nix

*Just use your favorite package manager;)*

Well, we are fully equipped and ready to go! To start Vim just type in your terminal:

{% highlight bash %}
  $ vim
{% endhighlight %}

## The most common question

Don't worry if you are stuck there. A lot of people don't know [how to exit the Vim editor][exit_vim]{:target="blank"}. There is even a book!

![exiting_vim](/assets/images/posts/vim/exit_vim.jpg)

Ok, I'm kidding:)

To exit the vim editor you should be in **Normal mode**. Hit `Esc` key to enter it. Then type `:` to enter **Command-line mode**. A colon will appear at the bottom of the screen and you can type in the `quit` command or just `q`(short for *quit*). To execute the command, press the `Enter` key.

## Modes

How you've seen it is all about modes. Vim is not like any other editor or IDE which you've been using previously. But this difference and makes it so powerful.

## Vimtutor

Less words, more action. Let's get started! Vim comes with a nice tutor. Open your terminal and type in:

{% highlight bash %}
  $ vimtutor
{% endhighlight %}

This is your best friend for the next few days. I want you to become really comfortable with the main motions. It should be on your fingertips.

## ~/.vimrc

Vim is not really impressive out of the box, but it is higly customizable. You can use a dotfile called `vimrc` to impove its appearence and add plugins for your purposes. At this point you are ready to unleash your power with Vim! It's up to you;)


#### Additional recources: [vimcasts][vimcasts], [vimadventures][vimadventures], [google][google].

>And remember, VIM is a process!

[homebrew]: https://brew.sh/
[exit_vim]: https://stackoverflow.blog/2017/05/23/stack-overflow-helping-one-million-developers-exit-vim/
[vimcasts]: http://vimcasts.org/
[vimadventures]: https://vim-adventures.com/
[google]: https://www.google.com/
