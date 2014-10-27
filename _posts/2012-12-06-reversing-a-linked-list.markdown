---
layout: post
title: "Reversing a Linked List"
categories: linked-list data-structures c
---

Over the last few months, I have been working with Ruby almost exclusively. At the same time, though, I miss using C. Ruby can be a little tricky at times, but it's nowhere as demanding as C. And I miss that. So when I read that reversing a linked list shows up among the various challenges presented to software developers in interviews, I decided that working through the challenge was a great opportunity to return to C.

For the sake of brevity, let's start by assuming that we have already created a singly linked list. There is a global header pointer ```gHeadPtr``` for the first node and a global tail pointer ```gTailPtr``` for the last node. Each node in the list holds a pointer ```next``` which points to the following node. The last node points to ```NULL```. Standard setup.

Note that the list is not a doubly linked list. In other words, each node has a ```next``` pointer, but not a ```previous``` pointer. The exercise becomes much easier with a ```previous``` pointer and is simply a matter of swapping ```next``` and ```previous```. In the case here, we're working with a list of nodes which only have a ```next``` pointer. That makes things a little more involved.

We start with a function to reverse the list. Before any operations, we check to make sure the list isn't empty or that it is longer than one node.

``` c
void ReverseList( void ) {
  if ( gHeadPtr == NULL ) {
    // List is empty
    printf( "Cannot reverse empty list!\n\n" );
    return;
  }

  if ( gHeadPtr->next == NULL ) {
    // list contains only one node, nothing to be done
    printf( "List contains only one node!\n\n" );
    return;
  }
  ...
}
```

Now comes the more involved bit. When moving through a linked list, it's common practice to introduce a third pointer in addition to the global pointers. The third pointer is effectively a marker as we move through the list. Since we're reversing the list, though, we'll need an additional pointer.

The strategy will be to move three pointers through the list. In front will be the ```forward_node```. Next will be our ```gHeadPtr```, which we'll gradually move to its new home at the end of the list. At the same time, we'll have a ```rear_node``` to keep track of where we have been. We first assign both pointers to point to the head pointer.

``` c
void ReverseList( void ) {
  ...
  struct node *rear_node, *forward_node;

  forward_node = gHeadPtr;
  rear_node = gHeadPtr;
  gTailPtr = gHeadPtr;
  ...
}
```

The tough part in manipulating a linked list is knowing how to keep track of the links between the nodes. It's fairly simple to move a pointer through a list of nodes, but when we're reassigning the ```next``` pointer, it's all too easy to lose track of a node in the process and break our entire list. By using a ```forward_node``` and a ```rear_node```, we will always have a reference to the nodes immediately before and after the moving ```gHeadPtr```.

The code is as follows:

``` c
void ReverseList( void ) {
  ...
  // repeat until no additional nodes remain
  while ( gHeadPtr->next != NULL ) {
    forward_node = gHeadPtr->next; // move forward_node past head pointer
    gHeadPtr->next = rear_node;    // reassign next pointer to rear pointer
    rear_node = gHeadPtr;          // move rear to current head
    gHeadPtr = forward_node;       // move head to forward_node
  }
  ...
}
```

Note that in the first time through the loop, when we assign the ```gHeadPtr->next``` to point to the ```rear_node```, we're basically having ```rear_node->next``` point to itself, ```rear_node```. This little detail also affects ```gTailPtr```, which is the same node. Since we won't be moving ```gTailPtr``` any further, we can wait to clean up this little mess after the loop.

First, we finalize the move by making sure ```gHeatPtr->next``` points to the position of ```rear_node```. And then finally, we fix that funny little pointer which pointed to itself and have it point to ```NULL``` to mark the list's end.

``` c
void ReverseList( void ) {
  ...
  // finish move
  gHeadPtr->next = rear_node;

  // terminate linked list
  gTailPtr->next = NULL;

  return;
}
```

And there's the algorithm to reverse a singly linked list. There remains the question of efficiency. And there are perhaps better ways. But it's always good to start with something that actually works first.
