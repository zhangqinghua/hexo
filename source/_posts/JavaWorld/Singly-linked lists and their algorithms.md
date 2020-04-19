---
title: Singly-linked lists and their algorithms

tag:
- Data structures and algorithms in Java

categories:
- JavaWorld

mathjax: true

date: 2020-04-06 00:00:05
---
Like arrays, which were introduced in Part 3 of this tutorial series, linked lists are a fundamental data structures category upon which more complex data structures can be based. Unlike a sequence of elements, however, a linked list is a sequence of nodes, where each node is linked to the privious and next node in the sequence. Recall that a node is an object created from a self-referential class, and a self-referential class has at least one field whose reference type is the class name. Nodes in a linked list are linked via a node reference. Here's an example:
```java
class Employee
{
   private int empno;
   private String name;
   private double salary;
   public Employee next;
   // Other members.
}
```

In this example, Employee is a self-referential class because its next field has type Employee. This field is an example of a link field because it can store a reference to another object of its class -- in this case another Employee object.

This tutorial introduces the ins and outs of singly linked lists in Java programming. You'll learn operations for creating a singly linked list, inserting nodes into a singly liked list, delting nodes from a singly liked list, concatenating a singly linked list to another singly linked list, and inverting a singly linked list. We'll also explore algorithms most commonly used for sorting singly linked llists, and conclude with an example demonstrating the Insertion Sort algorithm.

## What is a singly linked list?
A singly linked list is a linked list of nodes where each node has a single link field. In this data structure, a reference variable contains a reference to the first (or top) node; each node (except for the last or bottom node) links to the next one; and the last node's link field contains the null reference to signify the list's end. Although the reference variable is commonly named top, you can choose any name you want.

Figure 1 presents a singly linked list with three nodes.

![Figure 1. A singly linked list where top references the A node, A connects to B, B connects to C, and C is the final node](001.jpg)

Below is pseudocode for a singly linked list.

```
DECLARE CLASS Node
  DECLARE STRING name
  DECLARE Node next
END DECLARE
DECLARE Node top = NULL
```
Node is self-referential class with a name data field and a next link field. top is a reference variable of type Node that holds a reference to the first Node obejct in a singly linked lsit. Because the list doesn't yet exist, top's initial value is NULL.

## Creating a singly linked list in Java
You create a singly linked list by attaching a single Node object. The following pseudocode creates a Node object, assigns its refernce to top, initializes its data filed, and assigns NULL to its link field:
```
top = NEW Node
top.name = "A"
top.next = NULL
```

Figure 2 shows the initial singly linked list that emerges from this pseudocode.

![Figure 2. The initial singly linked list consits of a single Node (A)](002.jpg)

This operation has a time complexity of $O(1)$ -- constant. Recall that $O(1)$ is pronounced "Big of Oh of 1". (See Part 1 for a reminder of how time and space complexity measurements) are used to evaluate data structures.

## Inserting node into a singly linked list
Inserting a node into a singly linked list is somewhat more complicated than creaing a singly linked list becuase there are three cases to consider:
- Insertion before the first node
- Insertion after the also node
- Insertion between two nodes

### Insertion before the first node
A new node is inserted before the first node by assgning the top node's reference to the new code's link field adn assigning the new node's reference to the top variable. This operation is demonstrated by the following pseudocode:
```
DECLARE Node temp
temp = NEW Node
temp.name = "B"
temp.next = top
top = temp
```

The resulting two-Node list appears in Figure 3.

![Figure 3. The expanded two-Node singly linked list palces Node B ahead of Node A](003.jpg)

The operation has a time-complexty of $O(1)$.

### Insertion after the last node
A new node is inserted after the last node by assigning null to the new node's link filed, traversing the singly linked list to find the last node, and assigning the new node's reference to the last node's link field, as the following pseudocode demonstrates:
```
temp = NEW Node
temp.name = "C"
temp.next = NULL
DECLARE Node temp2
temp2 = top 
// We assume top (and temp2) are not NULL 
// because of the previous pseudocode.
WHILE temp2.next NE NULL
   temp2 = temp2.next
END WHILE
// temp2 now references the last node.
temp2.next = temp
```

Figure 4 reveals the list following the insertion of Node C after Node A.

![Figure 4. Node C comes last in the expanded three-node singly linked list](004.jpg)

This operation has a time complexity of $O(n)$ -- linear. Its time complexity could be improved to $O(1)$ by maintaining a reference to the last node. In that case it wouldn't be necessary to search for the last node.

### Insertion between two nodes
Inserting a node between two nodes is the most complex case. You insert a new node between two nodes by traversing the list to find the node that comes before the new node, assigning the reference in the found node's link field to the new node's link field, and assigning the new ndoe's reference to the found node's link field. The following pseudocode demonstrates these tasks:
```
temp = NEW Node
temp.name = "D"
temp2 = top 
// We assume that the newly created Node inserts after Node 
// A and that Node A exists. In the real world, there is no 
// guarantee that any Node exists, so we would need to check 
// for temp2 containing NULL in both the WHILE loop's header 
// and after the WHILE loop completes.
WHILE temp2.name NE "A"
   temp2 = temp2.next
END WHILE
// temp2 now references Node A.
temp.next = temp2.next
temp2.next = temp
```

Figure 5 presents the list following the insertion of Node D between Nodes A and C.

![Figure 5. The ever-growing singly linked list places Node D between Nodes A and C](005.jpg)