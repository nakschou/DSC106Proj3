<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import RangeSlider from 'svelte-range-slider-pips';
    import { Popover, Button } from 'flowbite-svelte';

    let data = [];
    let filteredData = [];
    let minBatchIndex = 0;
    let maxBatchIndex = 29; // Set this based on your data
    let selectedMinBatch = minBatchIndex;
    let selectedMaxBatch = maxBatchIndex;
    let values = [0,29];
    let popoverTitle = 'Details';
    let popoverContent = 'Hover over an item to see details.';
    let totalmoney = 0;


    onMount(async () => {
        data = await d3.json('data/yc_data_cleaned.json');
        // Dynamically determine the min and max batch numbers
        const batchIndices = data.map(d => d['Batch Number']);
        minBatchIndex = Math.min(...batchIndices);
        maxBatchIndex = Math.max(...batchIndices);
        selectedMinBatch = values[0];
        selectedMaxBatch = values[1];
        filteredData = data;
    });

    $: filteredData = data.filter(d => d['Batch Number'] >= values[0] && d['Batch Number'] <= values[1]);
  
    // Function to create or update the treemap
    function createTreemap(data) {
        // change total money
        totalmoney = data.reduce((acc, d) => acc + d.value, 0);

        const margin = { top: 10, right: 10, bottom: 10, left: 10 },
            width = 800 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;

        const svg = d3.select("#treemap")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .style("border", "1px solid black");

        const color = d3.scaleOrdinal(d3.schemeCategory10);
        //const color = d3.scaleOrdinal().domain(data.map(d => d.id)).range(d3.schemeCategory10);

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
        .attr("id", d => `rect_${d.data.id}`)
        .style("fill", d => color(d.data.id))
        .text(function(d){ return d.parent.data.value})
        .on("mouseover", function(event, d) {
            const nodeId = d.data.id; // Assuming 'id' is the property holding the ID.
            popoverTitle = nodeId;
            popoverContent = `Value: ${formatMoney(d.data.value)}`;
            this.dispatchEvent(new CustomEvent('node-hover', {
                detail: { id: nodeId },
                bubbles: true // This makes the event bubble up through the DOM
            }));
            let newcolor = d3.hsl(color(d.data.id));
            newcolor.l -= 0.2;
            d3.select(this).style("fill", newcolor);
        })
        .on("mouseout", function(d) {
            d3.select(this).style("fill", d => color(d.data.id));
        }).merge(node)

        node.attr("x", d => d.x0)
        .attr("y", d => d.y0)
        .attr("width", d => d.x1 - d.x0)
        .attr("height", d => d.y1 - d.y0)
        .attr("id", d => `rect_${d.data.id}`)
        .style("fill", d => color(d.data.id));

        node.exit().remove();
    }

    onMount(() => {
        createTreemap(data);
    });

    function formatMoney(value) {
        // Convert value to a number in case it's passed as a string
        const num = Number(value);

        // Determine the suffix and calculate the final value based on the magnitude
        let suffix = 'M';
        let finalValue = num;
        if (num >= 1000) { // For billions
            suffix = 'B';
            finalValue = num / 1000;
        }

        // Format the number to always have three decimal places
        // Note: toLocaleString rounds the number, not just truncates it, providing a more accurate representation
        const formattedNumber = finalValue.toLocaleString('en-US', {minimumFractionDigits: 3, maximumFractionDigits: 3});

        return `$${formattedNumber}${suffix}`;
    }

    function convertToSummedCategoriesWithNullHandling(dataList) {
        const categorySums = {};

        // Sum funding amounts by category, treating null as 0
        dataList.forEach(item => {
            const fundingToAdd = item.Funding === null ? 0 : item.Funding;
            if (item.Category == null) {
            } else{
                if (categorySums[item.Category]) {
                    // If category exists, add to its funding
                    categorySums[item.Category] += fundingToAdd;
                } else {
                    // If category doesn't exist, initialize it with funding or 0 if null
                    categorySums[item.Category] = fundingToAdd;
                }
            }
        });

        // Convert the result to the desired array format
        const result = Object.keys(categorySums).map(key => {
            return { id: key, value: categorySums[key] };
        });

        return result;
    }


    $: if(filteredData) {
        
        createTreemap(convertToSummedCategoriesWithNullHandling(filteredData));
    }

</script>

<div class="slider-container">
    
    <label id="b1" for="maxBatch">Max Batch: {values[1]}</label>
    <label for="minBatch">Min Batch: {values[0]}</label>
    <RangeSlider range min={0} max={29} pips all="label" bind:values/>
</div>
<Popover class="w-64 text-sm font-light" title={popoverTitle} triggeredBy="#treemap">
    {popoverContent}
</Popover>


<svg id="treemap"></svg>

<style>
    .slider-container {
        display: flex;
        flex-direction: column;
        gap: 10px;
    }
</style>
