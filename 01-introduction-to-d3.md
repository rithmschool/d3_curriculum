# Introduction to D3

### Objectives:

By the end of this chapter, you should be able to:

- Describe what d3 is and how it is used
- Select elements in the DOM using d3
- Set attributes on elements dynamically by passing functions into d3 selectors
- Append elements to the DOM using d3
- Add event listeners using d3
- Explain what SVG is, and append elements to an SVG using d3

### What is d3?

[d3](https://d3js.org/) is a popular and powerful JavaScript library used for data visualization. The term d3 is short for ddd, which itself is short for **d**ata **d**riven **d**ocuments. Any time you see an interactive graph or data visualization on the internet, you should suspect that d3 is being used. For example, The New York Times uses d3 for its interactive articles; [here's](http://www.nytimes.com/interactive/2012/05/17/business/dealbook/how-the-facebook-offering-compares.html?_r=0) an example.

Since d3 is a JavaScript library like jQuery, it's relatively easy to get started. Just like with other libraries, you can use d3 in your JavaScript by linking to the latest version of the d3 source code via a `script` tag. Let's use this code to begin:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Dope D3 (a.k.a. D4)</title>
</head>
<body>
  <h1 id="page-title">This is my first D3 example!</h1>
  <p class="first-paragraph">Here's my first paragraph.</p>
  <p>Here are some reasons why d3 is dope:</p>
  <ol>
    <li>Makes data more engaging.</li>
    <li>Has built-in math helpers.</li>
    <li>Includes several additional data types.</li>
    <li>Supports graph animations.</li>
    <li>Makes drawing graph axes a breeze!</li>
  </ol>
  <script src="https://d3js.org/d3.v4.js"></script>
</body>
</html>
```

Open this page up in Chrome, hop into the console, and type `d3`. If you get an object back, you can be confident that you've set things up correctly!

Note: The source in the script tag refers to version 4 of d3 (hence the `d3.v4.js`). This version of d3 was released in June of 2016. If you're reading d3 tutorials from before then, you should be aware that version 4 introduced a number of changes that aren't compatible with earlier versions of d3. We'll focus on version 4 here, but just be careful if you're reading older tutorials!

### Selecting DOM elements with d3

d3 can do a lot of data manipulation, but even if you're not working with any data, it can also be used to manipulate the DOM. While its functionality doesn't intersect entirely with jQuery (you can't easily do event delegation with d3, for instance), when it comes to traversing the DOM and appending things to it, d3 and jQuery can, in many cases, be used interchangeably. 

Using the same HTML as in the above, hop into the console and let's make our first selection:

```javascript
var firstSelection = d3.select("#page-title");
```

The `d3.select` method is sort of analogous to the vanilla JavaScript `document.querySelector` method, in that both accept a string corresponding to a CSS selector, and both just search until they find a single element matching the selector. If you'd like to find all elements matching the selector, you should use `d3.selectAll` (analogous to `document.querySelectorAll`).

If you look at `firstSelection` in the console, you'll see that it's got a structure we haven't seen before: it's an object constructed from the native d3 `Selection` function, and it has two properties: `_groups` and `_parents`. The underscore in front of these names indicates that they are intended to be *private*; if you find yourself accessing these properties directly, you're probably using d3 incorrectly. For example, if you want to pass from the d3 selection object to the HTML element, you can call the `.nodes` method, which will return to you an array of the HTML elements:

```javascript
firstSelection.nodes()[0]; // this should return the h1 element on the page.
```

While this d3 selection object might seem strange, it's really not so different than what we saw with jQuery. When we use the jQuery function to select elements, it returns a jQuery object. This allows us to apply jQuery methods to that object. Similarly, in d3, `select` and `selectAll` return to us d3 selections. This allows us to apply d3 methods to those selections.

For example, method chaining is a major part of d3 (we'll see this throughout the next couple of chapters). Working with d3 selections exclusively, rather than mixing d3 selections with plain old HTML elements, allows us to use method chaining with ease. For instance, you can chain together multiple selections:

```javascript
var firstLi = d3.select("ol").select("li");
```

This is our first example of method chaining in d3, but it's far from our last.

### Modifying DOM elements with d3

Just like in jQuery, we can modify the attributes and style of our selections using d3. Try writing the following examples in the Chrome console:

```javascript
// more method chaining here!
d3.select("#page-title")
	.style("color", "blue")
	.attr("class", "intro");
```

The above code will select the `h1` with an id of "page-title," set the font color to blue, and add a class of "intro".

When adding or removing classes, it's also common to use the `classed` method, which takes a string and a boolean. The string should be a space-separated list of class names, and the boolean determines whether those classes should be added to or removed from the selection:

```javascript
// adds "items" class to all li's:
d3.selectAll("li")
	.classed("items", true);

// removes "first-paragraph" class from all p tags:
d3.selectAll("p")
	.classed("first-paragraph", false);
```

If you don't supply a second argument to `classed`, it will simply tell you whether the first element in your selection has the class you pass into it:

```javascript
d3.select(".first-paragraph")
	.classed("first-paragraph"); // true

d3.select("ol")
	.classed("this-should-return-false"); // false
```

Also like in jQuery, there are `text` and `html` methods in d3 to set the inner text and inner HTML of a selection. Try these out in the console:

```javascript
d3.select("h1")
	.text("New Title!");
	
d3.select("body")
	.html("<h6>Whoops, all gone.</h6>");
```

If you'd like to read more about the selection API in d3, head over to the [docs](https://github.com/d3/d3-selection).

### Passing functions to d3 methods

So far we've only passed primitive values into our selection methods. But we can also pass in functions. Check this out:

```javascript
d3.selectAll("li")
	.text(function() {
		return Math.random() + " is a random number!"; 
	});
```

When you throw this into the console, you should see that each `li` gets its own random number: in other words, the callback function to `text` is being executed once per matching element in the selection.

While this example might seem a little contrived, it's not at all uncommon for you to want to iterate through a selection of elements and apply some function to each one. Here's a more realistic example:

```javascript
d3.selectAll("li")
	.style("background-color", function(d,i) {
		if (i % 2 === 1) return "#cccccc";
	});
```

In this example, we're using d3 to stripe the list, so that every other item has a different background color. How did we do this? Well, in these callback functions, the second argument refers to the index of the element within the current selection group. In d3, this argument is typically denoted `i`. So the callback is basically returning a light gray hex code whenever the index is odd.

What about that first argument in the callback, the one we called `d`? That's a great question! We'll come to that in the next chapter.

Want another example? Try the following:

1. Select the `p` tag with a class of 'first-paragraph'
2. call the `style` method on the selection, and get ready to set the `color` of the selection.
3. To determine the color, generate a random number. If it's less than 0.5, set the color to be red. Otherwise, set it to be blue.

Try executing your code several times in the console to ensure that it randomly sets the paragraph's text to be one of these two colors!

### Adding to and removing from the DOM with d3

Also as with jQuery, you can use d3 to add or remove elements from the DOM. To remove an item, create a selection and then call `.remove()` on it:

```javascript
d3.select("p").remove();
```

To add elements to the DOM, d3 provides the `append` and `insert` methods. The `append` method is more straightforward: it simply appends the element you want to add as the last child of each element in your selection:

```javascript
// append hr tags to each li:
d3.selectAll("li")
	.append("hr");
```

`insert` works in a similar way, but it allows you to pass in a second argument which specifies where to insert the new element. The new element will be placed _before_ the second argument in the DOM.

```javascript
// insert an hr tag before the h1:
d3.select("body")
	.insert("hr","h1");
```

For more details on these methods, check out the [docs](https://github.com/d3/d3-selection#modifying-elements).

### Event Listeners in d3

Here's another similarity to jQuery: there's an `on` method used for registering event listeners. Unlike jQuery, though, the callback doesn't take the event object as an argument. If you want details of the event, use `d3.event`:

```javascript
d3.select("h1").on('mouseover', function() {
	console.log(d3.event);
});
```

You can access the event target using `d3.event.target`, or you can simply use the keyword `this`, which will refer to the DOM element that triggered the event. Here's another example:

```javascript
d3.selectAll("li").on('click', function() {
	// generate random numbers between 0 and 255;
	var randomRed = Math.floor(Math.random() * 256);
	var randomBlue = Math.floor(Math.random() * 256);
	var randomGreen = Math.floor(Math.random() * 256);
	var randomColor = "rgb("+randomRed+","+randomBlue+","+randomGreen+")";
	d3.select(this).style('color', randomColor);
});
```

### Why Use jQuery?

At this point, you may be wondering why we should use jQuery at all. After all, doesn't d3 do everything that jQuery does?

Well, there are a few differences in terms of DOM manipulation functionality. For example, d3 doesn't support event delegation, while jQuery does. 

But more importantly, d3 is fundamentally a library for data visualization. jQuery is primarily a library for DOM manipulation (including AJAX). If you're not trying to do anything with data, sticking to jQuery is probably fine.

We'll dig into how d3 works with data in the next chapter. But before we do, let's talk a bit about SVGs, as we'll be relying on these elements heavily when using d3 to graph data.

### A Primer on SVG

SVG (short for Scalable Vector Graphics) is a way for us to draw images in the DOM. The use of vector graphics means that the images don't degrade in quality as you resize them (for more on vector graphics, Wikipedia offers a [nice starting point](https://en.wikipedia.org/wiki/Vector_graphics)). The [MDN docs](https://developer.mozilla.org/en-US/docs/Web/SVG) perhaps summarize it best: "SVG is essentially to graphics what HTML is to text."

d3 relies on SVG pretty heavily for drawing graphs. We'll dig into it more in the next chapter, but for now let's just create an SVG element and draw some rectangles and circles.

We can use the same HTML starter code as before. Let's begin by appending an SVG to the DOM! 

```javascript
d3.select("body")
  .append("svg")
    .attr("width", 500)
    .attr("height", 500);
```

You might be wondering what's up with the spacing in the above code. From the [d3 docs](https://github.com/d3/d3-selection):

>By convention, selection methods that return the current selection use _four_ spaces of indent, while methods that return a new selection use only _two_. This helps reveal changes of context by making them stick out of the chain.

In this case, the indentation is meant to help emphasize the fact that the `width` and `height` attributes that are being set are on the `svg`, not on the `body`.

Let's create our first rectangle. How can we do this in d3? By appending a `rect` element to our svg!

```javascript
d3.select('svg')
  .append('rect')
    .attr('width',100)
    .attr('height',200)
    .attr('x',50)
    .attr('y',150)
    .attr('fill','red')
    .attr('stroke-width',5)
    .attr('stroke','blue');
```

That's a lot of attributes we've just set, but if you type this code correctly you should see a red rectangle with a blue border appear in your svg!

Let's dig into these attributes a bit more:

- `width` - this sets the width of the rectangle.
- `height` - this sets the height of the rectangle.
- `x` -  this sets the horizontal position of the top-left corner of the rectangle. The value answers the question "how far from the left edge of the SVG should the left side of the rectangle be?"
- `y` - this sets the vertical position of the top-left corner of the rectangle. The value answers the question "how far from the top edge of the svg should the top of the rectangle be?" It's important to emphasize the fact that this value is calculated from the top of the SVG, _not_ the bottom! 
- `fill` - this sets the color of the interior of the rectangle.
- `stroke-width` - this sets the width of the outline of the rectangle.
- `stroke` - this sets the color of the outline of the rectangle.

Once you've drawn your rectangle, let's draw a circle too!

```javascript
d3.select('svg')
  .append('circle')
    .attr('cx',250)
    .attr('cy',200)
    .attr('r',100)
    .attr('fill','purple');
```

You can probably guess what some of these new attributes do, but here's a quick break down:

- `cx` - this sets the horizontal position of the center of the circle, relative to the left edge of the SVG. 
- `cy` - this sets the vertical position of the center of the circle, relative to the top edge of the SVG.
- `r` - this sets the radius of the circle.

There's plenty more you can do with SVGs (the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/SVG) are a pretty comprehensive resource). We'll dig into things a bit more in the next chapter.

### Exercise

Read through [this article](https://www.dashingd3js.com/svg-basic-shapes-and-d3js) on the basics of SVG shapes and d3, and work through the examples. We'll see more examples of d3 shapes in the next chapter! 

#### [⇐ Previous](./07-functional-programming.md) | [Table of Contents](./../readme.md) | [Next ⇒](./09-data-joins-in-d3.md)
