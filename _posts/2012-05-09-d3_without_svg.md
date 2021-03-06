---
layout: html5_post
title: D3 Without SVG
css: d3_no_svg.css
categories:
- tutorial
---
<script src="https://code.jquery.com/jquery-1.12.1.min.js">
<script type="text/javascript" src="vis/js/libs/es5-shim.min.js">
</script>
<script type="text/javascript" src="vis/js/libs/d3.v2.js">
</script>
<script type="text/javascript" src="vis/js/d3_no_svg.js">
</script>

Typically, when creating interactive visualizations with d3.js, you are building your graphics up in SVG. While much of the [D3 library](http://d3js.org/) is platform-independent, and there is even a [small example using just html](http://mbostock.github.com/d3/tutorial/bar-1.html) in the D3 tutorials, SVG is the typically used for all but the simplest D3 experiments.

Recently, the New York Times published an interactive visualization on the [electorial map](http://elections.nytimes.com/2012/electoral-map) for the up-coming American presidential election. It features a [bubble chart](http://vallandingham.me/bubble_charts_in_d3.html) inspired interactive ‘game’ in which users can make their own predictions about which way undecided states will swing to see how the election might turn out.

I think its a great piece. One detail that isn’t immediately apparent until you start looking at the code is that the entire visualization is built with **no SVG**. This means that the entire visualization works great in both modern browsers and old ones that don’t support SVG like **IE8** (I tested it and its just a little less smooth then in Chrome, but otherwise fine).

Here I’d like to look at just this one facet of how this visualization was made: how they use D3 without SVG.

SVG-less D3 Demo
----------------

To explore how the NYT group created their visualization, I’ve created a small demo using the same methods they did. The result is below:

<div id="canvas">
</div>
<br/>
<a href="#" id="move"><strong>Move</strong></a>

Overview
--------

[Here is the code for this visualization on github](https://github.com/vlandham/vlandham.github.com/blob/master/vis/js/d3_no_svg.js)

[And the corresponding CSS](https://github.com/vlandham/vlandham.github.com/blob/master/css/pages/d3_no_svg.css)

Before looking at the implementation details, let’s get a high-level overview of how this works:

First we create a *class* that represents a bubble in the vis. This class is called `SimpleBubble`. It stores the data as well as the visual representation for that data. Here, the visual component is a series of `divs` that are styled using css.

By *class* I really mean that it is a constructor function with additional methods added to it’s [prototype object](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Object/prototype) . In javascript, there are no classes, but an object inherits properties from it’s prototype object. And functions in javascript are [really Function objects](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function) . So a function has a prototype object to inherit functionality from. By defining ‘methods’ at the prototype level, we can instantiate instances of the function object (in this case, the bubbles in the graph), and they retain these methods. There are lots of posts online that explain javascript prototypes more. [Here is one about OO in JS](http://mckoss.com/jscript/object.htm) .

Next we create a class that represents the entire visualization. This `SimpleVis` class is handed the data we want to visualize, and the name of the `DOM` element to build the visualization at. It then instantiates `SimpleBubble` objects to store and visualize this data. The D3 [force layout](https://github.com/mbostock/d3/wiki/Force-Layout) is used to position/move the bubbles and is kept at the `SimpleVis` level.

Sound good? Ok, lets look at the nitty-gritty of how this works.

Useful jQuery Methods
---------------------

This visualization depends a lot on jQuery to make things easier. Before we look at the demo code, it might be useful to explain the jQuery methods used.

### [jQuery.proxy()](http://api.jquery.com/jQuery.proxy/)

The `proxy` method wraps a function in a particular [context](https://developer.mozilla.org/en/JavaScript/Reference/Operators/this) . This is very similar to the fat arrow `=>` in [coffeescript](http://coffeescript.org/#fat_arrow) . In fact, the class-based approach used in this demo would make it a good candidate for implementation in coffeescript. However, the NYT visualization used the prototype approach, so I wanted to try it out as well, for this demo.

`proxy` used in the demo and the original visualization to keep a consistent context inside of mouse-related callback functions. This allows us to get access to the location and data of the specific `SimpleBubble` that is being moused over to display in the tooltip.

### [jQuery.append()](http://api.jquery.com/append/)

`append` is used to combine the various `divs` used to make up the `SimpleBubble` visual representation. It is similar to `d3.append`, but works on jQuery objects. `append` is also used to add the bubbles to the main visualization `div`.

### [jQuery.remove()](http://api.jquery.com/remove/)

The compliment to `append`, `remove` removes an element from the `DOM`. We use it here as a way to get rid of the tooltip box. Probably not the most efficient method, but it works.

Now that the jQuery introduction is over, lets look in more detail at the two *classes* that make up this demo, starting with our data container/view class `SimpleBubble`.

SimpleBubble
------------

As mentioned above, instances of `SimpleBubble` store the data and `DOM` elements used to represent a bubble. Below is its `init` function which gets called in the constructor when we say something like `var b = new SimpleBubble(data, id, canvas);` :

{% highlight javascript %}
SimpleBubble.prototype.init = function() {
  /* Elements that make up the bubbles display*/
  this.el = $("<div class='bubble' id='bubble-" + this.id + "'></div>");
  this.elFill = $("<div class='bubbleFill'></div>");
  this.el.append(this.elFill);

  /* Attach mouse interaction to root element */
  /* Note use of $.proxy to maintain context */
  this.el.on('mouseover', $.proxy(this.showToolTip, this));
  this.el.on('mouseout', $.proxy(this.hideToolTip, this));

  /* Set CSS of Elements  */
  this.radius = this.data;
  this.boxSize = this.data * 2;

  this.elFill.css({
    width: this.boxSize,
    height: this.boxSize,
    left: -this.boxSize / 2,
    top: -this.boxSize / 2,
    "background-color": colors(this.data)});
};
{% endhighlight %}

So first we create some free standing `divs` that will make up the visual representation of the bubble. We store them as part of the `SimpleBubble` instance. the `jQuery.append()` method is used to attach the sub-elements to the root `this.el` element.

This demo only uses one sub-element, `this.elFill`. The NYT Visualization uses a number of child elements to hold text, background images, etc.

The `jQuery.proxy()` method is used to allow us to use tooltip functions defined in `SimpleBubble` to show and hide the tooltip when mousing.

Finally, CSS is applied using the jQuery object’s `css` method for the child `elFill` element. This is done to position it correctly inside the parent element.

Getting the positioning correct requires just the right css position incantations. While this all could be specified in the javascript, I decided to put them in a stand-alone CSS file, since they are not dependent on any data associated with the bubble. Here is the CSS applied to the bubble elements:

{% highlight css %}
.bubble {
  display: block;
  position: absolute;
}

.bubbleFill {
  position: absolute;
  display: block;
  border: solid 1px white;
  -webkit-border-radius: 70px;
  -moz-border-radius: 70px;
  border-radius: 70px;
}
{% endhighlight %}

You can see that the main `.bubble` elements and their `.bubbleFill` child elements have their positions set to `absolute`. This means `.bubble` element’s position is based on its parent element (the main visualization div) and `.bubbleFill` element’s position is based on `.bubble`. So modifying the `left` and `top` stylings of `.bubble` will be done relative to the main visualization element (and not the entire page) and will in turn modify its child element’s positions.

The other thing to note is that we use the `border-radius` property to make the `divs` into **circles**. This leads to a complication with IE, as will be talked about more later.

Speaking of moving the bubbles, let’s look really quickly at the `move` method:

{% highlight javascript %}
SimpleBubble.prototype.move = function() {
  this.el.css({top: this.y, left:this.x});
};
{% endhighlight %}

Pretty simple. The bubble’s `x` and `y` are expected to be modified by `SimpleVis`. The `move` function just modifies the css of the bubble to shift its position. This isn’t the most abstracted way to implement this functionality, but it works.

The `data` given to the bubble instance in this demo is a simple integer used to determine its size. In a real visualization, you would probably use a javascript object to pass more information to the `SimpleBubble` for it to store.

The tooltip labels are lazily done, because I got lazy, so lets not focus on those, ok?

SimpleVis
---------

`SimpleVis` stores the force layout, creates new `SimpleBubble` instances for each data point, and moves these bubbles based on the force layout. Most of this functionality is in it’s `init` method, so lets look at that:

{% highlight javascript %}
SimpleVis.prototype.init = function() {
  /* Store reference to original this */
  var me = this;

  /* Initialize root visualization element */
  this.canvas.css({
    width: this.width,
    height: this.height,
    "background-color": "#eee",
    position: "relative"});

  /* Create Bubbles */
  for(var i=0; i< this.data.length; i++) {
    var b = new SimpleBubble(this.data[i], i, this.canvas);
    /* Define Starting locations */
    b.x = b.boxSize + (20 * (i+1));
    b.y = b.boxSize + (10 * (i+1));
    this.bubbles.push(b);
    /* Add root bubble element to visualization */
    this.canvas.append(b.el);
  };

  /* Setup force layout */
  this.force = d3.layout.force()
    .nodes(this.bubbles)
    .gravity(0)
    .charge(this.bubbleCharge)
    .friction(0.87)
    .size([this.width, this.height])
    .on("tick", function(e) {
      me.bubbles.forEach( function(b) {
        me.setBubbleLocation(b, e.alpha, me.centers);
        b.move();
      });
    });

  this.force.start();
};
{% endhighlight %}

Ok, so first we setup the `div` that will be holding our visualization. `this.canvas` is a jQuery object wrapped around a `div` present in the page. This div gets passed into the `SimpleVis` constructor, as we will see. Here the position is set to `relative` using the `css` method instead of in a static style-sheet. This is just to show another way it could be done.

Then we iterate over our data and add create new bubbles for each data point. Initial positions are just a guess on my part. They could be set to anything - depending on how you wanted the initial state of your visualization to look.

Finally, the `d3.layout.force` is initialized. This process is very similar to how [my previous bubble chart example](http://vallandingham.me/bubble_charts_in_d3.html) worked. Note that a method in `SimpleVis` determines the next location for each bubble on each **tick**. And then each bubble is moved to that location.

We use `me` so that we can refer to the original `SimpleVis` inside of the `tick` callback function.

The force’s `charge` parameter is defined by a function that will be passed each node. Lets look at this function really quickly:

{% highlight javascript %}
  this.bubbleCharge = function(d) {
    return -Math.pow(d.radius,1) * 8;
  };
{% endhighlight %}

So, just like before, it uses the node’s radius to determine a relative charge to push away other nodes with. Quick and easy collision ‘detection’.

Lastly, lets checkout the `setBubbleLocation` method:

{% highlight javascript %}
SimpleVis.prototype.setBubbleLocation = function(bubble, alpha, centers) {
  var center = centers[this.bin(bubble.id)];
  bubble.y = bubble.y + (center.y - bubble.y) * (0.115) * alpha;
  bubble.x = bubble.x + (center.x - bubble.x) * (0.115) * alpha;
};
{% endhighlight %}

We can see that we get a center location based on the bubble’s `id`, then modify the `x` and `y` variables to move the bubble closer to that position. This movement is tempered by the force’s `alpha`, which will decay as time goes on.

Caveats and Trade Offs
----------------------

### The Binding Question

So just because we **can** use D3 without SVG doesn’t mean its always a great solution. We’ve gained access to D3’s great force directed layout. Using some shims (described below), we can also use D3’s scales and some of its array manipulation features.

We didn’t use one of D3’s most powerful constructs - **binding** data to the data’s representation using the built in `selectAll` and `data` functions. While we certainly can use D3 to bind data to any `DOM` element, the choice to use a class-based approach to encapsulate the data and representation in `SimpleBubble` replaced this D3 binding functionality.

More investigation might bring out a way to cleanly use D3’s binding capabilities with more complicated object based representations.

### It Works in IE?

The original NYT graphic works great in IE8. This demo has some problems.

Specifically, the round appearance of the bubbles is created using the CSS `border-radius` property - but **IE8 doesn’t support this**! So how does the NYT graphic still look good? If the browser is found not to support `border-radius`, then a static circular image is used instead.

Generating a bunch of `.gif` images for this demo isn’t something I’m prepared to do, so I’ll have to be satisfied with boxes in IE. But otherwise it is impressive how cross-platform the NYT visualization is. They include a number of tweaks to optimize the experience for the iPad as well. Something that I might look at more later.

Other Tips
----------

### es5-shim

This demo also uses [es5-shim](https://github.com/kriskowal/es5-shim) to add missing functionality to Arrays in IE. The NYT visualization doesn’t use this particular library, but d3 uses some of these functions in places like scales, so its useful to have.

### html5

As [this bug report](https://github.com/mbostock/d3/issues/599) points out, loading d3 in IE won’t work unless the page is html5. So keep this in mind.

[Here is a link](https://github.com/vlandham/vlandham.github.com/blob/master/vis/js/d3_no_svg.js) to the code again. Thanks to the NYT group for creating this great graphic to learn from. In a future post, we may explore the drag and drop capabilities that make this visualization so interactive.
