---
title: "Playing with D3.js"
excerpt: "I haven't done much on my composer project until recently when I came across the javascript library, D3.js. D3 stands for Data-Driven Documents. It's all about efficient binding between data and DOM elements. With just a few hours of work, I was able to create something that takes some JSON and generates a beautiful interactive graph..."
modified: 2012-02-24
header:
  teaser: 
tags: 
  - svg
  - D3
  - COSA
  - Composer
---


I haven't done much on my [composer]({{ base_path }}/post-composer/) project until recently when I came across the javascript library, <a href="http://d3js.org/">D3.js</a>. D3 stands for Data-Driven Documents. It's all about efficient binding between data and DOM elements. With just a few hours of work, I was able to create something that takes some JSON and generates a beautiful interactive graph...

<!--more-->
<div id="graph" style="background-color:#fff">
</div>

I'll leave out all the gory details about how it was generated. The important part is that it was generated from the following JSON...

```js
{
  "component":{
    "id":"myComponent",
    "vertices":[
      {"id":"Cell1", "vertexType":"cell"},
      {"id":"Cell2", "vertexType":"cell"},
      {"id":"Cell3", "vertexType":"cell"},
      {"id":"Cell4", "vertexType":"cell"},
      {"id":"Cell5", "vertexType":"cell"},
      {"id":"Cell6", "vertexType":"cell"},
      {"id":"Connector1", "vertexType":"connector", ioCount:5}
    ],
    "edges":[
      {"source": "Cell1", "target": "Cell2", "type": "connection", "targetConnection": "InputMaster"},
      {"source": "Cell1", "target": "Cell2", "type": "association"},
      {"source": "Cell1", "target": "Cell3", "type": "connection", "targetConnection": "InputInverted"},
      {"source": "Cell1", "target": "Cell4", "type": "connection", "targetConnection": "InputSlave"},
      {"source": "Cell3", "target": "Cell4", "type": "connection", "targetConnection": "InputSlaveInverted"},
      {"source": "Cell4", "target": "Cell5", "type": "connection", "targetConnection": "Input"},
      {"source": "Cell4", "target": "Connector1", "type": "connection", "targetConnection": "Input", "targetIndex": 0},
      {"source": "Cell1", "target": "Connector1", "type": "connection", "targetConnection": "Input", "targetIndex": 1},
      {"source": "Connector1", "target": "Cell6", "type": "connection", "targetConnection": "Input", "sourceIndex": 4}
    ],
    "data":[
    ]
  }
}
```

<p>
The difficult part was making up for the sleep I enjoyed many moons ago in my high school trigonometry class. I had to figure out how to offset those little circles from the centers of the larger circles according to the angle of the line. The math is only simple once you figure out what equations to plug in.
</p>
<p>
There will still be a lot of work before it becomes useful. I need to be able to add and remove various types of nodes with different representations of them and add/remove connections between them. A component will usually have one or more variables and/or small memory heaps that it is in charge of, so there will need to be a way to manage that. And all aspects of a given component need to be transformed to/from JSON.
</p>




<p>
The difficult part was making up for the sleep I enjoyed many moons ago in my high school trigonometry class. I had to figure out how to offset those little circles from the centers of the larger circles according to the angle of the line. The math is only simple once you figure out what equations to plug in.
</p>
<p>
There will still be a lot of work before it becomes useful. I need to be able to add and remove various types of nodes with different representations of them and add/remove connections between them. A component will usually have one or more variables and/or small memory heaps that it is in charge of, so there will need to be a way to manage that. And all aspects of a given component need to be transformed to/from JSON.
</p>
<style type="text/css">

line.link {
  stroke: #999;
  stroke-opacity: .6;
}

.cursor {
  fill: none;
  stroke: brown;
  pointer-events: none;
}

.link {
  stroke: #999;
}



path.link {
  fill: none;
  stroke: #666;
  stroke-width: 1.5px;
  stroke-dasharray: 9, 5;
  stroke-opacity: .6;
}

marker#licensing {
  fill: green;
}

path.link.licensing {
  stroke: green;
}

path.link.resolved {
  stroke-dasharray: 0,2 1;
}

.connector,
.cell {
  fill: #ccc;
  stroke: #333;
  stroke-width: 1px;
}

text {
  font: 10px sans-serif;
  pointer-events: none;
}

text.shadow {
  stroke: #fff;
  stroke-width: 3px;
  stroke-opacity: .8;
}
</style><script src="http://d3js.org/d3.v2.js"></script><script type="text/javascript">
var json = {
  "component":{
    "id":"myComponent",
    "vertices":[
      {"id":"Cell1", "vertexType":"cell"},
      {"id":"Cell2", "vertexType":"cell"},
      {"id":"Cell3", "vertexType":"cell"},
      {"id":"Cell4", "vertexType":"cell"},
      {"id":"Cell5", "vertexType":"cell"},
      {"id":"Cell6", "vertexType":"cell"},
      {"id":"Connector1", "vertexType":"connector", ioCount:5}
    ],
    "edges":[
      {"source": "Cell1", "target": "Cell2", "type": "connection", "targetConnection": "InputMaster"},
      {"source": "Cell1", "target": "Cell2", "type": "association"},
      {"source": "Cell1", "target": "Cell3", "type": "connection", "targetConnection": "InputInverted"},
      {"source": "Cell1", "target": "Cell4", "type": "connection", "targetConnection": "InputSlave"},
      {"source": "Cell3", "target": "Cell4", "type": "connection", "targetConnection": "InputSlaveInverted"},
      {"source": "Cell4", "target": "Cell5", "type": "connection", "targetConnection": "Input"},
      {"source": "Cell4", "target": "Connector1", "type": "connection", "targetConnection": "Input", "targetIndex": 0},
      {"source": "Cell1", "target": "Connector1", "type": "connection", "targetConnection": "Input", "targetIndex": 1},
      {"source": "Connector1", "target": "Cell6", "type": "connection", "targetConnection": "Input", "sourceIndex": 4}
    ],
    "data":[
    ]
  }
};


var connectionTypes = [
  {name: "Input", color: "#fff"},
  {name: "InputInverted", color: "#000"},
  {name: "InputMaster", color: "#40D2FF"},
  {name: "InputSlave", color: "#fff"},
  {name: "InputSlaveInverted", color: "#000"},
  {name: "Output", color: "#f00"}
]


replaceObjectIdsWithReferences(json);

var w = 600,
    h = 400,
    cellRadius =20,
    ioNodeRadius = 3.5,
    associationRadius = 40;

var force = d3.layout.force()
    .nodes(d3.values(json.component.vertices))
    .links(json.component.edges)
    .size([w, h])
    .linkDistance(100)
    .charge(-500)
    .on("tick", tick)
    .on("end", endLayout)
    .start();




var svg = d3.select("#graph").append("svg:svg")
    .attr("width", w)
    .attr("height", h);


var defs = svg.append("defs")

var connectionMarkerEntries = defs.selectAll("marker")
    .data(connectionTypes) //["connection"])
  .enter()
  
connectionMarkerEntries.append("marker")
    .attr("id", function(d) { return "cellConnection" + d.name })
    .attr("refX", function(d) { 
      if (d.name == "Output")
        return -(cellRadius - 1);
      return cellRadius + ioNodeRadius * 2 + 1;
    })
    .attr("refY", ioNodeRadius + 1)
    .attr("markerWidth", ioNodeRadius * 2 + 2)
    .attr("markerHeight", ioNodeRadius * 2 + 2)
    .attr("orient", "auto")
    .append("circle")
      .attr("r", ioNodeRadius)
      .attr("fill", function(d) { return d.color })
      .attr("stroke", "#000")
      .attr("transform", "translate(" + (ioNodeRadius + 1) + "," + (ioNodeRadius + 1) + ")");

connectionMarkerEntries.append("marker")
    .attr("id", function(d) { return "connectorConnection" + d.name })
    .attr("refX", ioNodeRadius + 1)
    .attr("refY", ioNodeRadius + 1)
    .attr("markerWidth", ioNodeRadius * 2 + 2)
    .attr("markerHeight", ioNodeRadius * 2 + 2)
    .attr("orient", "auto")
    .append("circle")
      .attr("r", ioNodeRadius)
      .attr("fill", function(d) { return d.color })
      .attr("stroke", "#000")
      .attr("transform", "translate(" + (ioNodeRadius + 1) + "," + (ioNodeRadius + 1) + ")");


defs.append("clipPath")
  .attr("id", "clipWorkspace")
  //.attr("maskContentUnits", "objectBoundingBox")
  //.attr("clipPathUnits","objectBoundingBox")
  .append("rect")
    .attr("x", 20)
    .attr("y", 20)
    .attr("rx", 20)
    .attr("ry", 20)
    .attr("width", w - 40)
    .attr("height", h - 40)
    .attr("stroke","gray")
    .attr("fill","none")
    ;//.attr("fill", "#000");   


var associations = svg.append("svg:g")
  .attr("clip-path", "url(#clipWorkspace)")
  .selectAll(".association")
    .data(force.links().filter(function(val) { return val.type == "association"; }))
  .enter().append("svg:path")
    .attr("class", "link association");

var connections = svg.append("svg:g")
  .attr("clip-path", "url(#clipWorkspace)")
  .selectAll(".connection")
    .data(force.links().filter(function(val) { return val.type == "connection"; }))
  .enter().append("svg:line")
    .attr("class", "link connection")
    .attr("marker-end", function(d) {
      if (d.target.vertexType == "cell") 
        return "url(#cellConnection" + d.targetConnection + ")";
      else
        return "url(#connectorConnection" + d.targetConnection + ")";
    })
    .attr("marker-start", function(d) { 
      if (d.source.vertexType == "cell") 
        return "url(#cellConnectionOutput)";
      else
        return "url(#connectorConnectionOutput)";
    });

var cellNodes = svg.append("svg:g")
  .attr("clip-path", "url(#clipWorkspace)")
  .attr("mask", "url(#maskWorkspace)")
  .selectAll("g.cell")
    .data(force.nodes().filter(function(d) { return d.vertexType == "cell"; }))
  .enter().append("svg:circle")
  
    .attr("class", "cell")
    .attr("r", cellRadius)
    .call(force.drag)
    //.call(node_drag)
    ;

var connectorNodes = svg.append("svg:g")
  .attr("clip-path", "url(#clipWorkspace)")
  .attr("mask", "url(#maskWorkspace)")
  .selectAll("g.connector")
    .data(force.nodes().filter(function(d) { return d.vertexType == "connector"; }))
  .enter().append("svg:rect")
  
    .attr("class", "connector")
    .attr("width", ioNodeRadius * 2)
    .attr("height", function(d){ return d.ioCount * ioNodeRadius * 2 })
    .call(force.drag)
    //.call(node_drag)
    ;


var texts = svg.append("svg:g")
  .attr("clip-path", "url(#clipWorkspace)")
  .selectAll("g")
    .data(force.nodes().filter(function(d) { return d.vertexType == "cell"; }))
  .enter().append("svg:g");
  
  
// A copy of the text with a thick white stroke for legibility.
texts.append("svg:text")
    .attr("x", 8)
    .attr("y", ".31em")
    .attr("class", "shadow")
    .text(function(d) { return d.id; });

texts.append("svg:text")
    .attr("x", 8)
    .attr("y", ".31em")
    .text(function(d) { return d.id; });
    
var alphaText = svg.append("svg:g")
  .attr("clip-path", "url(#clipWorkspace)")
  .append("svg:text")
  .attr("x", 30)
  .attr("y", 30)
  .text(force.alpha());

var border = svg.append("svg:g").append("rect")
  .attr("x", "20")
  .attr("y", "20")
  .attr("rx", "20")
  .attr("ry", "20")
  .attr("width", w - 40)
  .attr("height", h - 40)
  .attr("fill-opacity", "0")
  .attr("stroke","#000")
  .attr("stroke-width","1px")
  .attr("pointer-events", "none");


function replaceObjectIdsWithReferences(json) {
  var nodeMap = {};
  
  json.component.vertices.forEach(function(vertex){
    nodeMap[vertex.id] = vertex;
  });
  
  json.component.edges.forEach(function(edge) {  
    edge.source = nodeMap[edge.source];
    edge.target = nodeMap[edge.target];
  });
};

function replaceObjectReferencesWithIds(json) {
  json.component.edges.forEach(function(edge) {  
    edge.source = edge.source.id;
    edge.target = edge.target.id;
  });
};


function getRadiusPoint(fromPoint, toPoint, radius){
  var vect = { x : toPoint.x - fromPoint.x, y: toPoint.y - fromPoint.y };
  var length = Math.sqrt(vect.x * vect.x + vect.y * vect.y);
  var scale = radius / length;
  vect.x = vect.x * scale;
  vect.y = vect.y * scale;
  return { x: fromPoint.x + vect.x, y: fromPoint.y + vect.y };
}


function tick() {
  connections
      .attr("x1", function(d) { 
        if (d.source.vertexType == "cell")
          return d.source.x; 
        return d.source.x + ioNodeRadius * 3;
      })
      .attr("y1", function(d) { 
        if (d.source.vertexType == "cell")
          return d.source.y; 
        return d.source.y + ioNodeRadius + ioNodeRadius * 2 * d.sourceIndex;
      })
      .attr("x2", function(d) { 
        if (d.target.vertexType == "cell")
          return d.target.x; 
        return d.target.x + ioNodeRadius * 3;
      })
      .attr("y2", function(d) { 
        if (d.target.vertexType == "cell")
          return d.target.y; 
        return d.target.y + ioNodeRadius + ioNodeRadius * 2 * d.targetIndex;
      });

  associations.attr("d", function(d) {
    var dx = d.target.x - d.source.x,
        dy = d.target.y - d.source.y,
        dr = Math.sqrt(dx * dx + dy * dy);
        
    var angle = Math.atan2(d.target.y - d.source.y, d.target.x - d.source.x),
        angleDiff = 90 * Math.PI / 180,
        x1 = d.source.x + associationRadius * Math.cos(angle + angleDiff),
        y1 = d.source.y + associationRadius * Math.sin(angle + angleDiff),
        x2 = d.target.x + associationRadius * Math.cos(angle - Math.PI - angleDiff),
        y2 = d.target.y + associationRadius * Math.sin(angle - Math.PI - angleDiff);
    
    
    // Use elliptical arc path segments to represent association...
    return "M" + x1 + "," + y1 + "A" + dr + "," + dr + " 0 0,0 " + x2 + "," + y2;
  });

  cellNodes.attr("transform", function(d) {
    return "translate(" + d.x + "," + d.y + ")";
  });
  
  connectorNodes.attr("transform", function(d) {
    return "translate(" + d.x + "," + d.y + ")";
  });

  texts.attr("transform", function(d) {
    return "translate(" + d.x + "," + d.y + ")";
  });
  
  alphaText.text(force.alpha());
}

function endLayout()
{
  // once layout has settled, mark all nodes as fixed so that further manual changes don't
  // result in more automatic layout
  force.nodes().forEach(function(d) {d.fixed = true;});
}


var node_drag = d3.behavior.drag()
    .on("dragstart", dragStart)
    .on("drag", dragMove)
    .on("dragend", dragEnd);

function dragStart(d, i) {
    force.stop() // stops the force auto positioning before you start dragging
}

function dragMove(d, i) {
    d.px += d3.event.dx;
    d.py += d3.event.dy;
    d.x += d3.event.dx;
    d.y += d3.event.dy; 
    tick(); // this is the key to make it work together with updating both px,py,x,y on d !
}

function dragEnd(d, i) {
    d.fixed = true; // of course set the node to fixed so the force doesn't include the node in its auto positioning stuff
    tick();
    force.resume();
}



</script>
