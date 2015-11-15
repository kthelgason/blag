---
layout: post
title:  "Homophily and Schelling models"
date:   2015-04-26 22:49:30
tags: category-theory sociology
---
Homophily is defined on [wikipedia][1] as the tendency of individuals to associate and bond with others similar to themselves. Homophily is an effect that each of us are surely familiar with from our lives, for example, most of my friends are software developers, and interested in the same things I am. 

This phenomenon shows up in many network studies in some form, often factors such as race, gender, ethnicity or other *immutable* properties contribute to homophily, but homophily also shows up in other areas such as hobbies, books read and websites frequented. 

<!-- more -->

The classic oft-cited example of homophily is racial segregation in american neighborhoods.

![Detroit](/assets/detroitlabel.jpg)

This image shows a map of Detroit, where blue dots represent caucasian families and green dots African American. In this example 8 Mile Road is a clear divider, and (hopefully) most cities are not this bad, but the phenomenon can be seen almost anywhere one cares to look.

As a different example, in [this study][6] published in the New England Journal of Medicine, Christakis et al. show that the probability of a person becoming obese increases with each obese friend in his or her social network. This is an example of behavioral homophily.

An interesting question to ask is whether it is the similarity that causes people to connect or vice versa, that people become more and more like their friends and family.  In the [classic 2001 paper][2] on the subject, McPherson et al. asserted that it is in fact similarity that brings people together. This opinion is echoed in a [more recent paper][3] by Crandall et al. where they analysed data from wikipedia on edits. In that study the authors define similarity between two authors to be a function of how many articles they have both edited and find that the similarity increases very rapidly just before they first make contact, but also just after. This seems to suggest that similarity leads people together, but that the connection also makes people more similar.


Schelling's model of segregation
----------------------

The American Economist [Thomas Schelling][4] created a simple model that he thought might help explain why segregation is so widespread. His model consists of several agents living on a grid. The agents are either type A or type B. Now each of the agents prefers to live among his peers according to parameter $$t$$. In each *round* of the simulation, an agent decides wether he would like to move or not. Specifically, for each agent $$a$$, if the proportion of $$a$$'s neighbors of opposite type is less than $$t$$ he stays put, otherwise he moves to a random location. 

Go ahead and play the simulation below, and try it for different values of $$t$$. I found it fascinating that even for low values such as $$t=0.4$$ clear clusters form very quickly.


<div class="container">
   <form>
   <button class="btn" type="button" onclick="start(); return false;">start</button>
   <input type="range" id="thresh" max="1" min="0.1" step="0.1" oninput="updateThresh(this.value)">
   <label id="tlabel"></label>
   </form>
<canvas id="grid" width="501px" height="501px" style="display: block; margin-left: auto; margin-right: auto"></canvas>
 </div>

<script>

var grid = [];
var size = 50;
var t = 0.5;
var initialN = 1150;
var c_grid = document.getElementById('grid');
var context = c_grid.getContext('2d');

function runStep (grid) {
  for (var i = 0; i < size*size; i += 1) {
    if (grid[i] && !is_satisfied(grid, i)) {
      var newIdx = getRandomEmptyLocation(grid);
      grid[newIdx] = grid[i];
      grid[i] = 0;
    }
  }
  drawGrid(grid);
}

function neighborhood (idx) {
  idx = [idx % size, Math.floor(idx / size)];
  var neighborinos = [];
  for (var i = -1; i < 2; i++) {
    for (var j = -1; j < 2; j++) {
      var neighbor = [idx[0] + i, idx[1] + j];
      if (neighbor[0] >= 0 && neighbor[0] < size && neighbor[1] >= 0 && neighbor[1] < size) {
        neighborinos.push(neighbor[1]*size + neighbor[0]);
      }
    }
  }
  return neighborinos;
}

function is_satisfied (grid, idx) {
  var n = neighborhood(idx);
  var similar = 0;
  for (var i = 0; i < n.length; ++i) {
    if (grid[n[i]] === grid[idx] || grid[n[i]] === 0) {
      similar++;
    }
  }
  return (similar/n.length) >= t;
}

function getRandomEmptyLocation (grid) {
  while (true) {
    var x = Math.floor(Math.random() * size);
    var y = Math.floor(Math.random() * size);
    var idx = y * size + x;
    if (!grid[idx]){
      return idx;
    }
  }
}

function drawGrid (grid) {
  var offset = 0.5;
  var scale = 10;
  for (var x = 0; x < size; x += 1) {
    for (var y = 0; y < size; y += 1) {
      var color;
      switch (grid[y*size + x]) {
        case 0:
          color = "#efecca"; break;
        case 1:
          color = "#a7a37e"; break;
        case 2:
          color = "#002f2f"; break;
      }
      context.fillStyle = color;
      context.fillRect(x*scale+offset,y*scale+offset,scale,scale);
    }
  }
}

function initGrid (grid, size, typeA, typeB) {
  for (var i = 0; i < (size * size); ++i) {
    grid[i] = 0;
  }
  for (var i = 0; i < typeA; ++i) {
    var idx = getRandomEmptyLocation(grid);
    grid[idx] = 1;
  }
  for (var i = 0; i < typeB; ++i) {
    var idx = getRandomEmptyLocation(grid);
    grid[idx] = 2;
  }
}

function updateThresh (val) {
  t = val;
  document.getElementById('tlabel').innerHTML = "Threshold: " + t;
}

function start() {
  initGrid(grid, size, initialN, initialN);
  drawGrid(grid);

  window.setInterval(function() {
    runStep(grid);
  }, 500);
}

initGrid(grid, size, initialN, initialN);
drawGrid(grid);
document.getElementById('tlabel').innerHTML = "Threshold: " + t;
document.getElementById('thresh').value = t;
</script>







[1]:      http://en.wikipedia.org/wiki/Homophily
[2]:		http://arjournals.annualreviews.org/doi/abs/10.1146/annurev.soc.27.1.415
[3]:		http://www.cs.cornell.edu/~dph/papers/kdd08-sim.pdf
[4]: 		http://en.wikipedia.org/wiki/Thomas_Schelling
[5]: 		http://en.wikipedia.org/wiki/Groupthink
[6]:		http://media.timesfreepress.com/docs/2010/01/Obesity_studies_0104.pdf
