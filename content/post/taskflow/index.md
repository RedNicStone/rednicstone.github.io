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

