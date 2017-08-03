# Intermediate d3

### Objectives:

By the end of this chapter, you should be able to:

- Create linear scales to assist with graphing data in d3
- Create axes in d3 to label data
- Create scatterplots in d3
- Create histograms in d3

By now we've seen how to bind data to elements using `d3`, and we've explored how we can use the `enter` and `exit` selections when there's a mismatch between the amount of data we have and the number of elements in our selection. We've also created some simple svgs and used d3 to add rectangles and circles to those svgs. 

Now it's time to explore a few more features of d3 that you'll need to be comfortable with when you're graphing data.

Throughout this chapter, we'll be using the following code as a baseline:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>D3 graphing</title>
</head>
<body>
  <script src="https://d3js.org/d3.v4.min.js"></script>
  <script src="states.js"></script>
  <script src="app.js"></script>
</body>
</html>
```

We'll start with an empty `app.js` file. The `states.js` file is filled with data on different U.S. states; you can find the data [here](./examples/states.js).

### Scales 

To begin, create `app.js` and open it up in Sublime. Since this is the last file to load, we'll have access both to d3 and to the array of data on states from the `states.js` file. Let's first use d3 to create an SVG and then bind some circles to the states data.

```js
document.addEventListener("DOMContentLoaded", function() {

  var width = 500;
  var height = 500;

  var svg = d3.select("body")
              .append("svg")
                .attr("width", width)
                .attr("height", height);

  svg.selectAll("circle")
     .data(states)
     .enter()
     .append("circle")
       .attr("cx", function(d) { return d.population; })
       .attr("cy", function(d) { return d.electoralVotes; })
       .attr("r", 10);

});
```

If you open up your `index.html` in Chrome, you should see a blank screen. What gives? Did we forget to append the circles correctly?

Head to the Elements tab in Chrome, and you'll see that all the SVG elements are there, just far off the screen. This is because the SVG only has a width of 500 pixels, but many of the population figures are in the millions! In other words, the data we're trying to graph is sitting outside the bounds of our graph area.

So how can we fix this problem? One approach would be to try to scale all of our data by some factor, to make sure that the circles we're appending are visible. For example, if you divide the population figures by 10000, and multiply the electoral vote figures by 10, the data will fit more easily into a range between 0 and 500 on each axis.

```js
svg.selectAll("circle")
     .data(states)
     .enter()
     .append("circle")
       .attr("cx", function(d) { return d.population/10000; })
       .attr("cy", function(d) { return d.electoralVotes*10; })
       .attr("r", 10);
```

This is still a flawed approach, though. For one, some of our points are still not visible: the most populous states are still out of range. We need to choose our scaling factors more carefully, and for that we need to take a closer look at our data and see what values it ranges over. For another, it looks like electoral votes _decrease_ as population increases, because of the way SVGs measure _y_ position: a `cy` attribute of 0 corresponds to a point at the _top_ of the SVG, not the bottom.

Thankfully, d3 can handle both of these problems for us using a _scale_. A scale is just a mapping between two intervals: for example, if you want to map the points from 0 to 1 to the points from 0 to 100, a _linear_ scale would map 0.5 to 50, 0.25 to 25, and so on.

d3 has built-in functionality for creating and using scales. Here's how we can scale our data to better fit in the SVG:

```js
var xMin = d3.min(states, function(d) { return d.population; });
var xMax = d3.max(states, function(d) { return d.population; });
var yMin = d3.min(states, function(d) { return d.electoralVotes; });
var yMax = d3.max(states, function(d) { return d.electoralVotes; });

var xScale = d3.scaleLinear()
               .domain([xMin,xMax])
               .range([0,width]);

var yScale = d3.scaleLinear()
               .domain([yMin,yMax])
               .range([height,0]);

svg.selectAll("circle")
   .data(states)
   .enter()
   .append("circle")
     .attr("cx", function(d) { return xScale(d.population); })
     .attr("cy", function(d) { return yScale(d.electoralVotes); })
     .attr("r", 10);
```

There are a few things going on here. First, we're using d3's `min` and `max` methods to calculate the minimum and maximum values for population and electoral votes. This affords us greater flexibility: we don't have to manually go through the data to find the min and max, and if we added or removed data, we could call these methods again to update the min and max for us.

Next we're creating two linear scales. When you create a scale in this way, you need to define two things: a _domain_ and a _range_. The domain is the interval of values you're mapping _from_ (in this case, the min and max values of our data). The range is the interval of values you're mapping _to_, and will almost always depend on the dimensions of the SVG.

If you're confused by the scales, just play around with them in the Chrome console. You'll see that they really do map numbers from one interval to numbers in another.

```js
var yMin = d3.min(states, function(d) { return d.electoralVotes; });
var yMax = d3.max(states, function(d) { return d.electoralVotes; });

var yScale = d3.scaleLinear()
               .domain([yMin,yMax])
               .range([height,0]);

yMin; // 3
yMax; // 55 - California has lots of people in it.

yScale(3); // 500 - note we flip the yScale so that the yMin gets mapped to 500, which is at the BOTTOM of the SVG
yScale(55); // 0
yScale(29); // 250 - the midpoint of one interval gets mapped to the midpoint of the other interval, as expected.
```

Finally, once we have our scales, we use them when we plot the circles. Notice that we're not plotting the data itself anymore, but rather the result of the data after it's passed into either the _x_ or _y_ scale.

If you refresh the page, you should notice an immediate effect. The data now looks much more reasonable: as a state's population grows, so too does its number of electoral votes.

One slightly annoying thing is that the points at the extreme ends of the scale are getting cropped. While the centers of all the points fit in the SVG, the radii sometimes spill outside of the bounds. The most common way to fix this is to include some padding in the scales:

```js
var padding = 10;

var xScale = d3.scaleLinear()
               .domain([xMin,xMax])
               .range([padding,width - padding]);

var yScale = d3.scaleLinear()
               .domain([yMin,yMax])
               .range([height - padding,padding]);
```

If you want finer control over the padding you can set separate values for top, left, right, and bottom. But for now, we'll use one value for all four sides.

Let's graph a different relationship: median income versus population density. We don't have population density in our objects, but we can calculate it since we know total population and total area! This change will require us to alter how we calculate minima and maxima, as well as the `cx` and `cy` attributes on our circles:

```js
document.addEventListener("DOMContentLoaded", function() {

  var width = 500;
  var height = 500;
  var padding = 10;

  var svg = d3.select("body")
              .append("svg")
                .attr("width", width)
                .attr("height", height);

  var xMin = d3.min(states, function(d) { return d.population / d.area; });
  var xMax = d3.max(states, function(d) { return d.population / d.area; });
  var yMin = d3.min(states, function(d) { return d.medianIncome; });
  var yMax = d3.max(states, function(d) { return d.medianIncome; });

  var xScale = d3.scaleLinear()
                 .domain([xMin,xMax])
                 .range([padding,width - padding]);

  var yScale = d3.scaleLinear()
                 .domain([yMin,yMax])
                 .range([height - padding,padding]);

  svg.selectAll("circle")
     .data(states)
     .enter()
     .append("circle")
       .attr("cx", function(d) { return xScale(d.population / d.area); })
       .attr("cy", function(d) { return yScale(d.medianIncome); })
       .attr("r", 10);

});
```

(In this case, Washington D.C. is a major outlier, since its population density is so high. You may want to remove it from the dataset for now.)

Let's get some more practice with scales by creating a couple more! Let's first resize the dots on our plot so that geographically larger states have larger dots on the scatterplot. We'll start by calculating a min, max, and creating a scale:

```js
var rMin = d3.min(states, function(d) { return d.area; });
var rMax = d3.max(states, function(d) { return d.area; });
var rScale = d3.scaleLinear()
               .domain([rMin,rMax])
               .range([5,25]);
```

(You may also want to increase the padding on the left, since Alaska is so huge.)

Next, we need to update the `r` attribute on each circle, so that it's no longer a fixed value but instead depends on our scale:

```js
  svg.selectAll("circle")
     .data(states)
     .enter()
     .append("circle")
       .attr("cx", function(d) { return xScale(d.population / d.area); })
       .attr("cy", function(d) { return yScale(d.medianIncome); })
       .attr("r", function(d) { return rScale(d.area); });
```

Let's create one more scale, but this one will be a little different. Notice that in our set of data we keep track of how many democratic and republican representatives each state has. We can use this to calculate a measure of bipartisanship for each state. Our measurement will simply be the percentage of representatives who are Republican. In other words, a score of 1 means every representative is a Republican, a score of 0 means every representative is a Democrat, and a score of 0.5 means half of the representatives are Democrats and half are Republicans.

Let's first write a helper function to calculate this percentage:

```js
function partisanScore(d) {
  return d.republicanReps / (d.democraticReps + d.republicanReps);
}
```

Let's also create a scale which maps numbers from 0 to 1 onto colors. This might seem like a more complicated process: after all, until now we've always mapped intervals of numbers to other intervals of numbers. But in fact, d3's scales are quite flexible. Take a look!

```js
var colorScale = d3.scaleLinear()
                   .domain([0,1])
                   .range(["blue","red"]);
                   
// some examples
colorScale(0); // "rgb(0, 0, 255)"
colorScale(1); // "rgb(255, 0, 0)"
colorScale(0.5); // "rgb(128, 0, 128)"
colorScale(0.1); // "rgb(26, 0, 230)"
```

Cool, right? Now let's add some color to our points. We'll do this with the "fill" attribute:

```js
svg.selectAll("circle")
   .data(states)
   .enter()
   .append("circle")
     .attr("cx", function(d) { return xScale(d.population / d.area); })
     .attr("cy", function(d) { return yScale(d.medianIncome); })
     .attr("r", function(d) { return rScale(d.area); })
     .attr("fill", function(d) { return colorScale(partisanScore(d)); });
```

You can also set a border around the points if you like, using the `stroke` attribute.

We've now got a graph which illustrates our set of data across four dimensions: population density, land area, median income, and partisanship! Are there any other relationships in the data that you'd like to explore? Try altering the values for the `cx`, `cy`, or `r` attributes and see what other interesting relationships you can find!

### Axes

There are a number of other features that would be good to add to our graph, but we'll focus on just one more. Right now, there's no way to tell what relationship is being graphed, because we don't have any axes! 

Creating axes manually can be a real pain: you've got to decide on the scale, draw lines, draw tick marks, draw labels for the tick marks, and so on. d3 simplifies this process with a number of built-in axis methods. Let's continue with our current application, and see how we can create axes in just a few lines of code.

Let's start by creating a _y_ axis.

```js
svg.append('g')
  .attr("transform", "translate(" + padding + ",0)")
  .call(d3.axisLeft(yScale));
```

With this code we're doing a few things:

1. We're appending a new element to the SVG, called a `g` (short for "group")
2. We're moving this group to the right to account for our padding.
3. We're using d3's built-in `axisLeft` method to create an axis corresponding to our `yScale`.

After that, d3 takes care of the rest! It makes the lines, the tick marks, and the tick labels. It doesn't do everything: if you want a label for the axis, you'll need to do that yourself. And if you want more fine-grained control over the tick marks, you should check out the [docs](https://github.com/d3/d3-axis). But this is a pretty good amount of functionality for just three lines of code! (If your axis labels are getting cropped, you can increase the padding or use `axisRight` instead of `axisLeft`.)

The code for the _x_ axis is very similar:

```js
svg.append('g')
   .attr("transform", "translate(0," + (height - padding) + ")")
   .call(d3.axisBottom(xScale));
```

If you want to change the default styles of the axes, you can do that with CSS or by setting styles when you append the group. Note that the default axis styling for version d3 wasn't the nicest; it's been cleaned up quite a bit in v4.

### Bar Chart

Let's revisit bar charts briefly and create one with our data instead of a scatterplot. There's a lot of data to choose from: let's create a bar graph that plots the median income for each state, and colors the bars by partisanship. You should try to implement as much of this as possible on your own, before continuing on to the discussion.

We'll begin with a similar approach to what we did with the scatterplot: we'll make an SVG, give it some width and height, and create a scale for the _y_ axis which is generated from the data on median income. We'll keep our color scale too. Finally, we'll also want better control over the padding, so we'll specify different padding values for each of the four sides of the SVG.

```js
var width = 1000;
var height = 600;
var padding = {
  left: 40,
  top: 10,
  right: 0,
  bottom: 100
};

var svg = d3.select("body")
            .append("svg")
              .attr("width", width)
              .attr("height", height);

var yMin = d3.min(states, function(d) { return d.medianIncome; });
var yMax = d3.max(states, function(d) { return d.medianIncome; });

var yScale = d3.scaleLinear()
               .domain([yMin,yMax])
               .range([height - padding.top,padding.bottom]);
               
var colorScale = d3.scaleLinear()
                   .domain([0,1])
                   .range(["blue","red"]);               
``` 

Our _x_ axis is different in this case, because we're not scaling numbers to numbers, or even colors to numbers. Instead, what we want is an _ordinal_ scale: we just want to rank states and label them by their names. 

Fortunately, d3 provides us with another type of scale to deal with this scenario: a _band_ scale. We can use this scale to map a set of non-numeric data (e.g. a list of state names) onto our axis. Here's the syntax for that scale:

```js
var xScale = d3.scaleBand()
               .domain(states.map(function(d) {
                 return d.name;
               }))
               .range([padding.left,width - padding.right])
               .padding(0.1);
```

The `padding` method lets us specify how much space we want in between our bars. The larger the number that we pass in (between 0 and 1), the more space there will be.

Next, we need to create our rectangles. We'll also need our `partisanScore` function from before, to control the fill of each rectangle:

```js
svg.selectAll("rect")
   .data(states)
   .enter()
   .append("rect")
     .attr("x", function(d) { return xScale(d.name); })
     .attr("width", xScale.bandwidth())
     .attr("y", function(d) {return yScale(d.medianIncome) - padding.bottom + padding.top; })
     .attr("height", function(d) { return height - yScale(d.medianIncome); })
     .attr("fill", function(d) { return colorScale(partisanScore(d)); });
     
function partisanScore(d) {
  return d.republicanReps / (d.democraticReps + d.republicanReps);
}
```

There are a couple of things to note. First, the band scale does a fair amount of the work when it comes to drawing the scale. The `x` attribute is set by the scale itself; as for the width, the scale has a method on it called `bandwidth` which will automatically size the rectangles so that they are as wide as they need to be to fill the SVG.

The vertical components are a bit trickier. The `y` attribute isn't so bad, but you need to remember to account for the padding on top and bottom of the SVG. If we don't, there won't be enough room along the bottom for the x-axis along with its labels. The height is probably the trickiest one to get right. Since the scale itself transforms large incomes to small values, we need to subtract the height from the value of the scale. If you don't do this, bars which should be short will be tall (and vice versa).

Finally, we can add our axes. The _y_ axis should look quite familiar:

```js
svg.append('g')
     .attr("transform", "translate(" + padding.left + "," + (-padding.bottom+padding.top) + ")")
     .call(d3.axisLeft(yScale));
```

However, the _x_ axis requires a bit more care. If we try to do what we did for the _y_ axis, the code should look like this:

```js
svg.append('g')
     .attr("transform", "translate(0," + (height - padding.bottom+padding.top) + ")")
     .call(d3.axisBottom(xScale))
```

Unfortunately, this presents a problem: the text along the axis is unreadable! Because the state names are all horizontally aligned and relatively long, they overlap on top of one another.

One solution is to rotate the text by 90°:

```js
svg.append('g')
     .attr("transform", "translate(0," + (height - padding.bottom+padding.top) + ")")
     .call(d3.axisBottom(xScale))
   .selectAll("text")
     .attr("transform", "rotate(90)");
```

Now the text is vertically aligned, but it's too high up and not quite centered relative to the tick mark. With a little bit of nudging, we can get things looking good:

```js
svg.append('g')
     .attr("transform", "translate(0," + (height - padding.bottom+padding.top) + ")")
     .call(d3.axisBottom(xScale))
   .selectAll("text")
     .attr("transform", "rotate(90) translate(" + padding.top + "," + (-3-xScale.bandwidth()/2) + ")")
     .style("text-anchor", "start");
```

Notice that we not only translated the text elements a bit, we also set the `text-anchor` property to `start` (rather than its default of `middle`). For more on this property, check out [MDN](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/text-anchor).

That's it! We've created a bar chart in d3 using real-world data. What other data would you like to see in bar chart form?     

### Additional Resources

We've covered a fair amount of ground when it comes to d3, but there's still plenty left to discover. Here are some other areas to learn about if you're interested:

- Check out the [docs](https://github.com/d3/d3-scale) to learn more about other kinds of scales available in d3.
- Transitions are another big part of d3 that we haven't talked about! You can learn about them [here](https://github.com/d3/d3-transition).
- d3's `data` function takes an optional second argument, called a key function. What do these functions do, and what's a good use case for them? ([This](http://code.hazzens.com/d3tut/lesson_4.html) lesson is a good place to start.)
- Force layouts are a popular way to visualize data in d3, but one we haven't discussed at all. For a primer, check out [this](https://medium.com/@sxywu/understanding-the-force-ef1237017d5#.mb6nsbo3a) article (and the references therein).

### Exercise

Complete Part II of the [d3 Exercises](https://github.com/rithmschool/intermediate_js_exercises/tree/master/d3_exercises).

#### [⇐ Previous](./09-data-joins-in-d3.md) | [Table of Contents](./../readme.md) | [Next ⇒](./11-project.md)
