+++
categories = []
date = "2016-12-02T21:45:19+01:00"
tags = []
title = "D3's Abstraction Trouble"

+++

<script src="https://d3js.org/d3.v4.min.js"></script>

D3 is a wonderful idea. A graphing library in Javascript that uses
functional idioms, outputs SVG and is chock full of contributions from
the community? No wonder it's so successful.

And yet, D3 code has a tendency to grow unwieldy very quickly. As
happens in SQL, the means of abstraction in the language (in our case
the library) are not central, and details just tend to grow and grow.

Let's see a simple example to illustrate this.

We begin with a simple toy problem, and then add a little complexity
to it to see what happens.

Say we're plotting the result of a race. We have a list of runners,
and for each of them we have a list of splits.

Take our first runner:

	{
		"name": "Alice",
		"nationality": "UK",
		"splits": [5.05, 11.32, 17.49, 24.22]
	}

Let's display a line with the runner's name, and circles representing
the splits.

<div id="single"></div>

<script>
	var runners = [{
		"name": "Alice",
		"nationality": "UK",
		"splits": [5.05, 11.32, 17.49, 29.22]
	}, {
		"name": "Bob",
		"nationality": "US",
		"splits": [6.05, 13.32, 19.49, 28.22]
	}, {
		"name": "Cal",
		"nationality": "Spain",
		"splits": [5, 10.32, 15.49, 22.22]
	}];

	var plot = d3.select("#single")
		.append("svg")
		.attr("width", 300)
		.attr("height", 20);

	plot.append("text")
		.text(runners[0].name)
		.attr("dy", 20);

	plot.selectAll("circle")
		.data(runners[0].splits)
		.enter()
		.append("circle")
			.attr("r", 5)
			.attr("cx", function(t) {return 100 + 5*t;})
			.attr("cy", 10)
			.style("stroke", "red")
			.style("fill", "red");
</script>

The way you do things in D3 is to build up your SVG graph by accretion. Start by
making a ```<div>``` in your HTML where you want the plot, and give it an ID so
that we can locate it, for example, "single". Then

	var plot = d3.select("#single")
		.append("svg")
		.attr("width", 300)
		.attr("height", 20);

This generates an empty ```<svg>``` element with the desired dimensions.
We want to print the runner's name, so we add a ```<text>``` element. Bear in
mind that our runner, Alice, lives in an array with other runners. She is
```runner[0]```.

	plot.append("text")
		.text(runners[0].name)
		.attr("dy", 20);

To the right of the runner's name, we want to mark splits by dots, placed at a
distance proportional to the times. <br/>
Here we start to use D3's core features: the binding operations, and selections.
We're going to select the existing circles in the plot, which is the empty
selection. To it we're going to bind the array with the splits, and for each
element of it, we shall add a red ```<circle>```.

	plot.selectAll("circle")
		.data(runners[0].splits)
		.enter()
		.append("circle")
			.attr("r", 5)
			.attr("cx", function(t) {return 100 + 5*t;})
			.attr("cy", 10)
			.style("stroke", "red")
			.style("fill", "red");

In the above, ```.data()``` binds the data to the selection, and ```.enter()```
return the elements of the data that were not already bound. You can learn about
D3's selections, and ```enter()```, ```exit()```, ```update()``` here.

Notice that the line ```.attr("cx", function(t) {return 100 +
5*t;})```, which takes a function. This is a pervasive idiom in D3,
and nothing more than the usual ```map``` from functional land.

Now, let's plot all the runners.

<div id="all"></div>

<script>
	var plotAll = d3.select("#all")
		.append("svg")
		.attr("width", 300)
		.attr("height", 90);

	var runLines = plotAll.selectAll("g")
		.data(runners)
		.enter()
		.append("g")
			.attr("transform",
				function(ignore, k) {
					return "translate(0," + (k*30) + ")";
				});

	runLines.append("text")
		.text(function(d) {return d.name;})
		.attr("dy", 20);

	runLines.selectAll("circle")
		.data(function(d) {return d.splits;})
		.enter()
		.append("circle")
			.attr("r", 5)
			.attr("cx", function(t) {return 100 + 5*t;})
			.attr("cy", 10)
			.style("stroke", "red")
			.style("fill", "red");
</script>

Not much needs to be done on top of what we were doing before for a single
runner. Firstly, we need to have fresh line to display each runner. Thanks to
SVG's ```<g>``` grouping element, and to SVG transforms, we can have a fresh
canvas, with its own coordinate system, for each runner.
In the below, ```plotAll``` is a selection we have defined on an identified
```<div>```, as we did before with ```plot```.

	var runLines = plotAll.selectAll("g")
		.data(runners)
		.enter()
		.append("g")
			.attr("transform",
				function(ignore, k) {
					return "translate(0," + (k*30) + ")";
				});

The use of ```.enter()``` and ```.data()``` is as previous. The only new thing
we have seen is that the signature ```function(ignore, k) {}```. This is fairly
standard in Javascript, and within D3. The second argument is the index of the
datum within the data array. We name the first argument ```ignore``` as a
convention to signal that the value of the element is not used, just its order.

It is easy to rewire the code that wrote the text and the splits. We just need
to pull the data we're interested in from each element.

	runLines.append("text")
		.text(function(d) {return d.name;})
		.attr("dy", 20);

Already you can start to appreciate D3's failing abstraction. In a typical
functional programming approach, we would figure out how to plot individual
runners, then iterate or ```map``` over an array to produce the full plot for
all runners.

With D3, the default way of operating does not abstract the individual. To make
it more visual: you can make 5 sandwiches by repeating the process of making a
single sandwich 5 times, or you can put 5 slices of bread on 5 plates, then 5
pieces of cheese on the 5 slices, then 5 portions of ham on the 5 pieces of
cheese, then 5 slices of bred on top of the portions of ham. This second way of
operation is the default D3 way.

Of course, this is just preference so far, but let's see where the low
abstraction can give rise to problems.
<br/>
In our runner array, we keep track of each runner's nationality,as we saw above
with Alice, who is British. Let's imagine we'd like to color the dots for the splits according
to nationality.

	UK ==> red
	US ==> blue
	Spain ==> green

In the fragment we use to plot the circles, it's not apparent how we could get the nationality,
given that we're iterating over the splits.

	runLines.selectAll("circle")
		.data(function(d) {return d.splits;})
		.enter()
		.append("circle")
			.attr("r", 5)
			.attr("cx", function(t) {return 100 + 5*t;})
			.attr("cy", 10)
			.style("stroke", "red")
			.style("fill", "red");

One thing we could do is assign a class on each of the ```<g>``` elements we defined before.
We add a line to the bottom of the definition of ```runLines```.

	var runLines = plotAll.selectAll("g")
		.data(runners)
		.enter()
		.append("g")
			.attr("transform",
				function(ignore, k) {
					return "translate(0," + (k*30) + ")";
				})
			.classed(function(d) {return d.nationality;}, true);

But this feels lazy. We now need supporting CSS directives, and we need to make sure the text
is still rendered black, even if the splits are in color.

We could think of handing, to each circle in our splits code, not just the time, but the nationality. That is, we could define a procedure to convert the data.

	function getNatlSplits(runner) ==> [..., {nationality, time_i}, ...]

Then

	runLines.selectAll("circle")
		.data(function(d) {return getNatlSplits(d);})
		.enter()

But this is even clunkier. We need an auxiliary function, and we're repeating the same nationality
for all those elements.

We can also traverse the data structures to get the parent of the splits. But isn't this already
getting more complicated than it should have been?

## What if?

The problem with coloring the circles, simple as it is, is an illustration of D3's trouble
with abstraction.

Now imagine that we had a procedure ```plotRunner(runner)``` that produced the line with 
the runner name and the splits. To color the circles based on nationality would be trivial.
If we simply iterated over ```runners``` and called ```plotRunner``` for each, the code would
need very little change.

<div id="singleAbstracted"></div>

<script>
	var plotAbstracted = d3.select("#singleAbstracted")
		.append("svg")
		.attr("width", 300)
		.attr("height", 20);

	var plotRunner(selection) {
		selection.append("text")
			.text(selection.data().name)
			.attr("dy", 20);

		plot.selectAll("circle")
			.data(runners[0].splits)
			.enter()
			.append("circle")
				.attr("r", 5)
				.attr("cx", function(t) {return 100 + 5*t;})
				.attr("cy", 10)
				.style("stroke", "red")
				.style("fill", "red");
	}
	
	plotRunner(plotAbstracted);
</script>