---
layout: post
title:  "Building Faceted Search From Scratch - Part 1"
date:   2014-05-16 22:00
categories: elasticsearch 
---

Part 1 - The front end
---

This is the first part of my attempt to build a faceted product search from scratch using angular and elasticsearch.

I want to start this series from a front end point of view. This gives me the ability to think about the requirements 
of the system.

But first of all what is a faceted search. According to [wikipedia](http://en.wikipedia.org/wiki/Faceted_search), 
a faceted search or faceted navigation  

> allows users to explore a collection of information by applying multiple filters. 

Faceted search is used in almost every online shop these days. In many cases it helps narrow down results 
of a full-text search.
 
This is the user-story I made up:

As a customer I want to find articles based on

- Product name
- Brand
- Color
- Size
- Prize range

After defining the search parameters I want to see a list of results. I don't want to scroll long list. If there are more than a bunch of results, I want to see the best matches first. I want to page through more results and I want to see how many results there a in total.

Here is a rough mock-up of the idea.

![Mockup]({{ site.url }}/assets/media/Mockup%20Faceted%20Search.png)

From here on
