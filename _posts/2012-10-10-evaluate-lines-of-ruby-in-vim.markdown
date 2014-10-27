---
layout: post
title: "Evaluate Lines of Ruby in Vim"
categories: ruby vim
---

One of the joys of working with software is finding the right tools and customizing them for optimal use. In that sense, I've been tweaking Vim for use with Ruby. One feature in particular which I've wanted to include is the ability to evaluate a line of Ruby and print the result as an inline comment. For example, we start with:

``` ruby
"string".reverse
```

and then hit some key combination and get this:

``` ruby
"string".reverse # => "gnirts"
```

The feature comes built-in in TextMate, but requires additional set up to work in Vim.

After some quick research, I found an [answer](http://stackoverflow.com/questions/4752583/execute-and-update-markers-for-vim) on StackOverflow. The necessary software in question is ```rcodetools``` which includes the command line program: ```xmpfilter```.

Since ```rcodetools``` is a Ruby gem, installation is simply a matter of using ```gem``` to pull down the code:

``` bash
gem install rcodetools
```

There is more to ```rcodetools``` than just ```xmpfilter```. For those interested, ```rcodetools``` includes several README's explaining the installation process to get everything working with the editor of choice (e.g., Vim or emacs).

In my case, though, I was only interested in ```xmpfilter``` and so was happy to find [vim-ruby-xmpfilter](https://github.com/t9md/vim-ruby-xmpfilter), a Vim plugin which consists of a basic wrapper function to invoke ```xmpfilter``` in Vim.

After installing the Vim plugin, the last step is assigning key-mappings in the ```.vimrc``` file. Suggested key-mappings are included as part of ```vim-ruby-xmpfilter``` and are as follows:

``` 
nmap <buffer> <F5> <Plug>(xmpfilter-run)
xmap <buffer> <F5> <Plug>(xmpfilter-run)
imap <buffer> <F5> <Plug>(xmpfilter-run)

nmap <buffer> <F4> <Plug>(xmpfilter-mark)
xmap <buffer> <F4> <Plug>(xmpfilter-mark)
imap <buffer> <F4> <Plug>(xmpfilter-mark)
```

And then voila! Type a line of Ruby in Vim, press ```F4``` to mark the line for evaluation (i.e., ```# =>```) and then press ```F5``` and ```xmpfilter``` will fill in the result. Nice!
