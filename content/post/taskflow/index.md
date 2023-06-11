---
title: "Taskflow"
description: "About nodes, tasks and more"
date: 2023-06-10T04:30:21+01:00
draft: true
slug: "taskflow"
image: "cover"
categories:
    - Program flow
tags:
---

## Untangling the Mess

How do you write software where program flow is more complex than the underlying algorithms?
This is a question I've been asking myself for a while now. There is an inherent issue with
classical programming where a program becomes nearly unreadable in places where control flow
is non-linear, likely due to branching, vtables, lambdas or smart pointers.

I've never really had any major issues with this effect until I started working on
[CubiCAD](https://github.com/RedNicStone/CubiCAD), my first vulkan based engine. As it turns
out, even with the numerous layers of abstractions that vulkan provides, there are still a lot
of differences that are vendor, device and host specific. Our application needs to consider
these and handle different situations accordingly if you want good performance and cross-device
compatibility. In addition to that, vulkan objects often have non-trivial lifetimes and states
that need to be explicitly transitioned between. My solution to this at the time was to spam
c++ 11's shared pointers where I could as well as an extremely convoluted file that handled
program flow with hundreds of checks and conditional statements. Both were a pain to write and
maintain, so I was determined to come up with a better solution.

So to make sure I wouldn't run into this problem again before I revisited Vulkan, I spend some
time thinking about how to tackle this problem. It's been over a year now, and I believe to have
come up with a suitable solution. Just recently I started implementing those ideas as a library,
so I thought I'd share the concept and what I've got so far on this blog.


## Nodes, Edges and Graphs

While thinking about the problem, I came to the realization that pretty much every program flow
can be represented by a graph in some way. In that sense, every operation would be a node, every
piece of data an edge and every function its own separate (sub-)graph. I'm also not the first one
to think of code in that way, and you can find a lot of tools that represent programs in that way.
A number of visual programming languages come to mind like Unity's visual scripting, the shader
graph in Blender, Godot and Unreal, as well as countless others. They lower the bar of entry for
scripting by abstracting concepts and present it in a simpler and more digestible way. Even my
first ever programming experience was in a visual language, Lego's LSoft.

I personally love these visual languages. Granted, there are a lot of problems with them which we
need to address first to make such a system viable. Here are some of the more prominent concerns
of users that have tried these languages:
- Slow speed and memory consumption
- Little to no extensibility of the language
- Editors are often cumbersome to use
- Programs quickly get too complex to visually reinterpret
- Many programming techniques dont translate into a visual language


## What's Lightgraph?

I agree with most of these concerns and want to find a way to combat them. My newest project, 
[lightgraph](https://github.com/RedNicStone/lightgraph), aims to provide a viable graph-based 
alternative to describe program flow. It's a header only library (mainly due to the excessive
use of templates) writen in C++ 20. I have not made the repository public, but I do plan to do
so once the library reaches a usable state. It's intended to be more of a drop-in library for
projects built in c++ rather than its own language, however it's absolutely possible to build
a language around lightgraph. It is similar in some ways to 
[Taskflow](https://github.com/taskflow/taskflow) but provides a much expanded toolset and improved
performance. Let's go over some of the concepts lightgraph is based on.

### Graphs
These are the box standard graphs your probably used to from countless algorithms. They don't
enforce any restrictions themselves, but there are tools to check if they contain loops, cycles 
or other structures. There are a couple of different types which you can choose between depending
on your application. Depending on the type the graph may be read-only, append-only or fully editable.
The type also determines if it can only be traversed in edge direction or both ways. Types can be
converted between easily, and they are there to help optimize for memory usage and speed. For example
if you wanted to only load a graph, traverse it any not modify it, a compressed edge list
implementation is favorable over a linked-list based implementation. By default, a very efficient
page-based implementation is used which is a good compromise between speed, memory usage and editing
complexity.

### Graph attachments
Attachments are a way to associate data with a graph structure. In older versions, lightgraph stored
data interleaved within the graph structure, but this had some major drawbacks. With attachments
different objects may associate their own data with a common graph structure without interfering
or even having to know of the existence of each other's data. For example, a program that checks
whether a graph satisfies a number of requirements to be a DAG may need to associate an ID with
each node, which is as simple as creating a node attachment to store the IDs.

### Tasks and Task factories
These two concepts relate to the same overarching idea. Tasks factories describe a way to transform a
set of inputs into a set of outputs. Quick side note: Yes it's outputs, plural. Where in standard
programming  languages typically don't allow you to return multiple objects, but this is absolutely
allowed in graph based programming. Task factories have a fixed set of inputs and outputs and can
be instanced into tasks. This instantiation step is mainly there for thread safety in multithreaded
execution modes as well as to offer an easy way to register tasks to a global registry.

### Flows
The flow is a structure that is associated to a graph, similar to an attachment. Instead of storing
a user datatype however, it stores metadata information on the task flow. It assumes that every
node is a task and every edge is data. It stores what inputs connect to which outputs and what factories
can be used to instantiate a task. Flows can also act as a task factory themselves and therefor be 
used as a subgraph. This is very powerful and allows the users to split their implementation into multiple
sub-graphs, similar to what someone might do with functions in a classical language. Each flow also
associates with a queue and task type, which I will touch on next.

### Queues
A number of different queue implementations are provided, and each is compatible with one or more
node types. They aim to provide a method to traverse graphs and are mainly used to 'dispatch' flows.
Some of these queues run any function associated with the task, use code associated with the task to
generate programs that need compiling.
