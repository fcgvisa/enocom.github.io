---
layout: post
title: "Making a Ruby Script Executable"
categories: bash ruby command-line
---

It's common knowledge in the *nix community, but for many new developers turning a Ruby script into an executable command line program is akin to magic. While there are other references on the internet, for the post here, I will briefly explain how to go from running a Ruby script by invoking Ruby directly, to running the script by its name alone.

We will start by assuming we have a simple Ruby script which prints "hello" on the command line. Our script's name will be ```greeter.rb```. The file holds one line of Ruby code:

``` ruby
puts "Hello!"
```

To run the script, we must type ```ruby greeter.rb```. Wouldn't it be nice to just type ```greeter``` instead and still get the script to run? Yes, it would.

First, we need to tell Bash what to do with our file since we won't be passing the script to Ruby directly. To do that, we add the following to the very top of our script:

``` ruby
#!/usr/bin/env ruby
puts "Hello!"
```

The first line is a Bash directive and basically tells Bash what program to run our file with by asking for the current configured version of Ruby as specified by the ```env``` command. For more on how ```env``` works, try typing ```man env``` into the command line.

Second, we need to make our script executable, which requires changing the file permissions. If the concept of file permissions is new, read about it [here](http://en.wikipedia.org/wiki/File_permissions#Traditional_Unix_permissions). Bascially, files have three types of permissions. They can be read, written, and executed. Most files typically start out as only having read and write access. Since we want to execute our script, we're going to have to grant it execute permissions.

Doing that is just a simple Bash command. On the command line, navigate to the directory holding the ```greeter.rb``` file. Now, to check the permissions, run:

``` bash
ls -l greeter.rb
```

The output will look something like this:

``` bash
-rw-r--r--    1 username  staff   13 Feb 16  21:10 greeter.rb
```

Your own username will show up in the place of ```username```, and the creation date will naturally be different, but otherwise the output will be almost identical. The first part of the line is the revelant part. The letters ```r``` and ```w``` specify read and write permissions.

We're going to add execute permissions which will appear as an ```x``` in that line. To add execute permissions, run the following command.

``` bash
chmod 755 greeter.rb
```

Now, if you check the file permissions again with ```ls -l greeter.rb```, the output should be a little different.

``` bash
-rwxr-xr-x  1 username  staff     13 Feb 16 21:20 greeter.rb
```

The presence of ```x``` indicates that the file can be run directly without calling Ruby first. The following command should get our file to say "hello."

``` bash
./greeter.rb
```

Almost there. Now, we just need to get rid of the prefix ```./```, which tells Bash where to look for ```greeter.rb```, i.e., in the current directory. Before we complete this last step, though, let's rename our file to just ```greeter```.

``` bash
mv greeter.rb greeter
```

Now, for the last step. Everytime we call a Bash program, e.g., ```ls```, ```chmod```, ```mv```, etc., Bash searches through a predefined list of folders looking for those programs. This is called the path. To see what the path is set to on your computer, try:

``` bash
echo $PATH
```

The output should be a long string of various system-critical folders. We need to put our application into one of these folders. Traditionally, it's best to leave folders like ```/usr/bin/``` and ```/bin/``` alone. Instead, any kind of user additions should be placed in ```/usr/local/bin/```. If that folder doesn't exist, create it with:

``` bash
mkdir -p /usr/local/bin/
```

Now, we can either move our ```greeter``` into that folder, or leave the application where it is and just create a softlink (or an alias in OS X terms) within the ```/usr/local/bin/``` folder. To create an alias, we'll use the ```ln``` command. From the directory where ```greeter``` lives, type:

``` bash
ln -s $PWD/greeter /usr/local/bin/
```

Note that the `$PWD` variable will expand to an absolute path to our greeter script. Now, we're done and we can simply type ```greeter``` to invoke our Ruby script!

As a footnote, if any of the above Bash commands seem confusing, trying looking up their ```man``` page by typing ```man <command>```.
