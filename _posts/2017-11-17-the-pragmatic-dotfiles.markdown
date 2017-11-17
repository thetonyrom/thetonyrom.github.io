---
layout: post
title:  "The Pragmatic Dotfiles"
date:   2017-11-17 17:11:17 +0300
categories: dotfiles
---

Hi there! My name is Tony and I am going to show you the pragmatic way to manage *dotfiles*.

Unix command-line tools can be configured using plain-text hidden files, commonly known as *dotfiles*.
They are called "dotfiles" as they typically are named with a leading `.` making them hidden files on your system.
Since these files are all plain text, we can gather them together in a git repository and use that to track the changes you make over time.

Most likely, as a programmer, you will keep using and improving your dotfiles for your entire career.
Your dotfiles will most likely be the longest project you ever work on.
For this reason, it is worthwhile to organize your dotfiles project in a disciplined manner for maintainability and extensibility.

>You can’t connect the dots looking forward; you can only connect them looking backwards.
>So you have to trust that the dots will somehow connect in your future.
>
>Steve Jobs

## Initialization

At first you need to create a directory for your dotfiles. I suggest to call it `.dotfiles` and put it in the user's home.

{% highlight bash %}
  $ cd && mkdir .dotfiles
{% endhighlight %}

To view all your hidden files and directories you can use `ls`(list) command with an `-a`(all) flag.
Now you need to choose which of them you are going to manage and move to previosly created directory.

{% highlight bash %}
  $ ls -a
  $ mv .vimrc .zshrc .tmux.conf .dotfiles
{% endhighlight %}

At this point you are ready to put your *dotfiles* under version control.

{% highlight bash %}
  $ cd .dotfiles
  $ git init
{% endhighlight %}

If you haven't used version control systems before I highly recommend you to start now!
One of the most popular VCSs is Git. It was created by Linus Torvalds for development of the Linux kernel.
To learn more have a look at this [awesome book][git_book].

## Remote repository

Ok, it is time to push our changes to remote server. I won't describe all steps just give you a few alternatives to choose from: [GitHub][github], [GitLab][gitlab], [Bitbucket][bitbucket].

## Installation

With your dotfiles in their own repository, there are two possible ways to install dotfiles on systems: copying or symbolically linking files.
I prefer symlinks — there is no need to manage discrepancies between copies. Changes to configuration files are changes to the working copy in the repository.

It is as simple as [idempotent][idempotent] [shell script][bash_how_to].

{% highlight bash %}
  #!/bin/bash

  DOTDIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

  # vim
  ln -s ${DOTDIR}/.vimrc ~/.vimrc

  # tmux
  ln -s ${DOTDIR}/.tmux.conf ~/.tmux.conf

  # zsh
  ln -s ${DOTDIR}/.zshrc ~/.zshrc
{% endhighlight %}

## What's next?

Keep calm and master command line.

[git_book]: https://git-scm.com/book/en/v2
[github]: https://github.com/
[gitlab]: https://gitlab.com/
[bitbucket]: https://bitbucket.org/
[bash_how_to]: http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO.html
[idempotent]: https://en.wikipedia.org/wiki/Idempotence
