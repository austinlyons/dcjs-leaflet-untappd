# Beer I've Been Drinking
Learn [dc.js](https://dc-js.github.io/dc.js/), [Crossfilter](http://square.github.io/crossfilter/), and [Leaflet](http://leafletjs.com/) by visualizing [my](https://untappd.com/user/austinlyons) [Untappd](http://untappd.com) check-ins

# Demo
Try out the demo [here](https://austinlyons.github.io/dcjs-leaflet-untappd)

[![Demo
screenshot](img/demo.png)](https://austinlyons.github.io/dcjs-leaflet-untappd)

# Tutorial
Have you ever wished you could make a decent looking web-based data visualization? Maybe you're a UI/UX designer who wants to get beyond static wireframes, a graduate student who wants to impress your advising professor and research team, or an MBA who wants to go beyond Excel charts. This tutorial is for you.

I assume only rudimentary knowledge of HTML, CSS, and
JavaScript. We'll start with the HTML & CSS, but if you want to skip down to the fun
JavaScript stuff go right ahead. 

## Tools
We'll use CSS and HTML to position and style the data visualization. 
Instead of starting from scratch, let's leverage [Bootstrap's](http://getbootstrap.com/) existing
stylesheets and methodology to get a decent looking website in a short amount of time.

[dc.js](https://dc-js.github.io/dc.js/) is a great JavaScript library
that we'll use to create interactive charts and a corresponding table. 
It's built upon [d3](http://d3js.org/) and [crossfilter](http://square.github.io/crossfilter/), 
helps us to build simple charts quickly, and looks pretty good right out of the box. 

[Leaflet](http://leafletjs.com/) will be used to create an interactive map. 
Leaflet is easy to use and plays nicely with [Mapbox](https://www.mapbox.com/editor/#style) tiles,
which are a customizable and fun alternative to using Google Maps.

## Data
The visualization will use my actual Untappd check-in data. I
downloaded my beer drinking history via the Untappd API and put it into a JSON file
that you can access [here](untappd.json). 

![JSON screenshot](img/beer-json.png)

## Try it yourself
If you want to play with the code as we go, I recommend downloading
the source code from this GitHub repository, navigating to the directory where the files are
located, and kicking off Python's SimpleHTTP server so that you can see
your visualization in your own browser at http://localhost:8000 (If
this is new for you, see [this
link](https://github.com/lmccart/itp-creative-js/wiki/SimpleHTTPServer) for a bit more information). 

> In the terminal, navigate to the correct location and run the python server
![Terminal screenshot](img/navigate-and-serve.png)

> The demo will then be accessible in a browser at localhost:8000
![Running on localhost](img/running-on-localhost.png)

## HTML & CSS
The .css files can be found in the `css` folder. In addition to the
Bootstrap stylesheet, dc and leaflet come with their own default style
sheets that we'll include as well. Finally, we have our own CSS file
`main.css` for the little styling tweaks we'll make.

Let's start writing some HTML to include those CSS files. In
`index.html` we add the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>dc.js example - Untappd Data Visualization</title>
    <meta charset="UTF-8">
    <link rel="stylesheet" type="text/css" href="css/dc.css">
    <link rel="stylesheet" type="text/css" href="css/leaflet.css">
    <link rel="stylesheet" type="text/css" href="css/bootstrap.css">
    <link rel="stylesheet" type="text/css" href="css/main.css">
</head>
```

Next, we'll add all of the HTML scaffolding, adhering to the Bootstrap
[grid system](http://getbootstrap.com/css/#grid). 

### Tiny Bootstrap Grid Tutorial
> Skip this if you already grok Bootstrap

If you've never seen Bootstrap before, here's a brief summary of how it's grid layouts work.

In this "grid" way of doing things, we simply think of positioning our elements
one row at a time, where each row can be divvied up into 12 columns. 
So if you want to put 3 equally spaced items in a row, you just give
each item a width of 4 columns.

```html
<div class="container">
  <div class="row">
    <div class="col-md-4">
      Item 1
    </div>
    <div class="col-md-4">
      Item 2
    </div>
    <div class="col-md-4">
      Item 3
    </div>
  </div>
</div>
```

You can think of the grid system as a spreadsheet, so the HTML we just wrote 
would look like:
![Spreadsheet grid layout](img/grid-single-row.png)


Why col-md-4 instead of col-4? What's the "md" all about? It has to do with
responsive layouts, and "md" stands for medium. Check [the docs](http://getbootstrap.com/examples/grid/)
for more info.
> Here ends the tiny grid tutorial. 

### HTML Scaffolding

To achieve our final layout we will create four rows. The first is for our
header, the next is the row of pie charts and the map, the third row is
for the bar charts, and the final row is for the table. In each row 
we space things accordingly by choosing particular column widths. 

For illustration, this photo (forgive the lines that aren't at right
angles) shows where the rows are separated using teal lines and the
column widths using gray lines. As you can see, the second row uses
2 column widths for each pie chart and the remaining 6 columns for the map.
 
![Rows and columns](img/rows-and-columns.png)

In the "spreadsheet" analogy, this layout would look like this:
![Spreadsheet layout](img/grid-visualization-layout.png)

Below is all of the scaffolding HTML code. Notice the row and column spacing.
(There's a bit more in here too related to charts that we'll get into later.)

```html
<div class="container-fluid">
  <div class="row">
    <div class="col-xs-12 dc-data-count dc-chart" id="data-count">
      <h2>Beer History
        <small>
          <span class="filter-count"></span> selected out of <span class="total-count"></span> records |
           <a id="all" href="#">Reset All</a>
          </span>
        </small>
      </h1>
    </div>
  </div>
  <div class="row" id="control-row">
    <div class="col-xs-2 pie-chart">
      <h4>Year <small><a id="year">reset</a></small></h4>
      <div class="dc-chart" id="chart-ring-year"></div>
    </div>
    <div class="col-xs-2 pie-chart">
      <h4>Month <small><a id="month" href="#">reset</a></small></h4>
      <div class="dc-chart" id="chart-ring-month"></div>
    </div>
    <div class="col-xs-2 pie-chart">
      <h4>Day <small><a id="day">reset</a></small></h4>
      <div id="chart-ring-day" class="dc-chart"></div>
    </div>
    <div class="col-xs-6">
      <h4>Breweries</h4>
      <div id="map"></div>
    </div>
  </div>
  <div class="row">
    <div class="col-xs-6 col-md-3">
      <div class="dc-chart" id="chart-rating-count"></div>
    </div>
    <div class="col-xs-6 col-md-3">
      <div class="dc-chart" id="chart-community-rating-count"></div>
    </div>
    <div class="col-xs-6 col-md-3">
      <div class="dc-chart" id="chart-abv-count"></div>
    </div>
    <div class="col-xs-6 col-md-3">
      <div class="dc-chart" id="chart-ibu-count"></div>
    </div>
  </div>
  <div class="row">
    <div class="col-xs-12">
      <table class="table table-bordered table-striped" id="data-table">
        <thead>
          <tr class="header">
            <th>Brewery</th>
            <th>Beer</th>
            <th>Style</th>
            <th>My Rating</th>
            <th>Community Rating</th>
            <th>ABV %</th>
            <th>IBU</th>
          </tr>
        </thead>
      </table>
    </div>
  </div>
</div>
```

## JavaScript
Finally! Time for the interesting parts.

First we include a couple of JavaScript libraries and then start writing
custom code. I'll skip the map code right now and start with dc.js
related code.

```javascript
<script type="text/javascript" src="js/d3.js"></script>
<script type="text/javascript" src="js/crossfilter.js"></script>
<script type="text/javascript" src="js/dc.js"></script>
<script type="text/javascript" src="js/leaflet.js"></script>
<script type="text/javascript" src="js/underscore-min.js"></script>
<script type="text/javascript">
```

[Underscore](http://underscorejs.org/) is by no means necessary but I wanted to be the first to introduce you to it
if you've never seen it before. [Go read about it](http://underscorejs.org/). It's very useful.

### Parsing data
We begin by reading all of the JSON data from `untappd.json` using [`d3.json`](https://github.com/mbostock/d3/wiki/Requests#d3_json).
Before setting up any of the charting code, we preprocess the data we've
read so that it's in the format dc.js expects. We convert string
representation of numbers into actual numbers using the `+` operator
like this `d.count = +d.count`. We also round particular pieces of data
to the nearest 0.25, 0.5, or 10 to make the bar charts look better;
we are basically forcing the data into bins of a particular width. 
Finally, we use `d3.time.format` to pluck the values we care about (month,
day, etc) out of JavaScript Date objects.

```javascript
d3.json('untappd.json', function (error, data) {
  var beerData = data.response.beers.items;

  var fullDateFormat = d3.time.format('%a, %d %b %Y %X %Z');
  var yearFormat = d3.time.format('%Y');
  var monthFormat = d3.time.format('%b');
  var dayOfWeekFormat = d3.time.format('%a');

  // normalize/parse data so dc can correctly sort & bin them
  beerData.forEach(function(d) {
    d.count = +d.count;
    // round to nearest 0.25
    d.rating_score = Math.round(+d.rating_score * 4) / 4;
    d.beer.rating_score = Math.round(+d.beer.rating_score *4) / 4;
    // round to nearest 0.5
    d.beer.beer_abv = Math.round(+d.beer.beer_abv * 2) / 2;
    // round to nearest 10
    d.beer.beer_ibu = Math.floor(+d.beer.beer_ibu / 10) * 10;
    d.first_had_dt = fullDateFormat.parse(d.first_had);
    d.first_had_year = +yearFormat(d.first_had_dt);
    d.first_had_month = monthFormat(d.first_had_dt);
    d.first_had_day = dayOfWeekFormat(d.first_had_dt);
  });
```

### Crossfilter
[Crossfilter](http://square.github.io/crossfilter/) is the workhorse of the visualization. It does all of the grouping, aggregating, and filtering for us
and it does it all very quickly.
[Here's a nice
introduction](http://blog.rusty.io/2012/09/17/crossfilter-tutorial/) to bring you up to speed.

Now that we've cleaned up our data, we instantiate crossfilter and give
it our data.

```javascript
var ndx = crossfilter(beerData);
```

#### Dimensions
Next, we'll set up all of the
[dimensions](https://github.com/square/crossfilter/wiki/API-Reference#dimension). 
I think of these in my head as a list of x-axis values

```javascript
var yearDim  = ndx.dimension(function(d) {return d.first_had_year;}),
    // dc.pluck: short-hand for same kind of anonymous function we used for yearDim
    monthDim  = ndx.dimension(dc.pluck('first_had_month')),
    dayOfWeekDim = ndx.dimension(dc.pluck('first_had_day')),
    ratingDim = ndx.dimension(dc.pluck('rating_score')),
    commRatingDim = ndx.dimension(function(d) {return d.beer.rating_score;}),
    abvDim = ndx.dimension(function(d) {return d.beer.beer_abv;}),
    ibuDim = ndx.dimension(function(d) {return d.beer.beer_ibu;}),
    allDim = ndx.dimension(function(d) {return d;});
```

Take the rating dimension as an example. When we use this dimension to
create the chart, what we're doing is saying 
"for every item in our beer data, return the rating score as the x value
for this particular dimension". Notice that dc has a function `pluck` to make
this less verbose. 

Now how to get the corresponding y-values for each dimension?

#### Groups
[Groups](https://github.com/square/crossfilter/wiki/API-Reference#group-map-reduce) take
a dimension value as input and return an output value; they are the
mechanism by which you get a y-value for a particular x-value. If you're 
familiar with the map-reduce programming paradigm, that's what this
basically is. 

For each dimension in this simple visualization we are just plotting the number
of occurences on the y-axis, i.e. the number of 8% ABV beers I've had.
Counting the number of occurences is a very common thing to do so 
crossfilter has a built in function we'll use called `reduceCount`.


```javascript
  var all = ndx.groupAll();
  var countPerYear = yearDim.group().reduceCount(),
      countPerMonth = monthDim.group().reduceCount(),
      countPerDay = dayOfWeekDim.group().reduceCount(),
      countPerRating = ratingDim.group().reduceCount(),
      countPerCommRating = commRatingDim.group().reduceCount(),
      countPerABV = abvDim.group().reduceCount(),
      countPerIBU = ibuDim.group().reduceCount();
```

`reduceCount` is actually just a shorthand for the more flexible `reduce`.
`reduce` takes three parameters that tell crossfilter what to do whenever the group
is filtered. Specifically, what to do when data is added, removed, and
when the group is initialized.  A concrete example - in the ABV chart, 
if you zoom in to see only the 8% ABV beers, the crossfilter group 
needs to know how to remove the 5%, 6%, 7%, 9%, ... values. 

So using `reduce`, we would implement `countPerABV` as follows:

```javascript
var countPerABV = abvDim.group().reduce(
  function reduceAdd(p, v) {
    return p + 1;
  },
  function reduceRemove(p, v) {
    return p - 1;
  },
  function reduceInitial() {
    return 0;
  }
);
```

See the [annotated source](https://dc-js.github.io/dc.js/docs/stock.html#section-11)
 of dc.js' canonical example for advanced usage of `reduce`.

If you want to compute any functions for a particular dimension like mean or 
[standard deviation](https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance#Computing_shifted_data),
 you would need to use this reduce pattern. Or you could just use
[Reductio](https://github.com/esjewett/reductio), a library that makes
Crossfilter grouping easier and comes with implementations for common grouping
functions like standard deviation.

### Charts
The first thing we'll do is create all of the dc chart objects.
As we instantiate each chart object we'll pass in the CSS selector
of the corresponding HTML element that the chart will be drawn in.

```javascript
var yearChart   = dc.pieChart('#chart-ring-year'),
    monthChart   = dc.pieChart('#chart-ring-month'),
    dayChart   = dc.pieChart('#chart-ring-day'),
    ratingCountChart  = dc.barChart('#chart-rating-count'),
    commRatingCountChart  = dc.barChart('#chart-community-rating-count'),
    abvCountChart  = dc.barChart('#chart-abv-count'),
    ibuCountChart  = dc.barChart('#chart-ibu-count'),
    dataCount = dc.dataCount('#data-count')
    dataTable = dc.dataTable('#data-table');
```

Now we'll start configuring each individual chart, defining things
like width and height of the chart as well as associating each chart
with particular a dimension and group.

#### Year Chart
First we'll define the pie chart that will double as the control for 
filter all of the data by year. We've already positioned it via HTML to be the 
leftmost element in the second row.

```javascript
yearChart
    .width(150)
    .height(150)
    .dimension(yearDim)
    .group(countPerYear)
    .innerRadius(20);
```
Here we see the beauty of dc shine through in this very straightforward
definition. We use the year dimension and the count-per-year group
to define the data that drives the pie chart. Finally we set
a bit of aesthetics via the `innerRadius` method.

#### Month Chart
The month chart configuration is almost identical to the year chart,
but also passes a custom ordering function to `ordering` that 
instructs dc how to order the months when creating the pie chart.

```javascript
monthChart
    .width(150)
    .height(150)
    .dimension(monthDim)
    .group(countPerMonth)
    .innerRadius(20)
    .ordering(function (d) {
      var order = {
        'Jan': 1, 'Feb': 2, 'Mar': 3, 'Apr': 4,
        'May': 5, 'Jun': 6, 'Jul': 7, 'Aug': 8,
        'Sep': 9, 'Oct': 10, 'Nov': 11, 'Dec': 12
      };
      return order[d.key];
    });
```

Without the custom ordering function, the month and day charts
are sorted in alphabetical order as you move clockwise. 
![Out-of-order controls](img/calendar-out-of-order.png)
The year pie chart is already correctly sorted because 
we cast the year string to a number earlier when we parsed the data: 
`d.first_had_year = +yearFormat(d.first_had_dt);`

#### Day of the Week Chart
The day of the week pie chart is just like the month chart, except
that the custom ordering function is updated accordingly.

```javascript
dayChart
    .width(150)
    .height(150)
    .dimension(dayOfWeekDim)
    .group(countPerDay)
    .innerRadius(20)
    .ordering(function (d) {
      var order = {
        'Mon': 0, 'Tue': 1, 'Wed': 2, 'Thu': 3,
        'Fri': 4, 'Sat': 5, 'Sun': 6
      }
      return order[d.key];
    }
   );
```

#### Map
The next element in this row of HTML is the map, but let's stick with dc.js right
now and move on to the row of bar charts.

#### Bar Charts
The bar charts are nearly identical in their configuration. We'll start
with the chart that depicts the count of beers for each of my ratings.

```javascript
  ratingCountChart
      .width(300)
      .height(180)
      .dimension(ratingDim)
      .group(countPerRating)
      .x(d3.scale.linear().domain([0,5.2]))
      .elasticY(true)
      .centerBar(true)
      .barPadding(5)
      .xAxisLabel('My rating')
      .yAxisLabel('Count')
      .margins({top: 10, right: 20, bottom: 50, left: 50});
  ratingCountChart.xAxis().tickValues([0, 1, 2, 3, 4, 5]);
```

This chart definition is a bit more verbose than the pie chart.
We set the width, height, dimension, and group as usual. Then we 
tell dc what to use for the x-axis scale by using a [d3 quantitative
scale](https://github.com/mbostock/d3/wiki/Quantitative-Scales) with 
an input domain from 0 to 5.2. The rating data only ranges 
from 0 to 5, but the chart looks a little nicer if we set the 
range to be just a bit beyond 0 to 5. The rest of the methods are just
aesthetics, and the fastest way to understand them is to just play with
them - change the inputs and see what happens. 
`elasticY` tells the chart whether it should resize the y axis 
when data is filtered, the rest do what they say (center the bar, etc).

The next chart is the count of beers I've checked in grouped by the average
community rating for that beer. This chart is the essentially the 
same as the previous chart.

```javascript
commRatingCountChart
    .width(300)
    .height(180)
    .dimension(commRatingDim)
    .group(countPerCommRating)
    .x(d3.scale.linear().domain([0,5.2]))
    .elasticY(true)
    .centerBar(true)
    .barPadding(5)
    .xAxisLabel('Community rating')
    .yAxisLabel('Count')
    .margins({top: 10, right: 20, bottom: 50, left: 50});
commRatingCountChart.xAxis().tickValues([0, 1, 2, 3, 4, 5]);
```

Next up is the count-per-ABV chart. This chart is fun because we can use
it as a filter to see how I rated the especially alcoholic beers that I
drank. Unsurprisingly, I rated them especially well. You'll also notice
that the very alcoholic beers are mostly IPAs, Belgians, or Stouts. [Legend has 
it](http://www.northamericanbrewers.org/india_pale_ale.htm)
that India Pale Ales are very alcoholic by design. India Pale Ales
were created to survive the months-long 1700's journey from Britain to
India so additional hops were added to keep the beer flavorful. 
Since hops are bitter, extra sugar was also added offset the bitterness.
The yeast in beer convert sugar to alcohol, so more sugar means more alcohol. 
I'm not sure why Belgian beers are especially alcoholic; my best guess is that
many Belgian beers are [brewed by monks](https://en.wikipedia.org/wiki/Chimay_Brewery), 
so maybe God ordained beer to be delicious and alcoholic instead of flavorless and light.

Note that the only difference in the configuration of this chart is that
we use `d3.max` to calculate the maximum value of the domain. Again,
instead of going from 0 to the maximum value, we add a bit of padding on 
each side to make the chart look nicer.

```javascript
abvCountChart
    .width(300)
    .height(180)
    .dimension(abvDim)
    .group(countPerABV)
    .x(d3.scale.linear().domain([-0.2, d3.max(beerData, function (d) { return d.beer.beer_abv; }) + 0.2]))
    .elasticY(true)
    .centerBar(true)
    .barPadding(2)
    .xAxisLabel('Alcohol By Volume (%)')
    .yAxisLabel('Count')
    .margins({top: 10, right: 20, bottom: 50, left: 50});
```

The final bar chart is the number of beers I've had grouped by
[IBUs](http://beer.wikia.com/wiki/International_Bitterness_Units).
The only new line of code here is the use of
[xUnits](https://github.com/dc-js/dc.js/blob/master/web/docs/api-latest.md#xxscale---mandatory) 
`xUnits(function (d) { return 5;})`. Here we are just using fixed units
rather than the default `dc.units.integers` to make the chart look
nicer. 

```javascript
ibuCountChart
    .width(300)
    .height(180)
    .dimension(ibuDim)
    .group(countPerIBU)
    .x(d3.scale.linear().domain([-2, d3.max(beerData, function (d) { return d.beer.beer_ibu; }) + 2]))
    .elasticY(true)
    .centerBar(true)
    .barPadding(5)
    .xAxisLabel('International Bitterness Units')
    .yAxisLabel('Count')
    .xUnits(function (d) { return 5;})
    .margins({top: 10, right: 20, bottom: 50, left: 50});
```

#### Data Table
To this point we have a bunch of charts that can all be interacted with
to filter the set of all data down by year, month, day of the week,
rating, etc. Now we will use dc to add a [data table](https://github.com/dc-js/dc.js/blob/master/web/docs/api-latest.md#data-table-widget)
so we can see a list of those filtered results.

Here's what our HTML for the table looked like. 
```html
<table class="table table-bordered table-striped" id="data-table">
  <thead>
    <tr class="header">
      <th>Brewery</th>
      <th>Beer</th>
      <th>Style</th>
      <th>My Rating</th>
      <th>Community Rating</th>
      <th>ABV %</th>
      <th>IBU</th>
    </tr>
  </thead>
</table>
```

You'll notice that we create all of the column headings in the HTML to match
the order of the accessor functions we pass into the `columns` method.
```javascript
.dimension(allDim)
.group(function (d) { return 'dc.js insists on putting a row here so I remove it using JS'; })
.size(100)
.columns([
  function (d) { return d.brewery.brewery_name; },
  function (d) { return d.beer.beer_name; },
  function (d) { return d.beer.beer_style; },
  function (d) { return d.rating_score; },
  function (d) { return d.beer.rating_score; },
  function (d) { return d.beer.beer_abv; },
  function (d) { return d.beer.beer_ibu; }
])
.sortBy(function (d) { return d.rating_score; })
.order(d3.descending)
.on('renderlet', function (table) {
  // each time table is rendered remove nasty extra row dc.js insists on adding
  table.select('tr.dc-table-group').remove();
});
```

The `group` method call here is weird. 
The data table object expects group to be
[given a function that returns a
string](https://dc-js.github.io/dc.js/docs/stock.html#section-92), 
and then injects the string into the first row of the table.
![Yuck](img/table-with-first-row.png)
I'm not a fan of how it looks, so we'll use a
[renderlet](https://github.com/dc-js/dc.js/blob/master/web/docs/api-latest.md#renderletrenderletfunction) 
to remove that row from the DOM every time
the data table is redrawn. 

Renderlets are a great way to do additional
post-processing of a chart such as SVG manipulations, triggering other
chart events, or even interacting with other JavaScript objects as
we'll do later with the map.

Also, notice we [use d3 to select and
operate](https://github.com/mbostock/d3/wiki/Selections) on DOM elements 
instead of using jQuery: `table.select('tr.dc-table-group').remove();`

We configure the table to be sorted by rating score descending, using
`dc.pluck` as a short-hand for `function(d) { return d.rating_score; }`.

#### Data Count
We use the [data count
widget](https://github.com/dc-js/dc.js/blob/master/web/docs/api-latest.md#dc.dataCount)
to dynamically display the number of selected check-ins and the total number
of check-ins at the top of the visualization. 

```html
<div class="col-xs-12 dc-data-count dc-chart" id="data-count">
  <h2>Beer History
    <small>
      <span class="filter-count"></span> selected out of <span class="total-count"></span> records |
       <a id="all" href="#">Reset All</a>
      </span>
    </small>
  </h2>
</div>
```

```javascript
dataCount
    .dimension(ndx)
    .group(all);
```

#### Filters
The pie charts and bar charts are used as controls to filter the
resulting dataset, allow us to filter the dataset by month
and so forth. For our visualization to be usable, we need to 
give users the ability to turn off the filters after we've turned them
on.

To do this, we'll add some "reset" links in the HTML and then register
click handlers to reset the respective charts via
[`filterAll`](https://github.com/dc-js/dc.js/blob/master/web/docs/api-latest.md#filterall).

For each chart we'll add reset links like this:

```html
<div class="col-xs-2 pie-chart">
  <h4>Year <small><a id="year">reset</a></small></h4>
  <div class="dc-chart" id="chart-ring-year"></div>
</div>
```

Then sprinkle in a bit of JavaScript for the click handlers
and we are good to go:

```javascript
d3.selectAll('a#all').on('click', function () {
  dc.filterAll();
  dc.renderAll();
});

d3.selectAll('a#year').on('click', function () {
  yearChart.filterAll();
  dc.redrawAll();
});

d3.selectAll('a#month').on('click', function () {
  monthChart.filterAll();
  dc.redrawAll();
});

d3.selectAll('a#day').on('click', function () {
  dayChart.filterAll();
  dc.redrawAll();
});
```

#### Render!
Now that we've written all of the HTML, set up the crossfilter groups
and dimensions, and specified our dc charts it's showtime! A single 
line of code makes the magic happen:
[`dc.renderAll();`](https://github.com/dc-js/dc.js/blob/master/web/docs/api-latest.md#dcrenderallchartgroup)

### Leaflet
[Leaflet](http://leafletjs.com/) is an awesome library for making 
interactive maps. Since we have some empty real estate 
at the top right of the screen and
coordinates for the brewery that made each beer I checked-in, 
let's add a Leaflet map to our visualization. 

We included leaflet.js already:

```html
<script type="text/javascript" src="js/leaflet.js"></script>
```

and created a placeholder div for the map: `<div id="map"></div>`


Now let's instantiate the map:
```javascript
var map = L.map('map');
```

Whenever we filter or reset our dataset we'll create a
[marker](http://leafletjs.com/reference.html#marker) 
for each brewery and add it to a [feature
group](http://leafletjs.com/reference.html#featuregroup), which we will then
add to the map. Let's instantiate the feature group:

```javascript
var breweryMarkers = new L.FeatureGroup();
```

We'll use [Mapbox](https://www.mapbox.com/) tiles as the [tile
layer](https://www.mapbox.com/developers/api/maps/) for our map. If you
want to use Mapbox too ([you can use different tile providers if you'd
prefer](http://leaflet-extras.github.io/leaflet-providers/preview/)) then you'll 
need to [create a free Mapbox account](https://www.mapbox.com/signup/) 
and use your own id and [access
token](https://www.mapbox.com/help/define-access-token/).

```javascript
L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token={accessToken}', {
  id: 'mapbox.id.goes.here',
  accessToken: 'mapbox.access.token.goes.here',
  maxZoom: 16
} ).addTo(map);
```

Now that we've set up the map, let's write code that will add markers to
the map every time the data is filtered or reset. To do this we'll
put the code in the data table renderlet so that it will be run 
every time the data table is rerendered. 
Here's all the code, but let's walk through it.

```javascript
.on('renderlet', function (table) {
  breweryMarkers.clearLayers();
  _.each(allDim.top(Infinity), function (d) {
    var loc = d.brewery.location;
    var name = d.brewery.brewery_name;
    var marker = L.marker([loc.lat, loc.lng]);
    marker.bindPopup("<p>" + name + " " + loc.brewery_city + " " + loc.brewery_state + "</p>");
    breweryMarkers.addLayer(marker);
  });
  map.addLayer(breweryMarkers);
  map.fitBounds(breweryMarkers.getBounds());
});
```

First we'll clear the map by clearing the brewery markers 
feature group. 

`breweryMarkers.clearLayers();`

If we didn't do this, we'd be adding 
duplicate markers to the map every time the renderlet runs.

Next we'll use [underscore.js](http://underscorejs.org/) to loop
through each item in the (possibly filtered) dataset. 
Again, it's not necessary to use underscore.js but it is handy.
We retrieve the data from the "all" dimension using
[top](https://github.com/square/crossfilter/wiki/API-Reference#dimension_top):

`_.each(allDim.top(Infinity), function (d) {`

Then we'll grab the location and name from the brewery and create a
marker:

```javascript
var loc = d.brewery.location;
var name = d.brewery.brewery_name;
var marker = L.marker([loc.lat, loc.lng]);
```

Next we'll tell each marker what text to display when it's clicked on:

```javascript
marker.bindPopup("<p>" + name + " " + loc.brewery_city + " " + loc.brewery_state + "</p>");
```

When the loop finishes, we'll add the markers to the map: 
```javascript
map.addLayer(breweryMarkers);
```

Finally, let's zoom the map in as far as possible while still showing
all markers on the map:

```javascript
map.fitBounds(breweryMarkers.getBounds());
```

That's it for map code. Pretty simple.

## End
That's the end of this tutorial. If you see mistakes, have questions,
comments etc send me an email or a pull request.  

## TODOs
Other things that could be added to this example
* Use map as a filter. For example, zoom the map and then click a
  link to filter the dataset to only what is shown in the map
* New, more interesting charts such as rating/count per style of beer
* Sortable tables
* Clicking a row of the table causes brewery marker on map to pop up
* Get Foursquare location of where I checked in each beer and add to
  map. Then allow for toggling of "where I drank" layer from "where it's
  made" layer. This could allow for filtering data by "where I drank"
* Add reset links for the bar charts
* Create a column in the table with a thumbnail of each beer
