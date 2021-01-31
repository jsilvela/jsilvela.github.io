+++
categories = []
date = "2016-12-02T21:45:19+01:00"
tags = []
title = "D3’s Messy Abstraction Path"

+++

<script src="https://d3js.org/d3.v4.js"></script>

D3 is a wonderful idea. A graphing library in Javascript that uses
functional programming idioms, outputs SVG and is chock full of contributions from
the community. No wonder it’s so popular.

And yet, D3 code has a tendency to grow unwieldy very quickly. As
happens in SQL, the means of abstraction in the language (in our case
in the library) are obscure, and details in “normal” code tend to pile
up.

(Note that in SQL, the `WITH` statement allows abstraction and composition nicely, but a lot of people, or at least a lot of SQL codebases, do not leverage it.)

## D3’s normal abstraction path: low abstraction

We begin with a simple toy problem, and then add a little complexity
to see how D3 copes.

Say we’re plotting the result of a race. We have a list of runners,
and for each of them we have a list of splits.

Take our first runner:

``` javascript
{
    "name": "Alice",
    "nationality": "UK",
    "splits": [5.05, 11.32, 17.49, 29.22]
}
```

Let’s display a line with the runner’s name, and circles representing
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
        "splits": [6.05, 13.32, 23.49, 32.22]
    }, {
        "name": "Carlos",
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

The way you do things in D3 is to build up your SVG graph by
accretion. Start by putting a `<div>` in your HTML where you want
the plot to render, and give it an ID so that we can locate it, for example,
“single”. Then

``` javascript
var plot = d3.select("#single")
    .append("svg")
    .attr("width", 300)
    .attr("height", 20);
```

This generates an empty `<svg>` element with the desired
dimensions.  We want to show the runner’s name, so we add a
`<text>` element. Bear in mind that our runner, Alice, lives in an
array with other runners. She is `runner[0]`.

``` javascript
plot.append("text")
    .text(runners[0].name)
    .attr("dy", 20);
```

To the right of the runner’s name, we want to mark splits by dots,
placed at a distance proportional to the times. <br/> Here we start to
use D3’s core features: the binding operations, and selections.  We’re
going to select all the existing circles in the plot. There are none
at this point; we’re doing this to get a **selection**, which is the
main data type that D3 deals with. We now bind, to this empty selection, the
array with the splits, and for each split, we shall add a red
`<circle>`.

``` javascript
plot.selectAll("circle")
    .data(runners[0].splits)
    .enter()
    .append("circle")
        .attr("r", 5)
        .attr("cx", function(t) {return 100 + 5*t;})
        .attr("cy", 10)
        .style("stroke", "red")
        .style("fill", "red");
```

In the above, `.data()` binds the data to the selection, and
`.enter()` returns the elements of the data array that were not already
bound — i.e. all in our case, since the selection was empty. You can learn about D3’s `selection`s, and `enter()`,
`exit()`, `update()` [in this D3
guide](https://bost.ocks.org/mike/selection/).

Note the line `.attr("cx", function(t) {return 100 + 5*t;})`,
which takes a function as argument. This idiom is used widely in D3. It’s nothing more than the usual `map` from functional land; a
function being mapped over the splits array that we bound previously with
the `.data()` call.

Now, let’s plot all the runners.

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

Not much needs to be done on top of what we were doing before for a
single runner. Firstly, we need to have fresh line to display each
runner. Thanks to SVG’s `<g>` grouping element, and to SVG
transforms, we can have a fresh canvas, with its own coordinate
system, for each runner.  In the code below, `plotAll` is an empty `<svg>` element we have defined on an identified `<div>`, as we did before with
`plot` in the single runner case.

``` javascript
var runLines = plotAll.selectAll("g")
    .data(runners)
    .enter()
    .append("g")
        .attr("transform",
            function(ignore, k) {
                return "translate(0," + (k*30) + ")";
            });
```

The only novelty here is the signature `function(ignore, k)
{}`. This is fairly standard in Javascript, and within D3. The
second argument is the index of the datum within the data array. We
name the first argument `ignore` as a convention to signal that
in this case, the value of the argument is not used within the function body.

It is easy to repurpose the code that rendered the text and the splits. We
just need to pull the data we’re interested in from each
runner. <br/> We can see here the rendering of the runner names. The
rendering of the splits would work in the same way.

``` javascript
runLines.append("text")
    .text(function(d) {return d.name;})
    .attr("dy", 20);
```

Already you may have started to appreciate D3’s failing abstraction. In a
typical functional programming approach, we would figure out how to
plot individual runners, then iterate or `map` over an array to
produce the full plot for all runners.

In D3, the default way of operating does not abstract the
individual. To make it more visual: you can make 5 sandwiches by
repeating the process of making a single sandwich 5 times, or you can
put 5 slices of bread on 5 plates, then 5 pieces of cheese on the 5
slices, then 5 portions of ham on the 5 pieces of cheese, then 5
slices of bred on top of the portions of ham. This second way of
operation is the default D3 way.

## Problems due to low abstraction

This is working so far, but let’s see where the low
abstraction can give rise to problems.  <br/> In our runner array, we
keep track of each runner’s nationality, as we saw above with Alice,
who is British. Let’s imagine we’d like to color the dots for the
splits according to the runner’s nationality.

``` javascript
UK ==> blue
US ==> red
Spain ==> green
```

In the fragment we use to plot the circles, it’s not apparent how we
could get the nationality, given that we’re iterating over the splits.

``` javascript
runLines.selectAll("circle")
    .data(function(d) {return d.splits;})
    .enter()
    .append("circle")
        .attr("r", 5)
        .attr("cx", function(t) {return 100 + 5*t;})
        .attr("cy", 10)
        .style("stroke", "red")
        .style("fill", "red");
```

One thing we could do is assign a CSS class on each of the `runLines` we defined before.  We would add a line to the bottom of the
definition of `runLines`.

``` javascript
var runLines = plotAll.selectAll("g")
    .data(runners)
    .enter()
    .append("g")
        .attr("transform",
            function(ignore, k) {
                return "translate(0," + (k*30) + ")";
            })
        .classed(function(d) {return d.nationality;}, true);
```

But this feels lazy. We now need supporting CSS directives, and we
need to make sure the text is still rendered black, even if the splits
are in color.

We could think of handing, to each circle in our splits code, not just
the time, but the nationality. That is, we could define a procedure to
convert the data.

``` javascript
function getNatlSplits(runner) ==> [..., {nationality, time_i}, ...]
```

Then do

``` javascript
runLines.selectAll("circle")
    .data(function(d) {return getNatlSplits(d);})
    .enter()
    ...
```

But this is even clunkier. We need an auxiliary function, and we’re
repeating the same nationality for all those elements. And does the
`{nationality, time}` pair have any meaning outside of making our code work?

The problem here is that, in iterating over the `splits`, we have
lost access to the *parent* data that contained
`nationality`. <br/> We would solve this problem quite easily if
we could keep the parent data in scope.

## Abstraction the D3 way: too magical

The problem with coloring the circles, simple as it is, is an
illustration of D3’s trouble with abstraction.

Now imagine that we had a procedure `plotRunner(runner)` that
produced the line with the runner name and the splits. To color the
circles based on nationality would be trivial. The function parameter
would allow us access to both `splits` and `nationality` in the function body.

If we simply iterated over `runners` and called `plotRunner`
for each, the code would need very little change. But `plotRunner`
cannot function in a vacuum. It needs to do either of:

* Output an SVG snippet, to be aggregated by a higher level function
that generates the full SVG.
* Receive an *entry point* into the overall SVG into which it will
add its content.

The D3 way is the second. And D3 already has a handy type for entry
points into SVG: selections.

Let’s write our `plotRunner` function, and for the moment let’s
not worry about how we get the selection into it. Let’s just assume
it’s there, and call it `seln`.

``` javascript
var plotRunner = function(runner) {
    // NOTE: incomplete. Future version to explain
    // how we get the selection "seln"

    seln.append("text")
        .text(runner.name)
        .attr("dy", 20);

    seln.selectAll("circle")
        .data(runner.splits)
        .enter()
        .append("circle")
            .attr("r", 5)
            .attr("cx", function(t) {return 100 + 5*t;})
            .attr("cy", 10)
            .style("stroke", "red")
            .style("fill", "red");
}
```

This is pretty much what we did previously, but there is one crucial
difference: thanks to the function argument `runner`, we now have
access to the parent data within the function body.

Let’s go back to our color-by-nationality problem. We just need to
codify the color assignments, and modify `plotRunner` to leverage
them:

``` javascript
var nationalColor = {
    "US": "red",
    "UK": "blue",
    "Spain": "green"};

var plotRunner = function(runner) {
    ...
    ...
    seln.selectAll("circle")
        ...
        ...
            .style("stroke", nationalColor[runner.nationality])
            .style("fill", nationalColor[runner.nationality]);
}
```

Easy as pie.

We need to hook this up. `plotRunner`, as its name implies, plots
a single runner.  As we saw in previous examples, selections in D3 are arrays.  How do we iterate through them?

D3 has an operator `.each()` to iterate over selections. It
takes a function argument that can itself have either one argument
(the datum bound to the sub-selection), or two arguments (the datum, and the
ordinal of the sub-selection within the selection).<br/> There is one
more piece of magic to `.each()`. Within the function passed to
it, `this` will be the context of the current DOM element. We can
get the current selection with `d3.select(this)`.

So, the `plotRunner` function will be called from an `.each()`
loop, and we can now fill in the part we had left undefined.

``` javascript
var plotRunner = function(runner) {

    var seln = d3.select(this);

    seln.append("text")
        ...
        ...
```

Now we need to build the selection we will iterate over. We have seen
this before. We bind the `runners` array to an empty selection,
append `<g>` elements for each runner, and translate vertically.

``` javascript
var runnerLines = runnersFinal.selectAll("g")
    .data(runners)
    .enter()
    .append("g")
        .attr("transform",
            function(ignore, k) {
                return "translate(0," + (k*30) + ")";
            }
        );
```

And, finally, we tie it all together.

``` javascript
runnerLines.each(plotRunner);
```

There we go.

<div id="allAbstracted"></div>

<script>
    var nationalColor = {
        "US": "red",
        "UK": "blue",
        "Spain": "green"};

    var plotRunner = function(runner) {
        var seln = d3.select(this);

        seln.append("text")
            .text(runner.name)
            .attr("dy", 20);

        seln.selectAll("circle")
            .data(runner.splits)
            .enter()
            .append("circle")
                .attr("r", 5)
                .attr("cx", function(t) {return 100 + 5*t;})
                .attr("cy", 10)
                .style("stroke", nationalColor[runner.nationality])
                .style("fill", nationalColor[runner.nationality]);
    }

    var runnersFinal = d3.select("#allAbstracted")
        .append("svg")
        .attr("width", 300)
        .attr("height", 90);

    var runnerLines = runnersFinal.selectAll("g")
        .data(runners)
        .enter()
        .append("g")
            .attr("transform",
                function(ignore, k) {
                    return "translate(0," + (k*30) + ")";
                }
            );

    runnerLines.each(plotRunner);
</script>

Now, all this is very magical, in a bad way. Capturing selections
inside of `.each()` by using `this` is flimsy.  And so, we
rendered the plot by calling

``` javascript
runnerLines.each(plotRunner);
```

but the following will not work:

``` javascript
runnerLines.each(function(d) {plotRunner(d);});
```

How can this be? `this` depends on context, and in the second
version, `plotRunner` gets it from the enclosing `function (d) {` rather than from `.each()`. There goes your referential transparency.

D3 offers a lot of power, but its abstraction mechanism is somewhat
finicky.  I recommend that you use it, though. For all but simple
plots, the functional decomposition can clarify your code and make it
more flexible.

I wish there were less magic involved. *React*, for instance, achieves
a very coherent abstraction and is also written in Javascript. I don’t
know enough JS yet, but I’m getting interested in D3’s internals.

<hr/>
<hr/>

Final code listing:

``` javascript
var nationalColor = {
    "US": "red",
    "UK": "blue",
    "Spain": "green"};

var plotRunner = function(runner) {
    var seln = d3.select(this);

    seln.append("text")
        .text(runner.name)
        .attr("dy", 20);

    seln.selectAll("circle")
        .data(runner.splits)
        .enter()
        .append("circle")
            .attr("r", 5)
            .attr("cx", function(t) {return 100 + 5*t;})
            .attr("cy", 10)
            .style("stroke", nationalColor[runner.nationality])
            .style("fill", nationalColor[runner.nationality]);
}

var runnersFinal = d3.select("#allAbstracted")
    .append("svg")
    .attr("width", 300)
    .attr("height", 90);

var runnerLines = runnersFinal.selectAll("g")
    .data(runners)
    .enter()
    .append("g")
        .attr("transform",
            function(ignore, k) {
                return "translate(0," + (k*30) + ")";
            }
        );

runnerLines.each(plotRunner);
```
