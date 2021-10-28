# [plotly_project](https://li-emily.github.io/plotly_project)

## Overview of Project

### Purpose
> Roza has a partially completed dashboard that she needs to finish. She has a completed panel for demographic information and now needs to visualize the bacterial data for each volunteer. Specifically, her volunteers should be able to identify the top 10 bacterial species in their belly buttons. That way, if Improbable Beef identifies a species as a candidate to manufacture synthetic beef, Roza's volunteers will be able to identify whether that species is found in their navel.

### Goals
* Data visualization using JavaScript.
* Manipulate data by using and creating JavaScript functions and methods. 
* Finding and processing data using D3.json.
* Learning to build charts and graphs with Plotly.
* Becoming more familiar HTML/CSS editing.
* Deployment of result onto GitHub Pages.


### Resources
* Languages: Javascript, HTML, CSS
* Packages: D3.json, Plotly
* Interface: Visual Studio Code, Terminal, Conda Environment, Google Chrome Javascript Console
* Data: [samples.json](https://github.com/li-emily/plotly_project/blob/main/samples.json)

## Results
The new and updated dashboard has available data for all individuals that were tested. They can be easily picked by Test Subject ID Number, and their corresponding demographic information is displayed below. 

The bubble chart is displayed first, with a side blurb describing the purpose of each chart shown. 

![dashboard1](https://github.com/li-emily/plotly_project/blob/main/images/dashboard1.png)

Below is the latter part of the webpage. The bar graph and gauge chart are shown.

![dashboard2](https://github.com/li-emily/plotly_project/blob/main/images/dashboard2.png)

## JavaScript Code
### Default page
This initial init() function was given by Roza. It first references the dropdown select element, grabs the first ID (940) and sends the information over for the demographics function to build the information panel, then over to the charts function to build all three charts.

```
function init() {
  // Grab a reference to the dropdown select element
  var selector = d3.select("#selDataset");

  // Use the list of sample names to populate the select options
  d3.json("samples.json").then((data) => {
    var sampleNames = data.names;
  // Use forEach to grab all the individual data
    sampleNames.forEach((sample) => {
      selector
        .append("option")
        .text(sample)
        .property("value", sample);
    });

    // Use the first sample from the list to build the initial plots
    var firstSample = sampleNames[0];
    buildCharts(firstSample);
    buildMetadata(firstSample);
  });
};

// Initialize the dashboard
init();
```

### Changing Information
Roza also created a function to update the data once a new ID number is picked from the dropdown menu. It sends the chosen sample over to the demographics and chart building functions.

```
function optionChanged(newSample) {
  // Fetch new data each time a new sample is selected
  buildMetadata(newSample);
  buildCharts(newSample);
};
```

### Demographics

For the buildMetadata() function, the data was loaded in using d3, similar to the init() function. The snippet below shows JavaScript filtering and arrow function to grab the object array for the chosen sample ID. D3 is then used to select the div id from the HTML file, and then first clearing the panel to remove any previous data. Then by using Object.entries() and forEach(), a new JS arrow function is able to print all the resulting demographic information in key-value pairs.

```
// Filter the data for the object with the desired sample number
var resultArray = metadata.filter(sampleObj => sampleObj.id == sample);
var result = resultArray[0];
  
// Use d3 to select the panel with id of `#sample-metadata`
var PANEL = d3.select("#sample-metadata");

// Use `.html("") to clear any existing metadata
PANEL.html("");

// Use `Object.entries` to add each key and value pair to the panel
// Hint: Inside the loop, you will need to use d3 to append new
// tags for each key-value in the metadata.
Object.entries(result).forEach(([key, value]) => {
  PANEL.append("h6").text(`${key.toUpperCase()}: ${value}`);
});
```
### Chart Building
The buildCharts() function contains all three charts (bubble, bar, and gauge) that are produced for the web dashboard. Below shows how data sent by the init() and optionChanged() functions is then manipulated through to find the basic data variables that each chart will then be built on.

```
function buildCharts(sample) {
  // 2. Use d3.json to load and retrieve the samples.json file 
  d3.json("samples.json").then((data) => {
    // 3. Create a variable that holds the samples array. 
    var sampleData = data.samples;
    // 4. Create a variable that filters the samples for the object with the desired sample number.
    var sampleArray = sampleData.filter(obj => obj.id == sample);

    //  5. Create a variable that holds the first sample in the array.
    var first = sampleArray[0];

    // 6. Create variables that hold the otu_ids, otu_labels, and sample_values.
    var otuID = first.otu_ids;
    var otuLabels = first.otu_labels;
    var sampleValues = first.sample_values;
```

### Bubble Chart
Luckily for the bubble chart, the above variables can then be directly plugged into the trace function. The bubble chart layout is created, and then the information is plugged into Plotly to create the final chart, referencing the bubble div id from the HTML file.

```
// 1. Create the trace for the bubble chart.
var bubbleData = [{
  x: otuID,
  y: sampleValues,
  text: otuLabels,
  mode: 'markers',
  marker: {
    size: sampleValues,
    color: sampleValues,
    colorscale: 'YlGnBu'
  }
}];
...
// 3. Use Plotly to plot the data with the layout.
Plotly.newPlot("bubble", bubbleData, bubbleLayout); 
```

### Bar Chart
For the bar chart, we wanted a specific horizontal chart with 10 data points, sorted in descending order. This was achieved by using map(), slice(), and reverse() JavaScript functions. The map() function on yticks allowed us to add the OTU to each value. Slice() was used to grab the first 10 values, and then reverse() to change the arrays into descending order.

```
// 7. Create the yticks for the bar chart.
// Hint: Get the the top 10 otu_ids and map them in descending order  
//  so the otu_ids with the most bacteria are last. 

var yticks = otuID.map((id) => 'OTU' + id).slice(0,10).reverse();
var barLabels = otuLabels.slice(0,10).reverse();
var xvalues = sampleValues.slice(0,10).reverse();

// 8. Create the trace for the bar chart. 
var barData = [{
  x: xvalues,
  y: yticks,
  text: barLabels,
  type: 'bar',
  orientation: 'h',
  marker: {
    color: '#41b6c4'
  }
}];
```
### Gauge Chart
For the gauge chart, we had to grab a different array than the other two charts needed. The information was under the metadata array instead, and the data was found similar to the above. Some aspects of the chart had to be color coded for visual impact.

```
// 1. Create a variable that filters the metadata array for the object with the desired sample number.
var metadata = data.metadata;
var gaugeArray = metadata.filter(obj => obj.id == sample);

// 2. Create a variable that holds the first sample in the metadata array.
var firstGauge = gaugeArray[0];

// 3. Create a variable that holds the washing frequency.
var washFreq = firstGauge.wfreq;
    
// 4. Create the trace for the gauge chart.
var gaugeData = [{
  value: washFreq,
  type: "indicator",
  mode: "gauge+number",
  title: {text: "<b>Belly Button Washing Frequency</b> <br> Scrubs per week"},
  gauge: {
    axis: {range: [null,10], dtick: "2"},
    bar: {color: "gray"},
    steps:[
      {range: [0,2], color: "#edf8b1"},
      {range: [2,4], color: "#c7e9b4"},
      {range: [4,6], color: "#7fcdbb"},
      {range: [6,8], color: "#1d91c0"},
      {range: [8,10], color: "#253494"}
    ]
  }
}];
```

## HTML Webpage
For the webpage, the majority was already built by Roza. A few elements were edited and added on for a more effective and pleasing viewing experience.

```
<div class="col-md-12 jumbotron text-center" 
	  style= "background-color:#e7e7e7">
<h1 style="background-color: #afc5c0; color: white; 
	 font-family:'Courier New', Courier, monospace">
	 Belly Button Biodiversity Dashboard</h1>
```

The order of the charts was also flipped, with the bubble chart moved above the bar and gauge charts. An added side panel with information about each chart was put next to the bubble chart as well. The bubble chart was slightly resized so that this information panel could be added on the same row.

```
<div class="col-md-2">
  <div class="panel panel-default">
    
    <div class="panel-heading" style="background-color:#afc5c0">
      <h5 style="background-color:#afc5c0; text-align: center;">Bubble Chart</h5>
    </div>
    <div class="panel-body">
      <p>Bubble size measures the amount of each bacteria culture in the bellybutton.</p>
    </div>
    <div class="panel-heading" style="background-color:#afc5c0">
      <h5 style="background-color:#afc5c0; text-align: center;">Bar Graph</h5>
    </div>
    <div class="panel-body">
      <p>Graphs the top 10 bacteria cultures that were found in the bellybutton.</p>
    </div>
    <div class="panel-heading" style="background-color:#afc5c0">
      <h5 style="background-color:#afc5c0; text-align: center;">Gauge Chart</h5>
    </div>
    <div class="panel-body">
      <p>Shows weekly washes of each person's bellybutton.</p>
    </div>
  </div>
```
