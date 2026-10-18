---
title: Reviewing HTB Introduction to Python
date: 2026-10-18 09:00:00 -0500
categories: [Reviews on HTB]
tags: [Python, HTB]
description: My thoughts on Hack The Box's Introduction to Python module.
image:
  path: /assets/img/Reviews_on_HTB/10182026_python_thumbnail.png
  alt: 
---

## My Thoughts
Python is a high-level language that's great for beginners, since you don't have to worry about defining data types or dealing with pointers. If anything, the biggest adjustment is indentation, which Python uses to define blocks of code. HTB does an amazing job going over Python in the context of real-world scripting and security use. The module skips over a lot of typical beginner topics and jumps straight into control structures, data structures, file operations, loops and more. Although I believe this is great, a beginner may think otherwise; new programmers may be unfamiliar with the concepts. That being said, I'd'recommend having some previous experience with programming in order to feel comfortable and truly understand the content in the module. If you feel this is the right place to start your Python journey, check out the link below. If not, keep reading the rest of this post to learn more about the module and consider starting with a more basic course first.   


HTB Academy Introduction to Python: <https://academy.hackthebox.com/app/module/477>
![HTB ACADEMY IMAGE](assets/img/Reviews_on_HTB/10182026_python_image_fading.png)


## Operators
In the area of python operators, the module speed runs the subject by briefly mentioning how operators can be used in various ways. Areas covered include:
- Arithmetic
- Assignment
- Comparison
- Logical
- Bitwise
- Identity
- Membership

Although the module doesn't spend much time on each operator, it does just enough to allow the reader to gain a high-level understanding with only a few examples. My only recommendation is to try each of the examples on your own system; typing the code on your own and adding on to what was already built. By adding on, you force yourself to understand the order of code execution while learning the syntax for the language. Without repeating the module's lessons, here are quick-reference tables of each operator type: 


### Arithmetic

| Operator | Name           | Example  |
| -------- | -------------- | -------- |
| `+`      | Addition       | `x + y`  |
| `-`      | Subtraction    | `x - y`  |
| `*`      | Multiplication | `x * y`  |
| `/`      | True Division  | `x / y`  |
| `%`      | Modulus        | `x % y`  |
| `**`     | Exponentiation | `x ** y` |
| `//`     | Floor Division | `x // y` |

### Assignment

| Operator | Name                       |
| -------- | -------------------------- |
| `=`      | Assign                     |
| `+=`     | Assign with addition       |
| `-=`     | Assign with subtraction    |
| `*=`     | Assign with multiplication |
| `/=`     | Assign with division       |
| `%=`     | Assign with modulus        |
| `**=`    | Assign with exponentiation |
| `//=`    | Assign with floor division |

### Comparison

| Operator | Name             |
| -------- | ---------------- |
| `==`     | Equal            |
| `!=`     | Not equal        |
| `>`      | Greater than     |
| `<`      | Less than        |
| `>=`     | Greater or equal |
| `<=`     | Less or equal    |

### Logical

| Operator |
| -------- |
| `and`    |
| `or`     |
| `not`    |

### Bitwise

| Operator | Name        |
| -------- | ----------- |
| `&`      | And         |
| \|       | Or          |
| `^`      | Xor         |
| `~`      | Not         |
| `<<`     | Shift left  |
| `>>`     | Shift right |

### Identity

| Operator |
| -------- |
| `is`     |
| `is not` |

### Membership

| Operator |
| -------- |
| `in`     |
| `not in` |


## Control Structures, Loops, Functions, and Classes
HTB does a phenomenal job going over the foundation of if statements, loops, functions, and classes with their examples and use cases. Rather than simply showing the syntax, they go into detail as to why these contructs can be used and when to use them. In my own experience, many courses that I have completed fail to teach proper use cases for subjects, whick keeps students from fully grasping what is being taught. Examples used within this area mention file operations, working with and iterating through data structures, with the addition of making code portable and reusable.

## Data Structures 
Regarding data structures, the module covers many topics in a good amount of detail. I perhaps spent most of my time in this section as I wanted to understand the differences between them all. Unlike the earlier sections, not many real-world use cases were given for this area. Though, HTB makes up for it by demonstrating numerous examples of data structures paired with loops and functions. This let me come up with my own ideas and translate them into code. On the other hand, I'd recommend doing some outside research regarding each data structure as some of the explanations are not exactly straightforward. I'd strongly encourage modifying each sample code given to not only to be hands-on, but also to get more in tune with how data is actually stored within a program. 

| Data Structure       | Description                                                                    | Ordered / Indexed?                                                   |
| -------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| Array                | Stores items of the same type, usually numbers.                                | Yes, each element has an index.                                      |
| List                 | Stores items of any type, can grow or shrink.                                  | Yes, each element has an index.                                      |
| Tuple                | Immutable sequence that can store items of different types.                    | Yes, each element has an index.                                      |
| Dictionary           | Associative array of key-value pairs, implemented as a hash table.             | No, accessed by key (insertion order is preserved since Python 3.7). |
| Set                  | Unordered collection of unique items, implemented as a hash table.             | No, elements have no index.                                          |
| Graph (not built in) | Nodes connected by edges, usually built as a dict of lists or with `networkx`. | No                                                                   |
| Tree (not built in)  | Hierarchical nodes and edges, built with classes.                              | No, unless array-based, like a heap (`heapq`).                       |

## Encapsulation 
From the entire module, the example that caught my attention most was encapsulation. I'll admit, encapsulation is a bit new to me and it wasn't a topic I understood right away. The module describes it as, "Holds together the state and behavior of the object, hiding its state/behavior from the outside via private instance variables and methods." After studying HTB's example a bit longer and adding my own line of code, I found that this "hiding" isn't enforced in Python [for private attributes and methods alike]. Inside the class code below, Python quietly renames "__maxprice" to "_Computer__maxprice" [a process commonly referred to as name mangling], allowing any code that knows the new name to directly change the attribute without the setter. From a security standpoint, this matters. If a program relied on a "private" attribute to enforce something like a price limit on the user's side (client-side validation), the user could simply change it. In Python, private attributes signal the developer's intent; real enforcement has to happen somewhere the user can't reach, such as a server.

```python
class Computer:

    def __init__(self):
        self.__maxprice = 900

    def sell(self):
        print(f"Selling Price: {(self.__maxprice)}")

    def setMaxPrice(self, price):
        self.__maxprice = price

c = Computer()
c.sell()

# Change the price
c.__maxprice = 2200         # Will not work if you try to change the object directly BUT it will make a NEW attribute named "__maxprice"
c.sell()                    # Appears to have failed
print(c.__dict__)           # Proves it created a new unrelated attribute


# My added code: bypass the "private" attribute using its mangled name
c._Computer__maxprice = 100 #  Change the original (mangled) attribute directly
c.sell()

# Using setter function
c.setMaxPrice(1100)
c.sell()

"""
Selling Price: 900
Selling Price: 900
{'_Computer__maxprice': 900, '__maxprice': 2200}
Selling Price: 100
Selling Price: 1100
"""
```
  {: file="Encapsulation.py"}
