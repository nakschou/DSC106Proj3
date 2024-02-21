<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    export let data = [];
  
    // Function to create or update the treemap
    function createTreemap(data) {
      const margin = { top: 10, right: 10, bottom: 10, left: 10 },
            width = 800 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;
  
      const svg = d3.select("#treemap")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .style("border", "1px solid black");
  
      const color = d3.scaleOrdinal(d3.schemeCategory10);
  
      const root = d3.hierarchy({children: data})
        .sum(d => d.value)
        .sort((a, b) => b.value - a.value);
  
      d3.treemap()
        .size([width, height])
        .padding(1)
        (root);
  
      const node = svg.selectAll("rect")
        .data(root.leaves(), d => d.data.id);
  
      node.enter().append("rect")
        .attr("x", d => d.x0)
        .attr("y", d => d.y0)
        .attr("width", d => d.x1 - d.x0)
        .attr("height", d => d.y1 - d.y0)
        .style("fill", d => color(d.parent.data.id));
  
      node.attr("x", d => d.x0)
        .attr("y", d => d.y0)
        .attr("width", d => d.x1 - d.x0)
        .attr("height", d => d.y1 - d.y0);
  
      node.exit().remove();
    }
  
    onMount(() => {
      createTreemap(data);
    });
  
    $: if(data) {
      createTreemap(data);
    }
  </script>
  
  <svg id="treemap"></svg>
  