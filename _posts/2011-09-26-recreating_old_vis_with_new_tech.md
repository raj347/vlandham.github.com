---
layout: post
title: Recreating Old Visualizations with New Technology
categories:
- vis
---

### Using d3 and coffeescript to reproduce old charts

Recently, I stumbled upon a set of statistical charts from the 1800-1900’s in the [David Rumsey Map Collection](http://www.davidrumsey.com/luna/servlet/view/search?QuickSearchA=QuickSearchA&q=statistics&sort=Pub_List_No_InitialSort%2CPub_Date%2CPub_List_No%2CSeries_No&search=Search) . While perhaps not the best forms of data visualization by today’s standards (pie charts, stacked bar charts, etc), they serve as beautiful inspirations visualization techniques due to their organization, attention to detail, color scheme, and presentation.

Here’s an example from the collection:

[![](images/vis/rumsey_stat_1.png)](http://www.davidrumsey.com/luna/servlet/detail/RUMSEY~8~1~32117~1151459:Proportion-of-foreign-born-of-each-?qvq=q:%3D%22U.S.%2BCensus%2BOffice%22%2BAND%2B%3D%22Washington%22;lc:RUMSEY~8~1&mi=144&trs=288)

Based on the mantra’s of [Enrico Bertini](http://fellinlovewithdata.com/guides/how-to-become-a-data-visualization-expert-a-recipe) on [Fell in Love with Data](http://fellinlovewithdata.com) and perhaps also the guidelines of creativity from [Everything is a Remix](http://www.everythingisaremix.info/shop-now-open/), I wanted to try to recreate (copy) a small selection of these charts using modern tools.

I choose to try out [d3.js](http://mbostock.github.com/d3/) for this experiment. This is a javascript framework that is the official successor to the popular [Protovis](http://mbostock.github.com/protovis/) visualization toolkit. d3 allows you to represent your data with native SVG elements, which provides a lot of power in terms of layout and style. Furthermore, to make writing javascript more fun, I wrote the visualizations in [coffeescript](http://jashkenas.github.com/coffee-script/) - a language that compiles down to javascript.

The process of recreating these graphs was a lot of fun and results are in [my visualization section](vis/index.html#old_vis_d3) . Here’s an image of my d3 version of the example above ([click](vis/nationality_by_city.html) for the actual d3 version):

[![](images/vis/rumsey_stat_1_recreate.png)](vis/nationality\_by\_city.html)

Though I only have a few examples (3 right now, but I hope to have time to make more), I believe it shows some of the potential power of d3 in terms of customizing exactly how you want your visualizations to look. In the future, I might look at adding functionality like interactions and transformations of the raw data - all of which should be pretty easy with d3.
