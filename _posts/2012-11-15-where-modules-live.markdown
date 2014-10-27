---
layout: post
title: "Where Modules Live"
categories: ruby modules
---

Ruby modules have been on my mind frequently of late. While exploring the Ruby class hierarchy in ```irb```, I began to wonder where Ruby stores all its built-in and loaded modules. 

After lots of fruitless digging around, I finally found them! All modules exist as constants defined on ```Object```. Calling ```Object.constants``` will reveal all built-in modules, all loaded modules, and of course other constants defined on ```Object```.

This is probably common knowledge among Ruby ninjas, but it was a great source of excitment for me to find them all the same.
