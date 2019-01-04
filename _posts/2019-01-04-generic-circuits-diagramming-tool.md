---
layout: post
published: false
author: Alexandre Dumont
last_modified_at: '2019-01-04 11:32 +0100'
title: Generic circuits diagramming tool
---
I love to draw diagrams and create presentations of what I do. I think they are a great way to document and show what you've done or what you plan to do, not only to others, but first of all for yourself.

In electronics for example, I have the need to draw circuits, representing all the components I'll use and how they are connected together.

Similary, I enjoy designing harware in Verilog HDL, and again, when complexity arise, I need to decompose the components in modules, and represent how they are all connected together.

Usually those interconnected modules will have multiple ports (or pins): input/output pins, clock pins, power and ground pins.

There are lots of specialized tools existing that offer libraries with existing components one can select, place on a layout and simply connect together. The problem with those is that you depend on the availability of all the components of your design in the existing libraries. Some of those tools also usually can be extended, so advanced users can create new components and libraries, but this is out of the scope of what I pretend to do.

When we are speaking of designing RTL modules and sub-modules in Verilog, and combining those modules together, you cannot rely on libraries of existing components.

I expect expensive (and licensed) tools from FPGA vendors to offer this kind of diagramming functionalities, but as an hobbyist, I use primarily free opensource tools, and to date I haven't found these functionalities in the FPGA opensource tools.

What I have found

## A New Post

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
