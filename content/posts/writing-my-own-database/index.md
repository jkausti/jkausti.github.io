---
title: "Writing my own database management system"
draft: true
date: 2025-02-23T20:00:00.000Z
description: ""
categories:
  - Data Engineering
tags:
  - Data Modeling
  - Data Engineering
---

Outline:

* Intro
* Where do you even begin?
* Main components of a database management system
* Minimum viable product
* Conclusion


# Intro

 - Been working on my own database since January.
 - Inspired by youtube video of a guy writing one in 7 months. Very basic in terms
 of features, but still about 60000 lines of Rust code.
 - I've been working with databases for my whole career, so I thought it would
   be interesting to actually look into how the work under the hood.
 - I also recently started learning Zig, which is a systems programming language
   that is very similar to Rust, but with a focus on simplicity and performance.
 - There is a quite famous open source project named TigerBeetle, which is a
   financial transactions database written in Zig. They claim Zig is the systems
   programming language for the next 30 years, so I thought if I manage to write
   a database in Zig, I can also say that I really know the language.

# Where do you even begin?

 - The youtube video I mentioned earlier mentioned some resources that he had 
 used to get a better understanding of how databases work.
   - The Carnegie Mellon University youtube lectures are very good source.
   - The github repo of the guy that did the youtube video have extremely
     good explanations of the concepts, even with some nice Ascii drawings of
     slotted page architecture and various memory buffers. 
 - An important component that I though is a good place to start is the storage
   engine. The storage engine handles the communication between memory and disk,
   i.e. it reads and writes pages to disk.
 - If I can implement a basic version of the slotted page architecture including
 serialization and deserialization of the pages, I think I have a good starting
 point.

# Main components of a database management system
 - After that I think I'll move to the other end, i.e. implementing a REPL that the
   user can use to interact with the database through SQL.
 - Then I (think I) need a SQL parser, query planner to interpret the SQL to
   actual instructions. Then I also need some kind of execution engine that
   takes those instructions and translates them into storage engine
   instructions, i.e. "fetch these rows from this
   page".

# What features do I want?

 - To be honest, i'm not sure yet.
 - At least, I want to be able to create tables, insert rows, and select columns.
   - I think that when I get the hang of how to select things, then I will have
     a better understanding of how much work other
     features such as where-clauses and order-by clauses will require. 
 - I also want the concept of schemas.
 - I also feel like I need to implement b-tree indexing. Since it's one of the
   most crucial part of a relational database. From what I've studied so far,
   the self-balancing algorithm is quite complex so let's see.

# What do I want to get out of this?

 - Honestly, I'm not sure if I will ever have a job that requires me to write a
   database from scratch and I actually do it because I think it's fun.
 - Some people like woodworking, others gardening. I happen to like programming.
   I actually think it's a lot like woodworking, except you have unlimited raw
   materials at your disposal.
 - I also do it to learn Zig. I recently read "The Pragmatic Programmer" and one
   of the tips was to learn a new programming language every year. I think that's
   a good tip, because it forces you to think about programming in a different
   way.



## Introduction

Since January, I’ve been working on building my own database from scratch. The
idea came to me after watching a [YouTube
video](https://www.youtube.com/watch?v=5Pc18ge9ohI) of someone who built a basic
database all by themselves. While the project was relatively simple in terms of
features, it took 7 months for the guy to write it and consisted of about 60,000
lines of Rust code — a significant undertaking.

Having worked with databases throughout my career, I was always intrigued by how
they function under the hood. I decided it was time to dive in and explore
database internals firsthand. I've also recently started learning Zig,
a systems programming language that shares similarities with Rust but emphasizes
simplicity and performance.

Another source of inspiration is an open-source database named TigerBeetle, a
financial transactions database written in Zig. The developers behind it claim
that Zig is the systems programming language of the next 30 years. That piqued
my interest—if I could build a database in Zig, I’d gain not only a deep
understanding of databases but also a solid grasp of Zig itself.

## Where Do I Even Begin?

The YouTube video I watched provided several useful resources for understanding
database design. Two stood out in particular:

* [Carnegie Mellon University’s database lectures](https://www.youtube.com/watch?v=otE2WvX3XdQ&list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq): These offer a wealth of
knowledge on database internals.

* [This Github repo](https://github.com/antoniosarosi/mkdb) from the guy who
  made the youtube video: His repo includes detailed explanations of the
  concepts implemented, complete with ASCII drawings illustrating slotted page
  architecture, b-trees and other important concepts. I also know Rust just well
  enough to be able to read the code so I'm pretty sure it will be a great
  inspiration.

* __LLMs__: Not using LLMs to help with this task would be a pretty stubborn
  approach. They seem to have a pretty good understanding of database internals
  so I will be using them to get the actual implementation details right. They
  are not the best at writing Zig, since it's a pretty new language, but I think
  it will not be an issue. They halucinate a lot even when I write Python so
  I'm used to working around that.

One of the first components I decided to tackle was the storage engine. The
storage engine manages the communication between memory and disk, handling the
reading and writing of data pages. My first goal is to implement a basic
slotted page architecture, including serialization and deserialization of pages.
Achieving this would give me a strong foundation to build upon.

The first challenge that I encountered when starting was to understand how I
even write a data structure to disk and then read it back into memory. I need
some kind of file format to be able to write and read data to disk. And what even is
a file format? LLMs where pretty good at resolving this issue, and I might write
a more detailed post about it at some point, but the short answer is that a
file format is just bytes written to disk in a certain order. If you know the
order, you can also create a deserializer that reads the bytes back into memory and
recreates the structs that you wrote to disk. Pretty simple when you think about
it, but it's the kind of things that you really never have to do yourself unless
you are writing low-level stuff.

After resolving the initial hurdles, I feel it's going pretty well. I've managed
to implement a basic version of a slotted Page as well as a serializer and
deserializer for the page and related data structures. Once the storage engine
is semi-complete, I think I will jump to the other end and implement a REPL
(Read-Eval-Print Loop). This will allow users to interact with the database
using SQL commands.

After that, I’ll need to implement a SQL parser and query planner. The parser
will interpret SQL commands, while the query planner will convert them into
executable instructions. Finally, an execution engine will process those
instructions, fetching the necessary rows from disk through the storage engine.

This is my current high level understanding of what I need to do, but it may be
that I need to adjust my plan as I go along.

## What Features Do I Want?

To be honest, I haven’t settled on a complete feature set yet. However, there
are a few key functionalities I definitely want to implement:

* __Basic table operations__: Creating tables, inserting rows, and selecting
  columns.

* __Schema support__: A structured way to organize data.

* __B-tree indexing__: Since indexing is a crucial part of relational databases,
  I feel I should implement B-tree indexing. From my research so far, the
  self-balancing algorithm for B-trees is complex, so this will be an
  interesting challenge.

## What Do I Want to Get Out of This?

Will I ever need to build a database from scratch for a job? Probably not. But
I’m doing this because I enjoy it.

Some people take up woodworking or gardening as a hobby — I happen to enjoy
programming. In many ways, I see programming as similar to woodworking, except
that I have an infinite supply of raw materials (code, online learning material,
a computer) at my disposal. Time is actually the only truly limiting factor
here, but I'm not in a rush. My goal is to have fun and learn something new.

Another big motivation is learning Zig. I recently read The Pragmatic
Programmer, which suggests learning a new programming language every year. I
think that’s excellent advice, as it forces you to rethink how you approach
software development. I've been working with Python for many years not without
really making an effort to learn something else, so it's about time. One thing
in Zig that is truly different from Python is how you think about memory. In
Zig, you need to allocate memory yourself. And to do that successfully, you need
to truly understand the concepts of the stack and the heap, things I kind of
knew about from before but never actually understood in practice.

This project is my way of putting that idea into practice. Whether or not it
leads to something useful, I know I’ll come out of it with a deeper
understanding of databases and a new programming language, so it won't be wasted
time.
