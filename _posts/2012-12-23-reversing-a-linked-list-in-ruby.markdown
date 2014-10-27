---
layout: post
title: "Reversing a Linked List in Ruby"
categories: ruby data-structures linked-list
---

Feeling pretty good about my last exercise with C and reversing a linked list, I showed my code to a C veteran. Long story short, the old school guys know a ton of good stuff. Although the algorithm from my last post worked, it was most definitely not the best way to solve the problem. As a follow up, I decided to refactor my code and switch languages, this time using Ruby.

Aside from the various details involved in moving from a procedural to an object-oriented paradigm, the biggest change is how I've handled common list operations. The aforesaid C veteran advised me to write three simple functions: one for 1) putting an entry at the top of the list, 2) putting an entry at the bottom of the list, and 3) removing an entry from the top of the list. In effect, the idea is to implement common stack and queue behavior. The result makes manipulating the list much easier.

In what follows, I'm going to walk through my current implementation of a linked list in Ruby. Then, I'll explain how to reverse that list both destructively and non-destructively.

First, we start with code for a list entry.

``` ruby
class Entry
  attr_accessor :next, :data

  def initialize(data)
    @next = nil
    @data = data
  end
end
```

The ```next``` variable will be a pointer to the following entry in the list. For demonstration purposes, I've also included a ```data``` variable, which as the name suggests holds data within each entry. We won't concern ourselves with the data aspect of the linked list for now.

Our list will take the form of a class called ```List```.

``` ruby
class List
  attr_accessor :name

  def initialize
    @head = nil
    @tail = nil
  end
end
```

For convenience sake, I've included a variable to keep track of the list's name. More importantly, though, we start off with a head and tail pointer set to ```nil```. Unlike my last implementation which exposed the head and tail pointers, this time we'll make ```@head``` and ```@tail``` accessible only to other methods within the class. In other words, there will not be a setter or a getter for either variable.

Now that we have a basic setup, the next step is to add the three methods named above: 1) adding to the top of a list, 2) adding to the bottom of the list, and 3) removing from the top of a list. There are likely more idiomatic names in Ruby for these methods, but here I choose to follow the C veteran and use ```ptq``` ("put on top of queue"), ```pbq``` ("put on bottom of queue"), and ```rtq``` ("remove from top of queue").

We'll start with ```ptq```.

``` ruby
def ptq(entry)
  if @head.nil?
    @head = entry
    @tail = entry
  else
    entry.next = @head
    @head = entry
  end
end
```

The implementation for ```ptq``` is fairly straightforward. We have two states to consider: 1) the head pointer is set to ```nil```, or 2) the head pointer holds an entry. In the first case, we simply assign our new ```entry``` to ```@head``` and ```@tail```. Because the ```new``` method on ```Entry``` has already assigned ```entry.next``` to ```nil```, we have nothing more to do in this case. The linked list is terminated with the present entry.

In the second case, i.e., we already have some number of entries in the list, we first have our new entry to point to the list's current head. The next step is simply to move the ```@head``` pointer to point to the new entry. Now we have a working ```ptq```.

The implementation of ```pbq``` follows much the same logic.

``` ruby
def pbq(entry)
  if @head.nil?
    @head = entry
    @tail = entry
  else
    @tail.next = entry
    @tail = entry
  end
end
```

First, we deal with the situation when there isn't an entry pointed to by ```@head```, just as above. In the second situation, i.e., a number of entries already exist in the list, we simply have ```@tail.next``` point to our new entry, and then reassign ```@tail``` to point to the new entry. Done.

Before getting to our destructive and non-destructive reverse methods, there is one more important function to deal with: ```rtq```. The function here will remove an entry from the top of the list and return the entry.

``` ruby
def rtq
  return nil if @head.nil?
  entry = @head
  @head = @head.next
  return entry
end
```

If nothing exists in the list, we will return ```nil```. Otherwise, we grab ahold of the head entry, set ```@head``` to point to the next entry in the list, and then return the old head entry.

Now that we have the means to build a linked list, we're ready to implement a non-destructive and destructive reverse method. The destructive method is the more challenging one, so let's start with that.

Following the Ruby idiom, we'll name the method ```reverse!```. The basic idea behind ```reverse!``` is that we'll pop all the entries of the current list and put them on a stack. Once we have all the entries on the stack, we'll just assign ```@head``` to the top entry on the stack, which will have been the old ```@tail```.

``` ruby
def reverse!
  return if @head.nil?

  @tmp_head = self.rtq
  @tmp_head.next = nil
  @tail = @tmp_head

  until @head.nil?
    entry = self.rtq
    entry.next = @tmp_head
    @tmp_head = entry
  end

  @head = @tmp_head
end
```

If the ```reverse!``` method is called on an empty list, we'll return ```nil```. Otherwise, we start by popping the top entry off the current list and assigning it to ```@tmp_head```. We then reassign the ```next``` pointer of that entry to ```nil```. This ```next``` pointer will terminate the reversed list and so we assign ```@tail``` to the same entry.

The next ```until``` loop is at the heart of the ```reverse!``` method. We'll pop an entry off the current list and assign its ```next``` pointer to the entry in ```@tmp_head```. Each time through the loop ```@tmp_head``` will point to the entry which used to precede the current entry. By reassigning ```next``` to ```@tmp_head```, we're basically reversing the thread of the linked list. Once we have that thread reversed between two entries, we move ```@tmp_head``` in preparation for the next time through the loop.

The loop will continue until we empty the list. Once that happens, we only have to reassign ```@head``` to the entry in ```@tmp_head```. Now we've destructively reversed the linked list.

Reversing the linked list non-destructively is much simpler by comparison, especially since we've implemented ```ptq``` above. We'll create a new list, iterate over the old one, while pushing each entry onto the top of the new list. The result will be a new list, with the order of entries reversed. Assigning a sequence of numbers to the ```data``` attribute of ```Entry``` above is a good way to confirm that ```reverse``` does in fact work.

Before we can proceed, though, we need to implement the ```each``` method in our ```List``` class. We start by including ```Enumerable``` within our class.

``` ruby
include Enumerable

def each
  return nil if @head.nil?
  entry = @head
  until entry.nil?
    yield entry
    entry = entry.next
  end
end
```

After making sure we don't have an empty list, the ```each``` method starts with ```@head``` and spits out an ```entry```. Then, it moves to the next ```entry``` and repeats. Once we've reached ```nil```, we're at the end of the list, and ```each``` is done with its enumeration.

We're almost done. The non-destructive reverse is as follows:

``` ruby
def reverse
  new_list = List.new
  self.each { |entry| new_list.ptq(Entry.new(entry.data)) }
  return new_list
end
```

We make a new list, copy all the data in the old list to the new, and then return the new list. Finally, we're done.

After having done this exercise twice, once in C and now once in Ruby, it's perfectly clear that functions like ```ptq```, ```pbq```, and ```rtq``` are what makes the process dramatically easier. Rather than trying to do gymnastics with pointers, it's much simpler to implement stack-like and queue-like behavior. Reversing a linked list becomes simply a manipulation of those basic functions.


