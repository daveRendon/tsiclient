# TsiClient.ux Reference

## Components

### Line Chart

The line chart is used for rendering groups of time series.  A line chart can be created as follows...

```js
var tsiClient = new TsiClient();
var lineChart = new tsiClient.ux.LineChart(document.getElementById('chart'));
lineChart.render(data, chartOptions, chartDataOptionsArray);
```

where the parameter ``data`` follows the shape definied in [Chart Data Shape](#chart-data-shape), ``chartOptions`` contain some subset of the properties defined in [Chart Options](#chart-options), and ``chartDataOptionsArray`` is an array of objects that contain a subset of properties definied in [Chart Data Options](#chart-data-options).

By specifying [``events`` and ``states``](#line-chart-events-and-states-data-shape) in ``chartOptions``, you can create charts with multiple series types like in [this example](https://tsiclientsample.azurewebsites.net/examples/noauth/multipleseriestypes.html)

### Bar Chart

Bar charts are created in the same way as the line chart, and they take the same options and data shapes.  

```js
var tsiClient = new TsiClient();
var barChart = new tsiClient.ux.BarChart(document.getElementById('chart'));
barChart.render(data, chartOptions, chartDataOptionsArray);
```

Bar charts have a slider to step through the timestamps of time series.

### Pie Chart

Pie charts are created in the same way as bar charts, and also have a slider for stepping through time.

```js
var tsiClient = new TsiClient();
var pieChart = new tsiClient.ux.PieChart(document.getElementById('chart'));
pieChart.render(data, chartOptions, chartDataOptionsArray);
```

### Heatmap

Heatmaps are created using the same pattern as line charts, however, [searchSpan](#search-span-object) *must* be supplied for each element of chartDataOptionsArray.

```js
var tsiClient = new TsiClient();
var heatmap = new tsiClient.ux.Heatmap(document.getElementById('chart'));
heatmap.render(data, chartOptions, chartDataOptionsArray);
```

### Events Grid

A grid of events can be used to show a generic array of JSON in a scalable way.  Usage is as follows...

```js
var tsiClient = new TsiClient();
var eventsTable = tsiClient.ux.EventsTable(document.getElementById('chart'));
eventsTable.render(events, chartOptions)
```

where ``events`` is an array of flat JSON objects, with an example shape like this...

```js
[
    {
        timestamp: '2017-04-14T13:00:00Z',
        temperature: 27.5
    },
    {
        timestamp: '2017-04-14T13:01:00Z',
        pressure: 27.5
    }
]
```


## Classes

### AggregateExpression

AggregateExpressions are used to represent API queries against a Time Series Insights S SKU.  They include a method for transforming the object to query the API called ``toTsx()``, when transformed after an API call they become a data group as described in [Chart Data Shape](#chart-data-shape), and they can be used as [Chart Data Options](#chart-data-options).  Additional Chart Data Options can be specified as the final parameters, with supported properties defined [here](#chart-data-options).  Their usage is shown in [the basic charts example that uses the TSI platform](https://tsiclientsample.azurewebsites.net/examples/withplatform/basiccharts.html), or as follows...

```js
var aggregateExpression = new tsiClient.ux.AggregateExpression(
    {predicateString: "Factory = 'Factory1'"}, // filtering expression for data
    {property: 'Pressure', type: "Double"}, // measure column
    ['avg', 'min', 'max'], // desired measure types
    { from: startDate, to: endDate, bucketSize: '2m' }, // search span object
    {property: 'Station', type: 'String'},  // split by column
    {color: '#FF8C00', alias: 'Factory1Pressure') // ChartDataOptions
    );

// later, to call the API and visualize the result
tsiClient.server.getAggregates(token, '10000000-0000-0000-0000-100000000108.env.timeseries.azure.com', [aggregateExpression.toTsx())
    .then(function(result){
        var transformedResult = tsiClient.ux.transformAggregatesForVisualization(result, [aggregateExpression]);
        lineChart.render(transformedResult, null, [aggregateExpression]);
    })
```

### TsqExpression

TsqExpressions are used to represent API queries against a Time Series Insights PAYG SKU.  They include a method for transforming the object to query the API called ``toTsq()``, when transformed after an API call they become a data group as described in [Chart Data Shape](#chart-data-shape), and they can be used as [Chart Data Options](#chart-data-options). An example of TsqExpressions in use can be found [here](https://tsiclientsample.azurewebsites.net/examples/withplatform/PAYG.html). They are used as follows...

```js
var tsqExpression = new tsiClient.ux.TsqExpression(
    {timeSeriesId: ['df4412c4-dba2-4a52-87af-780e78ff156b']}, // time series instance json
    {AvgTemp: {
        kind: 'numeric',
        value: {tsx: '$event.temp.Double'},
        filter: null,
        aggregation: {tsx: 'avg($value)'}
    }}, // variable json
    { from: startDate, to: endDate, bucketSize: '6h' }, // search span object
    {color: '#60B9AE', alias: 'AvgTemp'} // ChartDataOptions
    );

// later, to call the API and visualize the result
tsiClient.server.getTsqResults(token, '10000000-0000-0000-0000-100000000109.env.timeseries.azure.com', [tsqExpression.toTsq()])
    .then(function(result){
        var transformedResult = tsiClient.ux.transformTsqResultsForVisualization(result, [tsqExpression]);
        lineChart.render(transformedResult, null, [tsqExpression]);
    });
```

## Functions

### transformAggregatesForVisualization

The shape of results returned by the Time Series Insights do not generally match [Chart Data Shape](#chart-data-shape).  To transform API results from the aggregates API of an S SKU, we use transformAggregatesForVisualization as follows...

```js
tsiClient.server.getAggregates(token, '10000000-0000-0000-0000-100000000108.env.timeseries.azure.com', [aggregateExpression.toTsx())
    .then(function(result){
        var transformedResult = tsiClient.ux.transformAggregatesForVisualization(result, [aggregateExpression]);
        // transformedResult is an array of data groups with time series and is suitable for visualization
        lineChart.render(transformedResult, null, [aggregateExpression]);  
    })
```

### transformTsqResultsForVisualization

To transform API results from the timeseries API of a PAYG SKU, we use transformTsqResultsForVisualization as follows...

```js
tsiClient.server.getTsqResults(token, '10000000-0000-0000-0000-100000000109.env.timeseries.azure.com', [tsqExpression.toTsq()])
    .then(function(result){
        var transformedResult = tsiClient.ux.transformTsqResultsForVisualization(result, [tsqExpression]);
        // transformedResult is an array of data groups with time series and is suitable for visualization
        lineChart.render(transformedResult, null, [tsqExpression]);
    });
```

## Additional References

### Chart Data Shape

Chart data is generally represented as a set of **groups** that each contain **time series**. Data for the Line, Pie, Bar, and Heatmap charts follows the following shape convention

```js
[
    {
        Factory1: // a name for this group
        {
            Station1:  // a name for this particular time series
            {
                '2019-02-07T21:00:00.000Z': {'temperature': 26.5}, // an object with keys representing ISO strings for time
                '2019-02-07T21:00:01.000Z': {'temperature': 24.3}, // with a value of a javascript object, whose keys represent value names
                ... 
            },
            Station2: {...},
            ...
        }
    },
    {
        Factory2:
        {
            Station1: {...},
            ...
        }
    },
    ...
]
```

The above structure is shown in the [Basic Charts Example](https://tsiclientsample.azurewebsites.net/examples/noauth/basiccharts.html), and the associated [Code](../pages/examples/noauth/basiccharts.html).

### Chart Options

Chart options are generally the second parameter to a component ``render`` method.  They allow users to change view properties for the chart, like theme, legend layout, etc, and can be explored in the [Chart Options Example](https://tsiclientsample.azurewebsites.net/examples/noauth/chartOptions.html). 

```js
lineChart.render(data, {theme: 'light', tooltip: true});
//                     ^this parameter is chartOptions
```

The most common available parameters for chart options are as follows (bold options represent default values if the option is not provided)...

|Property Name|Type|Value Options|Description|
|-|-|-|-|
|brushContextMenuActions|Array<any>|**null**, Array&lt;[brushContextMenuAction](#brush-context-menu-actions)&gt;|An array of objects defining brush actions
|events|Array<any>|**null**, Array&lt;[Event](#line-chart-events-and-states-data-shape)glt;|events passed into the linechart, an array of discrete time events|
|grid|boolen|**false**,true|If true, add accessible grid button to the ellipsis menu|
|includeDots|boolean|**false**|If true, the linechart plots dots for values|
|includeEnvelope|boolean|**false**|If true, include an area showing min/max boundaries in the line chart|
|interpolationFunction|string|**''**, 'curveLinear'|Name for interpolation function used for line chart lines|
|legend|string|**'shown'**,'compact','hidden'|Legend layout|
|noAnimate|boolean|**false**,true|If true, uppresses animated chart transitions|
|offset|any|**0**, -120, 'America/Los_Angeles'|Offset for all timestamps in minutes from UTC, or a timezone supported by moment.js|
|stacked|boolean|**false**|If true, stack bars in barchart|
|states|Array<any>|**null**, Array&lt;[State](#line-chart-events-and-states-data-shape)&gt;|An array of time range bound states passed into the linechart|
|theme|string|**'dark'**, 'light'|Component color scheme|
|timestamp|string|**null**,'2017-04-19T13:00:00Z'|If an ISO string, sets the slider in the bar or pie chart to the specified timestamp|
|tooltip|boolean|**false**,true|If true, display tooltip on hover over a value element|
|yAxisState|string|**'stacked'**, 'shared', 'overlap|State of the y axis in line chart|
|yExtent|[number, number]|**null**, [minValue, maxValue]|A minimum and maximum for the extent of the yAxis for this line chart, when the yAxisState is set to shared|

For very specific user interactions, check out [additional chart options](#additional-chart-options)

### Chart Data Options

Chart data options are generally the final parameter for an AggregateExpression(#aggregateexpression) or TsqExpression(#tsqexpression), or third parameter to a component ``render`` method.  In render, chartDataOptions is an array that allows users to define specific properties of the **groups** of data in the chart, like alias, color, etc.

```js
// data is an array of length 2
lineChart.render(data, chartOptions, [{alias: 'myFaveLines', color: 'red'}, {alias: 'worseLines', color: 'green'}]);
```

The available parameters for chart data options are as follows...

|Property Name|Type|Value Options|Description|
|-|-|-|-|
|color|string|'#4286f4'|The color of this group in a component|
|alias|string|'Factory1'|The display name for this group|
|contextMenu|Array&lt;[groupContextMenuAction](#group-context-menu-actions)&gt;|[]|Actions to take on context menu click on a group, or time series|
|searchSpan|[searchSpanObject](#search-span-object)|null|Specifies search span for this group|
|measureTypes|Array&lt;string>|['min', 'avg', max']|The measure properties specified in the time series of this group|
|interpolationFunction|string|'curveStep'|If 'curveStep' is set, step interpolation is used|
|includeEnvelope|boolean|true|If true, and a data group has measure types ['min', 'avg', max'], a shadow will be drawn to show the range of values|
|includeDots|boolean|true|If true, draw circles for each value in the group|
|yExtent|[number, number]|[0,400]|A minimum and maximum for the extent of the yAxis for this group|

***Note**: Some parameters are present in both chart options and chart data options. For boolean values, the property will evaluate to true if either value is true. For other types of values, the chart data option value will take precendence over the chart option value.* 

### Brush Context Menu Actions

To take action on a line chart brush action (like in the [Explore events example]('https://tsiclientsample.azurewebsites.net/examples/withplatform/exploreevents.html')), brushContextMenuActions are added in chartOptions, with the following shape...

```js
var brushContextMenuActions = [
    {
        name: "Log From and To Times",
        action: function(fromTime, toTime){
            console.log(fromTime, toTime);
        }
    }, 
    {
        name: "Explore Events",
        action: function(fromTime, toTime){
            // an implementation is in the explore events example
        }
    }
]

// later, when you render a line chart, use it like this
lineChart.render(data, {brushContextMenuActions: brushContextMenuActions});
```

### Group Context Menu Actions

To take action on a context menu click of a data group (like in the [Line chart generating bar and pie charts example]('https://tsiclientsample.azurewebsites.net/examples/withplatform/exploreevents.html')), groupContextMenuActions are added to chart data options, with the following shape...

```js
var groupContextMenuActions = [{
        name: "Print parameters to console",
        action: function(dataGroupName, timeSeriesName, timestamp) {
            console.log(ae);
            console.log(splitBy);
            console.log(timestamp);
        }
    }, 
    {
        name: "Some other function",
        action: function(dataGroupName, timeSeriesName, timestamp) {
            // left as an exercise
        }
    }
    ];
```

### Search Span Object

Search span objects define the range and bucket size of the time series of a data group.  Search span objects are useful for showing sparse data in a line chart, like [this example]('https://tsiclientsample.azurewebsites.net/examples/withplatform/basicCharts.html')They have the following shape...

```js
var searchSpanObject = {
    from: '2017-04-20T12:00:00Z', // a js date isostring
    to: '2017-05-20T12:00:00Z',
    bucketSize: '1h'  // an integer, followed by one of ms, s, m, h, d
}
```

### Line Chart Events and States Data Shape

Events and states are used to show diamonds and blocks of color for specifying categorical data overlayed with a line chart, like in the example [here](https://tsiclientsample.azurewebsites.net/examples/noauth/multipleseriestypes.html).  Good examples of such events are "an important incident occured" for a diamond, or "a compressor was on" for a block of color.  They are specified as an array of JSON objects--each element of the array will get its own vertical space on the chart.  The general shape for both events and states is as follows...

```js
var statesOrEvents = [
    {"Component States" : // This key is shown in the legend
        [
            {
                [from.toISOString()] : {  // this timestamp is where the event or state is drawn
                    'color': 'lightblue',  // the color of the diamond or block
                    'description' : 'Cooling fan on' // the text on hover
                }
            },
            {
                [(new Date(from.valueOf() + 1000*60*12)).toISOString()] : {
                    'color': '#C8E139',
                    'description' : 'Filling tank at maximum pressure'
                }
            },
            {
                [(new Date(from.valueOf() + 1000*60*32)).toISOString()] : {
                    'color': '#D869CB',
                    'description' : 'Pressing machine overheated'
                }
            }
        ]
    }
];

// later, use states or events like this...
lineChart.render(data, {states: statesOrEvents, events: statesOrEvents}, chartDataOptionsArray);
```

## Appendix

### Additional Chart Options

Some less common chart options that can be used for very specific user interactions are...

|Property Name|Type|Value Options|Description|
|-|-|-|-|
|aggTopMargin|number|**12**|Margin on top of each aggregate line(s) in pixels|
|arcWidthRatio|number| **0**|Ratio of the outer and inner circle in a pie chart, from 0 to 1|
|autoTriggerBrushContextMenu|boolean|**false**, true|If true, opens a menu defined by brushContextMenuActions on brush mouseup|
|availabilityLeftMargin|number|**60**|Left margin of the availability chart in pixels|
|brushClearable|boolean|**true**|If true, maintain brush selected region upon clear and non-selection of a new region|
|brushHandlesVisible|boolean|**false**|If true, draw handles on the line chart brush|
|brushMoveAction|(from:DateTime, to:DateTime) => any |**() => {}**|Action fired when the brush moves|
|brushMoveEndAction|(from:DateTime, to:DateTime) => any|**() => {}**|Action fired at the end of a mouse movement involving the brush|
|canDownload|boolean|**true**|If true, chart's ellipsis menu contains a download button to download the chart's data|
|color|string|**null**, 'purple', '#404040'|Color of the time selection ghost in availability chart|
|focusHidden|boolean|**false**|If true, hide focus element|
|fromChart|boolean|**false**|If true, a component is a subcomponent of another component|
|hideChartControlPanel|boolean|**false**|If true, hide panel with chart control buttons|
|includeTimezones|boolean|**true**|If true, include timezone dropdown in dateTimePicker|
|isArea|boolean|**false**|If true, lines in LineChart are also areas|
|isCompact|boolean|**false**|If true, availability chart is in compact mode (expanded mode if false)|
|is24HourTime|boolean|**true**|If true, display time in 24 hour format, (12 hour time with am/pm if false|
|keepBrush|boolean|**false**|If true, maintain brush selected region upon render|
|keepSplitByColor|boolean|**false**|If true, maintain split by colors when state updated|
|maxBuckets|number|**500**|Max number of buckets in availability chart|
|minBrushWidth|number|**0**|Minimum possible width of brush in a linechart in pixels|
|minutesForTimeLabels|boolean|**false**|If true, force time labels to minute granularity|
|onInstanceClick|(instance: any) => any|**() => {return {}}**|For model search: takes an instance and returns an object of context menu actions|
|onMouseout|() => void|**() => {}**|Action fired when the mouse leaves a chart value element (ex: line, bar, pie chart segment, etc.)|
|onMouseover|(aggKey: string, splitBy: string) => void|**() => {}**|Action fired when the mouse enters a chart value element|
|onSticky|(aggKey: string, splitBy: string) => void|**() => {}**|Action fired when a chart value element is stickied|
|onUnsticky|(aggKey: string, splitBy: string) => void|**() => {}**|Action fired when a chart value element is stickied|
|onKeydown|(d3Event: any, awesompleteObject: any) => void  |**() => {}**|Action fired when keydown action performed in ModelAutocomplete|
|onInput|(searchText: string) => void |**() => {}**|Action fired on input actions in ModelAutocomplete|
|preserveAvailabilityState|boolean|**false**|If true, save availability chart state on render|
|scaledToCurrentTime|boolean|**false**|If true,  base slider base component's scale on current time's values (all values if false)|
|singleLineXAxisLabel|boolean|**false**|If true, display x axis time labels on a single line (split into two lines if false)|
|snapBrush|boolean|**false**|If true, snap linechart brush to closest value|
|suppressResizeListener|boolean|**false**|If true, ignore components' resize function. This applies to components which draw an SVG|
|timeFrame|any|**null**|From and to to specify range of an event or state series
|withContextMenu|boolean|**false**|If true, the hierarchy uses a context menu when you click on a parent of leaf nodes
|xAxisHidden|boolean|**false**|If true, hide xAxis in chart
|yAxisHidden|boolean|**false**|If true, hide yAxis in chart
|zeroYAxis|boolean|**true**|If true, set bar chart's bar's bottom (or top if negative) to zero