---
layout: applet
title: Stick Breaking Process
permalink: /applets/stickbreaking/
---

## Stick Breaking Process and Dirichlet Process Priors

Before the stick has been broken the first time, the remainder has length 1 (i.e. all of the stick).
Each subsequent break affects the remainder only. Larger values of alpha lead to more, smaller sticks.

<div id="canvas"></div>
<script type="text/javascript">
    // The MIT License (MIT)
    //
    // Copyright (c) 2020 Paul O. Lewis
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

    // Written by Paul O. Lewis 21-Mar-2020

    // width and height of svg
    var w = 600;
    var h = 220;
    var bm = 20;

    var alpha = 1.0;
    var alphamin = 1.0;
    
    var ndarts = 100;
    var ndartsmin = 10;
    var ndartsmax = 100;
    var dart_radius = 3;
    var num_occupied = 0;
    var somebad = false;
    
    var sticks = [];
    var darts = [];
    var Ek = [];
    var remainder = 1.0;
    var remainder_cutoff = 0.001;
    
    var lot = new Random();
    
    var xscale = d3.scaleLinear()
        .domain([0,1])
        .range([0,w]);

    var yscale = d3.scaleLinear()
        .domain([0,1])
        .range([h-bm,0]);

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
        .attr("height", h-bm)
        .attr("fill", "lavender");
        
    // Create text showing current value of alpha
    plot_svg.append("text")
        .attr("id", "alpha")
        .attr("x", w/2)
        .attr("y", h - bm + 16)
        .attr("font-family", "Verdana")
        .attr("font-size", "16px")
        .style("text-anchor", "middle")
        .text("");     
        
    function precomputeExpectedNumberOccupiedTables() {
        // Compute expected number of occupied tables Ek
        Ek = [];
        var cum = 0.0;
        Ek.push(0.0);
        for (let i = 0; i < 10*ndartsmax; i++) {
            cum += alpha/(alpha + i);
            Ek.push(cum);
        }                
    }
    precomputeExpectedNumberOccupiedTables();
        
    function showStatus() {
        if (somebad) {
            plot_svg.select("text#alpha")
                .text("alpha = " + alpha.toFixed(1) + " | n = " + ndarts + " | E = " + Ek[ndarts].toFixed(1) + " | O = NA (break more sticks)");            
        }
        else {
            plot_svg.select("text#alpha")
                .text("alpha = " + alpha.toFixed(1) + " | n = " + ndarts + " | E = " + Ek[ndarts].toFixed(1) + " | O = " + num_occupied);            
        }
    }       
    showStatus();
        
    function reset() {
        // Delete all existing sticks
        plot_svg.selectAll("rect.stick").remove();
        sticks = [];
        
        // Delete all existing darts
        plot_svg.selectAll("circle.dart").remove();
        darts = [];

        num_occupied = 0;
        somebad = false;
        remainder = 1.0;
        
        precomputeExpectedNumberOccupiedTables();
        showStatus();
    }
    
    function breakStick() {
        // Draw Beta(1,alpha) random variable
        let x = lot.gamma(1, 1);
        let y = lot.gamma(alpha, 1);
        let s = x/(x + y);
        let stickx = 1.0 - remainder; 
        let stickw = s*remainder;
        let r = Math.floor(lot.uniform(0,255));
        let g = Math.floor(lot.uniform(0,255));
        let b = Math.floor(lot.uniform(0,255));
        var stick = {"x":stickx, "width":stickw, "color":d3.color("rgba(" + r + ", " + g + ", " + b + ", 1)")};

        // console.log("~~~~~~~~~~~~~~~~~~~~~");
        // console.log("alpha     = " + alpha);
        // console.log("s         = " + s);
        // console.log("remainder = " + remainder);
        // console.log("stickx    = " + stickx);
        // console.log("stickw    = " + stickw);

        sticks.push(stick);
        plot_svg.selectAll("rect.stick")   
            .data(sticks)
            .enter()
            .append("rect")
            .attr("class", "stick")
            .attr("x", function(d) {return xscale(d.x);})
            .attr("y", 0)
            .attr("width", function(d) {return xscale(d.width);})
            .attr("height", h-bm)
            .attr("fill", function(d) {return d.color;})
            .attr("stroke", "white");
        remainder -= stickw;
    }
    
    function partition() {
        while (remainder > remainder_cutoff) {
            breakStick();
        }
    }

    function throwDarts() {
        plot_svg.selectAll("circle.dart").remove();
        
        // Create one bin for every stick
        bins = [];
        let cum = 0.0;
        for (let i = 0; i < sticks.length; i++) {
            cum += sticks[i].width;
            bins.push(0);
        }
        
        somebad = false;
        darts = [];
        for (let i = 0; i < ndarts; i++) {
            let cx = lot.uniform(0,1);
            let cy = lot.uniform(0,1);
            let isbad = cx > cum ? true : false;

            let dartcolor = "red";
            if (isbad) {
                somebad = true;
            }
            else {
                let scum = 0.0;
                for (let s = 0; s < sticks.length; s++) {
                    scum += sticks[s].width;
                    if (cx < scum) {
                        bins[s] += 1;
                        dartcolor = sticks[s].color;
                        break;
                    }
                }
            }
            darts.push({"cx":cx, "cy":cy, "color":dartcolor});
        }
        
        // Determine how many bins have darts in them
        num_occupied = 0;
        for (let i = 0; i < bins.length; i++) {
            if (bins[i] > 0)
                num_occupied += 1;
        }
        showStatus();
        
        plot_svg.selectAll("circle.dart")   
            .data(darts)
            .enter()
            .append("circle")
            .attr("class", "dart")
            .attr("cx", function(d) {return xscale(d.cx);})
            .attr("cy", function(d) {return yscale(d.cy);})
            .attr("r", dart_radius)
            .attr("fill", function(d) {return d.color;})
            .attr("stroke", "white");
    }
    
    function modifyAlpha(increment) {
        alpha += increment;
        if (alpha < alphamin)
            alpha = alphamin;
        reset();
    }

    function modifySampleSize(increment) {
        ndarts += increment;
        if (ndarts < ndartsmin)
            ndarts = ndartsmin;
        if (ndarts > ndartsmax)
            ndarts = ndartsmax;
        reset();
    }

    // Listen and react to keystrokes
    function keyDown() {
        //console.log("key was pressed: " + d3.event.keyCode);
        if (d3.event.keyCode == 84 || d3.event.keyCode == 68) {
            // 68 is the "d" key
            // 84 is the "t" key
            throwDarts();
        }
        else if (d3.event.keyCode == 66) {
            // 66 is the "b" key
            breakStick();
        }
        else if (d3.event.keyCode == 82) {
            // 82 is the "r" key
            reset();
        }
        else if (d3.event.keyCode == 38) {
            // 38 is the "up arrow" key
            modifyAlpha(1);
        }
        else if (d3.event.keyCode == 40) {
            // 40 is the "down arrow" key
            modifyAlpha(-1);
        }
        else if (d3.event.keyCode == 37) {
            // 37 is the "left arrow" key
            modifySampleSize(-10);
        }
        else if (d3.event.keyCode == 39) {
            // 39 is the "right arrow" key
            modifySampleSize(10);
        }
        else if (d3.event.keyCode == 80) {
            // 80 is the "p" key
            partition();
        }
    }
    d3.select("body")
        .on("keydown", keyDown);
</script>

Notes:
* This applet was written for the graduate course EEB-5349 (Phylogenetics), not EEB-2245 (Evolution), so don't panic if you are an EEB2245 student and have encountered this by accident!
* b key breaks off fraction from the remainder (lavender) using a draw from Beta(1,alpha)
* p key mimics pressing the b key until the remainder is tiny (0.001)
* r key resets everything
* t key throws darts and computes the O statistic
* up/down arrow keys increase/decrease alpha (but smallest value is 1)
* right/left arrow keys increase/decrease the number of darts (within the range 10-100)
* E is the expected number of colored rectangles hit by at least one of the n darts
* O is the observed number of colored rectangles hit by at least one dart

This process provides a prior distribution known as the Dirichlet Process Prior (DPP). Imagine that the darts are sites and the colors of the rectangles represent different relative substitution rates. The stick breaking process illustrated in this applet shows what typical draws from a Dirichlet Process Prior look like for different values of alpha and different numbers of sites (darts). Used in a Bayesian MCMC analysis, a DPP would allow you to learn something about how many rate categories are present and which sites fall into which category. Normally, a hierarchical model would be used in which alpha is a hyperparameter and its hyperprior would be vague but nevertheless discourage alpha from getting too large.