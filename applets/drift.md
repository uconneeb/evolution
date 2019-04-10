---
layout: applet
title: Genetic Drift
permalink: /applets/drift/
---

## Genetic Drift in small populations

AA homozygotes are yellow, Aa heterozygotes are orange, and aa homozygotes are red. Starting frequency of A is 0.5 in each population.

Press the "n" key to advance to the next generation. Repeat to see eventual fixation of either A or a in each population. Refresh your browser to start over.

<div id="canvas"></div>
<script type="text/javascript">
    // The MIT License (MIT)
    // 
    // Copyright (c) 2019 Paul O. Lewis
    // 
    // Permission is hereby granted, free of charge, to any person obtaining a copy
    // of this software and associated documentation files (the “Software”), to deal
    // in the Software without restriction, including without limitation the rights
    // to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    // copies of the Software, and to permit persons to whom the Software is
    // furnished to do so, subject to the following conditions:
    // 
    // The above copyright notice and this permission notice shall be included in all
    // copies or substantial portions of the Software.
    // 
    // THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    // IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    // FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    // AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    // LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    // OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    // SOFTWARE.
    // 
    // written by Paul O. Lewis 10-Apr-2019
    
    // width and height of svg
    var w = 600;
    var h = 600;
    
    // There poprows x popcols isolated subpopulations
    var poprows = 4;
    var popcols = 4;
    
    // There indivrows x indivcols diploid individuals per subpopulation
    var indivrows = 3;
    var indivcols = 3;
    
    // Dimensions of cells in which individuals are shown
    var wcell = w/(popcols*indivcols);
    var hcell = h/(poprows*indivrows);
    var cell_avg_diam = (wcell + hcell)/2;
    
    // Radius of circle representing a single individual
    var rindiv = 0.3*cell_avg_diam;

    // Determines amount of Gaussian jigger to impart to each individual's position
    var jigger_stdev = 0.3*cell_avg_diam;
    
    // Genotype colors
    var genotype_color = ["yellow", "orange", "red"];
    
    // Initialize frequency of A allele in all subpopulations
    var freqA = [];
    for (let i = 0; i < poprows; i++) {
        for (let j = 0; j < popcols; j++) {
            freqA.push({"i":i, "j":j, "freq":0.5});
        }
    }
    
    // Returns index into data vector of individual on row indivrow, column indivcol,
    // in the subpopulation at row poprow and column popcol. 
    function getDataIndex(poprow, popcol, indivrow, indivcol) {
        return poprow*popcols*indivrows*indivcols + popcol*indivrows*indivcols + indivrow*indivcols + indivcol;
    }
    
    // Randomly draw a genotype given frequencies of AA, Aa, and aa.
    function drawOneGenotype(pp, pq2, qq) {
        let u = Math.random();
        if (u < pp)
            return 0;
        else if (u < pp + pq2)
            return 1;
        else
            return 2;
    }
    
    // Draw n genotypes for subpop at row i, column j
    // and recompute freqA for that subpop using the new genotypes
    function drawNGenotypes(i, j, n) {
        let k = i*popcols + j;
        let p = freqA[k].freq;
        let pp = p*p;
        let pq2 = 2.0*p*(1.0-p);
        let qq = 1.0 - pp - pq2;
        let pcount = 0;
        let qcount = 0;
        let v = [];
        for (k = 0; k < n; k++) {
            let g = drawOneGenotype(pp, pq2, qq);
            v.push(g);
            if (g == 0) {
                pcount++;
                pcount++;
                }
            else if (g == 1) {
                pcount++;
                qcount++;
                }
            else {
                qcount++;
                qcount++;
                }
        }
        let total_count = pcount + qcount;
        freqA[i*popcols + j].freq = pcount/total_count;
        return v;
    }   
    
    function getCellX(popcol, indivcol) {
        return wcell*(popcol*indivcols + indivcol + 0.5);
    }       
    
    function getCellY(poprow, indivrow) {
        return hcell*(poprow*indivrows + indivrow + 0.5);
    }       
    
    // Data for individuals is stored as list of objects containing information about each individual
    var indiv_data = [];
    for (let i = 0; i < poprows; i++) {
        for (let j = 0; j < popcols; j++) {
            let n = indivrows*indivcols;
            let v = drawNGenotypes(i, j, n);
            for (let k = 0; k < indivrows; k++) {
                for (let m = 0; m < indivcols; m++) {
                    let x = getCellX(j, m);
                    let y = getCellY(i, k);
                    indiv_data.push({"i":i, "j":j, "k":k, "m":m, "x":x, "y":y, "genotype":v[k*indivcols + m]});
                }
            }
        }
    }
    
    function nextGeneration() {
        for (let i = 0; i < poprows; i++) {
            for (let j = 0; j < popcols; j++) {
                let n = indivrows*indivcols;
                let v = drawNGenotypes(i, j, n);
                for (let k = 0; k < indivrows; k++) {
                    for (let m = 0; m < indivcols; m++) {
                        let x = getCellX(j, m);
                        let y = getCellY(i, k);
                        let indiv = getDataIndex(i, j, k, m);
                        indiv_data[indiv].genotype = v[k*indivcols + m];
                    }
                }
            }
        }
        d3.selectAll("circle.indiv")
            .attr("cx", function(d) {return d.x;})
            .attr("cy", function(d) {return d.y;})
            .attr("fill", function(d) {return genotype_color[d.genotype];});
    }
    
    // Data for lines separating populations
    var line_data = [];
    for (let i = 0; i < poprows + 1; i++) {
        let x1 = 0;
        let x2 = w;
        let y1 = (h/poprows)*i;
        let y2 = (h/poprows)*i;
        line_data.push({"x1":x1, "x2":x2, "y1":y1, "y2":y2});
    }
    for (let j = 0; j < popcols + 1; j++) {
        let x1 = (w/popcols)*j;
        let x2 = (w/popcols)*j;
        let y1 = 0;
        let y2 = h;
        line_data.push({"x1":x1, "x2":x2, "y1":y1, "y2":y2});
    }

    // Listen and react to keystrokes
    function keyDown() {
        console.log("key was pressed: " + d3.event.keyCode);
        if (d3.event.keyCode == 78) {
            // 78 is the "n" key
            nextGeneration();
        }
    }
    d3.select("body")
        .on("keydown", keyDown);

    // Select DIV element already created (see above) to hold SVG
    var plot_div = d3.select("div#canvas");

    // Create SVG element
    var plot_svg = plot_div.append("svg")
        .attr("width", w)
        .attr("height", h);

    // Create rect outlining entire area of SVG
    plot_svg.append("rect")
        .attr("x", 0)
        .attr("y", 0)
        .attr("width", w)
        .attr("height", h)
        .attr("fill", "lavender");
        
    // Create circles representing individuals
    plot_svg.selectAll("circle.indiv")
        .data(indiv_data)
        .enter()
        .append("circle")
        .attr("class", "indiv")
        .attr("cx", function(d) {return d.x;})
        .attr("cy", function(d) {return d.y;})
        .attr("r", rindiv)
        .attr("fill", function(d) {return genotype_color[d.genotype];})
        .attr("stroke", "none");

    // Create blue line from center of plot area to right edge
    plot_svg.selectAll("line.popbounds")
        .data(line_data)
        .enter()
        .append("line")
        .attr("class", "popbounds")
        .attr("x1", function(d) {return d.x1;})
        .attr("y1", function(d) {return d.y1;})
        .attr("x2", function(d) {return d.x2;})
        .attr("y2", function(d) {return d.y2;})
        .attr("stroke", "black");
</script>
