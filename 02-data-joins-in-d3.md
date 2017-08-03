# Data Joins in d3

### Objectives:

By the end of this chapter, you should be able to:

- Use the `data` method to join data to HTML or SVG elements
- Use the `enter` method to dynamically add elements joined with data to the page
- Use the `exit` method to dynamically remove elements from the page
- Create bar charts in d3!

### Joining data to elements

In the previous chapter, we saw how d3 could be used to manipulate the DOM. However, if that were all d3 were good for, it wouldn't be that interesting; after all, jQuery can do most of the things we've already seen d3 do.

So let's learn about what sets d3 apart. To begin, let's create some HTML boilerplate:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Putting the Data in d3</title>
</head>
<body>
  <h1>Read this important article!</h1>
  <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Inventore perferendis voluptatibus eaque expedita aspernatur, neque quo ratione aut vero, cumque, dolores similique repellendus saepe sapiente, voluptas veniam. Expedita, eius, distinctio.</p>
  <p>Sapiente maxime, unde esse quasi eius facere! Numquam nam officiis dolorem quo dignissimos nobis id quae blanditiis neque, magni similique, aspernatur eligendi quia. Voluptatum architecto perspiciatis amet, deleniti, sit earum.</p>
  <p>Nam maiores error sequi fugiat itaque, quae, odit quo officiis est atque laborum officia, doloremque. Molestiae recusandae maxime repellat aperiam tenetur eveniet sunt ipsum id aliquam, possimus deserunt, laudantium vero?</p>
  <p>Officia libero, est sapiente sint quaerat eaque veritatis neque quod animi nobis nostrum, quisquam rem ea sunt nihil optio earum voluptate ut non ducimus. Maiores soluta, nulla! Quis, possimus quasi?</p>
  <p>Voluptas dicta ducimus nulla, magni id quis ab ratione cupiditate? Necessitatibus exercitationem hic, saepe, voluptatem dolor quibusdam est autem nulla praesentium cumque assumenda illo similique ipsam culpa voluptates, minus! Neque.</p>
  <p>Officiis, veniam, tenetur. Blanditiis consequatur ab fugit veniam totam? Est atque ratione cum odio numquam, animi repellat, dolor nam molestiae facilis harum minus voluptates aspernatur adipisci soluta reprehenderit officia alias.</p>
  <p>Earum obcaecati magnam libero ad molestiae corrupti non, doloribus molestias blanditiis atque fugit voluptate eveniet, quae adipisci provident, repellendus ratione facilis voluptas ipsam? Dolore, repudiandae. Itaque repudiandae illo architecto quaerat.</p>
  <p>Eius impedit minima sapiente sit ut voluptate deleniti tempora, repellendus sed aliquam. A nobis tempora aliquam commodi, voluptate necessitatibus quaerat accusantium incidunt error, iste. Maiores autem mollitia rerum asperiores repellat.</p>
  <p>Neque et magnam ea corporis placeat, recusandae voluptatem quos in cum numquam, beatae animi assumenda. Praesentium non sequi, perspiciatis odit. Quaerat dolorem labore sint dolore, nemo cum amet dolor in?</p>
  <p>Adipisci, natus omnis reprehenderit. Saepe repudiandae accusamus ab? Sapiente eveniet minus ipsam minima quod quisquam assumenda officiis architecto voluptas iure ut odit ab ratione voluptatibus, est quidem cumque voluptate veritatis!</p>
  <script src="https://d3js.org/d3.v4.js"></script>
</body>
</html>
```

Let's hop into the Chrome console and learn about one of the most important d3 methods: the `data` method.

Before explaining how this method works, or what it's used for, let's first just examine a quick example:

```javascript
d3.selectAll('p')
	.data([4, 8, 12, 16, 20, 24, 28, 32, 36, 40])
	.style('font-size', function(d) {
		return d + 'px';
	});
```

Woah, what happened? Each paragraph now has a different font size!

This is the power of the d3 data method. It allows you to bind data to DOM elements, then use that data to affect the look of those elements on the page. 

We saw in the last chapter that you can pass callback functions to many methods in d3, including `attr` and `style`. We also saw that the second argument in that callback function refers to the index of the current element in the selection. Now we can understand what the _first_ argument is as well: it's the data joined to that element. In the above example, the first `p` tag gets joined with the value 4, the second `p` tag gets joined with the value 8, and so on. This means that when we dynamically set the font-size of each `p` tag, the first one will have a 4px font size, the second will have an 8px font size, and so on! To understand this more concretely, try playing around with the data that you're passing in, or with what you're returning in the `style` callback.

So how does this work? When you call the `data` method on a selection, under the hood d3 is adding a private `__data__` property to HTML elements in the selection. Refresh the page and try the following:

```javascript
document.querySelector('p').__data__; // undefined

// Let's bind some data...
d3.selectAll('p')
	.data([4, 8, 12, 16, 20, 24, 28, 32, 36, 40])
	
document.querySelector('p').__data__; // 4 - the data value has been joined to the p tag!
```

While you can access this property, you shouldn't ever be manipulating it directly. If you need to update the data, just call `data` on the selection again! Refresh the page once more and try this out:

```javascript
d3.selectAll('p')
	.data([4, 8, 12, 16, 20, 24, 28, 32, 36, 40]);
	
document.querySelector('p').__data__; // 4

d3.selectAll('p')
	.data(["first", 8, 12, 16, 20, 24, 28, 32, 36, 40]);
	
document.querySelector('p').__data__; // "first"
```

### Surplus data: the `enter` method

In the previous example, the number of data points we had (i.e. the size of the array we passed in to `data`) was equal to the number of paragraphs we had on the page. But very often, there's a mismatch between your data and the number of elements. For example, you may need to get some data via AJAX without knowing beforehand how many data points you're getting. So how can you be sure you'll have enough elements on the page to display the data?

When you have data without any corresponding elements, the way to deal with them is with d3's `enter` method.

For example, suppose you have the following html:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Putting the Data in d3</title>
</head>
<body>
  <h1>Things to do</h1>	
  <ul></ul>
  <script src="https://d3js.org/d3.v4.js"></script>
  <script src="todo.js"></script>
</body>
</html>
```

In your `todo.js`, let's write out some things to do:

```javascript
var todos = [
	"Create 1000 data visualizations with d3",
	"Eat dinner",
	"Sleep",
	"Hang out with friends"
];
```

Then, in the same JavaScript file, let's try to bind these todo items to HTML elements.

```javascript
d3.select("ul")
  .selectAll("li")
    .data(todos);
```

This code may seem a bit strange. After all, our HTML has a `ul`, but that `ul` doesn't have any `li`s in it. So when we select all of the `li`s, aren't we getting an empty selection?

At first, yes! In fact, if you refresh the page and take a look at `d3.select("ul").selectAll("li").nodes()`, you should see an empty array. But the power of the `data` function is that it allows us to bind to elements that _aren't yet present on the page_.

If you take a look at `d3.select("ul").selectAll("li").data(todos)` in the Chrome console, you'll see a d3 selection object which is similar to the ones we encountered in the previous chapter, but with an important difference: there are now two new key-value pairs! In addition to the `_groups` key and the `_parents` key that we saw before, there's an `_enter` key and an `_exit` key as well. We'll talk about `_exit` momentarily, but for now, let's investigate `_enter`.

To expose what's in the `_enter` property, we use the `enter` method. Let's take a look at the nodes inside of the enter selection:

```javascript
d3.select("ul")
  .selectAll("li")
    .data(todos)
    .enter()
    .nodes();
```

You should see that `nodes` returns an array of `EnterNode` objects to you. You can think of this as a placeholder: each `EnterNode` is basically a placeholder for some DOM element that doesn't exist yet. However, the `EnterNode` does keep track of the data that should be bound to it.

So how do we generate the actual HTML elements on the page? By appending them! Try this code out:

```javascript
d3.select("ul")
  .selectAll("li")
    .data(todos)
    .enter()
  .append("li")
    .text(function(d,i) {
    	return "Todo #" + (i + 1) + ": " + d;
    });  
```

With this, your todos will render to the page!

### Creating our first bar chart

Let's take a look at another example, this one using an SVG. Here's some HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Putting the Data in d3</title>
</head>
<body>
  <script src="https://d3js.org/d3.v4.js"></script>
  <script src="main.js"></script>
</body>
</html>
```

Let's draw a bar chart! Take a look at the following code and try to understand it line-by-line:

```javascript
// main.js
var svgWidth = 500;
var svgHeight = 500;
var barWidth = 90;
var barGap = 10;

d3.select("body")
  .append("svg")
    .attr("width", svgWidth)
    .attr("height", svgHeight)
  .selectAll('rect')
    .data([
      svgHeight * Math.random(), 
      svgHeight * Math.random(), 
      svgHeight * Math.random(), 
      svgHeight * Math.random(), 
      svgHeight * Math.random()
    ])
    .enter()
  .append('rect')
    .attr('width', barWidth)
    .attr('height', function(d) { return d; })
    .attr('y', function(d) { return svgHeight - d; })
    .attr('x', function(d, i) { return (barWidth + barGap) * i; })
    .attr('fill', 'blue');
```

Let's walk through this code. To begin, we're initializing some variables. The first two store the width and height of our `svg`, the third one stores the width of each bar in our bar chart, and the last one sets the gap between each bar (we'd like to give our bars some breathing room so they aren't bunched up right next to each other).

Next, we break out our d3 tools. First, we select the `body` and append an `svg` to it, setting the width and the height. Strictly speaking, this isn't necessary - we could have just as easily put the `svg` in our HTML file - but the extra practice with d3 syntax can't hurt.

After appending the `svg`, the next thing we do is create an empty selection by selecting all `rect` elements on the page, and joining some randomly generated data using the `data` method. As we saw before, this creates an enter selection, which is basically just a selection of placeholder nodes that are bound to data but waiting to be appended.

After calling `enter`, we then append rectangles to the page, one for each piece of data. Here's where the fun begins: now we have to figure out where to place the rectangles on the `svg`!

The `width` is relatively straightforward: this should just be the `barWidth` we set initially. The height isn't too bad either: in this case, we want the _i_ th data value to correspond to the height of the _i_ th bar, so the callback function just returns the data value.

Things get trickier when it comes to the `x` and `y` attributes of the rectangles. It might seem like the `y` coordinate should also equal the randomly generated height data. But remember: the value for the `y` attribute is measured from the _top_ of the `svg`, not the bottom! This is why the callback function for the `y` attribute returns `svgHeight - d` and not `d`. (If this seems confusing, try altering the callback so that it just returns `d` and see what happens!)

The callback used to set the `x` attribute is probably the most intricate of the bunch. Think about it this way: the first bar can be aligned all the way to the left of the `svg` (i.e., its `x` attribute can be set to 0). The second bar needs to be offset to the right, but by how much? Well, you need to give enough room for the first bar, and for the gap between the bars. In other words, it should be offset by `barWidth + barGap`. Similarly, the third bar should be offset by `2 * (barWidth + barGap`); that's one copy of `barWidth + barGap` for each of the first two bars. In general, then, the bar and index `i` should be offset by `(barWidth + barGap) * i`. 

If any of these expressions are confusing, try changing them to see what happens to the graph! There's no better way to familiarize yourself with this stuff than by breaking things and trying to understand why they're broken.

### Surplus elements: the `exit()` method

We've seen that if you join more data to a selection than you have elements, you gain access to an enter selection which lets you append elements to the DOM which are automatically joined to this excess data. But what about the opposite problem, when you join _less_ data to a selection than you have elements? In this case, you can use d3's _exit_ selection!

The most common use case for `exit` is when you remove some data and want to then remove the corresponding element on the page. Let's take a look at a quick example. Let's revisit our list of todos from before, but add an event listener so that clicking on a todo removes it. Let's use the HTML from before, and start with the JavaScript code that adds the todos to the page:

```javascript
var todos = [
	"Create 1000 data visualizations with d3",
	"Eat dinner",
	"Sleep",
	"Hang out with friends"
];

d3.select("ul")
  .selectAll("li")
    .data(todos)
    .enter()
  .append("li")
    .text(function(d,i) {
    	return "Todo #" + (i + 1) + ": " + d;
    });  
```

Below this code, let's add some logic for the event listeners:

```javascript
d3.selectAll('li').on('click', function(d) {

  // find the current todo's index in the array and remove it
  var idx = todos.indexOf(d);
  todos.splice(idx,1)
  
  // update the DOM
  d3.selectAll('li')
    .data(todos)
      .text(function(d,i) {
        return "Todo #" + (i + 1) + ": " + d;
      })
    .exit()
    .remove();

});
```

As you can see, the first thing that happens on click is that we find the index of the todo that was just clicked on, and then use `splice` to remove that todo from our array. This isn't enough, however; we also need to update the page!

Next, we select all of the `li`s, join them to the new data and update the text. It's the last two lines that are new: first we grab the exit selection by calling `exit`, and then we remove those elements from the page. To appreciate the importance of those two lines, try commenting them out and running the code again. Without calling `.exit().remove()`, the elements corresponding to data that no longer exists continue to linger on the page.

### Exercise

Complete Part I of the [d3 Exercises](https://github.com/rithmschool/intermediate_js_exercises/tree/master/d3_exercises).

### Additional Resources

[https://bost.ocks.org/mike/join/](https://bost.ocks.org/mike/join/)

[http://alignedleft.com/tutorials/d3/binding-data](http://alignedleft.com/tutorials/d3/binding-data)

#### [⇐ Previous](./08-introduction-to-d3.md) | [Table of Contents](./../readme.md) | [Next ⇒](./10-intermediate-d3.md)
