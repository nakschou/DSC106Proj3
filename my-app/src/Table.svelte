<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import RangeSlider from 'svelte-range-slider-pips';
    import { Popover, Button } from 'flowbite-svelte';

    let data = [];
    let master = [];
    let filteredData = [];
    let minBatchIndex = 0;
    let maxBatchIndex = 29; // Set this based on your data
    let selectedMinBatch = minBatchIndex;
    let selectedMaxBatch = maxBatchIndex;
    let values = [0,29];
    let popoverTitle = 'Details';
    let popoverContent = 'Hover over an item to see details.';
    let totalmoney = 0;
    let currcat = "All";
    let currstate = 0;
    let currcompany ={};
    let recorded_mouse_position = {
		x: 0, y: 0
	};
    let mouseX = 0;
    let mouseY = 0;


    const batchNames = [
        'S05', 'W06', 'S06', 'W07', 'S07', 'W08', 'S08', 'W09', 'S09', 'W10',
        'S10', 'W11', 'S11', 'W12', 'S12', 'W13', 'S13', 'W14', 'S14', 'W15',
        'S15', 'W16', 'S16', 'W17', 'S17', 'W18', 'S18', 'W19', 'S19', 'W20'
    ];

    const colorMapping = {
        'Other SaaS': '#530089',
        'Entertainment': '#CF0057',
        'Fintech': '#E70056',
        'Consumer': '#390099',
        'Industrial': '#FF5400',
        'Dev Tools': '#FF0054',
        'Real Estate': '#FF3F15',
        'Healthcare': '#9E0059',
        'Education': '#FF8900',
        'Agriculture': '#FFBD00',
        'Transport': '#6C0079',
        'Aerospace': '#FF2A2A',
        'Nonprofit': '#FFD250',
        'Resources': '#FFB000',
        'Government': '#FFA300'
    };

    const tooltip = d3.select("#tool")
    .append("div")
    .style("opacity", 1)
    .attr("class", "tooltip")
    .style("background-color", "white")
    .style("border", "solid")
    .style("border-width", "1px")
    .style("border-radius", "5px")
    .style("padding", "10px")



    // A function that change this tooltip when the user hover a point.
    // Its opacity is set to 1: we can now see it. Plus it set the text and position of tooltip depending on the datapoint (d)
    const mouseover = function(d) {
        tooltip
        .style("opacity", 1)
    }

    const mousemove = function(d) {
        tooltip
        .html("The exact value of<br>the Ground Living area is: " + d.id)
        .style("left", (d3.pointer(this)[0]+90) + "px") // It is important to put the +90: other wise the tooltip is exactly where the point is an it creates a weird effect
        .style("top", (d3.pointer(this)[1]) + "px")
    }

    // A function that change this tooltip when the leaves a point: just need to set opacity to 0 again
    const mouseleave = function(d) {
        tooltip
        .transition()
        .duration(200)
        .style("opacity", 1)
    }


    onMount(async () => {
        data = await d3.json('data/yc_data_cleaned.json');
        master = data;
        // Dynamically determine the min and max batch numbers
        const batchIndices = data.map(d => d['Batch Number']);
        minBatchIndex = Math.min(...batchIndices);
        maxBatchIndex = Math.max(...batchIndices);
        selectedMinBatch = values[0];
        selectedMaxBatch = values[1];
        filteredData = data;
    });

    $: filteredData = data.filter(d => d['Batch Number'] >= values[0] && d['Batch Number'] <= values[1]);
    
    function handleMouseMove(event) {
        mouseX = event.clientX;
        mouseY = event.clientY;
    }
    // Function to create or update the treemap
    function createTreemap(data) {
        // change total money
        totalmoney = data.reduce((acc, d) => acc + d.value, 0);

        const margin = { top: 10, right: 10, bottom: 10, left: 10 },
            width = 1200 - margin.left - margin.right,
            height = 600 - margin.top - margin.bottom;

        const svg = d3.select("#treemap")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        //.style("border", "1px solid black");

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
        .attr("width", 0)
        .attr("height", 0)
        .attr("id", d => `rect_${d.data.id}`)
        .style("fill", d => (d.children ? generateColorScale(d.parent.data.id, d.data.id) : colorMapping[d.data.id] || '#999'))
        .text(d => d.data.id)
        .on("mouseover", function(event, d) {
            const nodeId = d.data.id; // Assuming 'id' is the property holding the ID.
            popoverTitle = nodeId;
            popoverContent = `Value: ${formatMoney(d.data.value)}`;
            this.dispatchEvent(new CustomEvent('node-hover', {
                detail: { id: nodeId },
                bubbles: true // This makes the event bubble up through the DOM
            }));
            let newcolor = d3.hsl(colorMapping[d.data.id]);
            newcolor.l -= 0.2;
            d3.select(this).style("fill", d => newcolor || '#999');
            if (currstate === 1) {
                currcompany = getCompanyInfo(d.data.id, master)
            }
            recorded_mouse_position = {
							x: event.x,
							y: event.y
						}


        })
        .on("mouseout", function(d) {
            d3.select(this).style("fill", d => colorMapping[d.data.id] || '#999');
            mouseleave;
        })
        .on("mousemove", function(event, d) {
            recorded_mouse_position = {
                x: d3.pointer(this)[0],
                y: d3.pointer(this)[1]
            }

        })
        .on("click", function(event, d) {
            if (currstate === 0) {
                currcat = d.data.id; // Update currcat to the clicked category
                currstate = 1; // Change state to show breakdown by company
            } else {
                currcat = "All"; // Reset currcat to "All"
                currstate = 0; // Change state to show total funding by category
            }
            updateTreemap(); // Call a function to update the treemap with the new data
        }).merge(node).transition().duration(500) // Apply a transition for the entering elements
        .attr("width", d => d.x1 - d.x0)
        .attr("height", d => d.y1 - d.y0);

        const text = svg.selectAll("text")
        .data(root.leaves(), d => d.data.id);

        text.exit().remove();


        text.enter().append("text")
        .attr("x", d => d.x0 + 10)
        .attr("y", d => d.y0 + 20)
        .text(function(d) {
            if(d.x1 - d.x0 > 200) {
                return `${d.data.id}\n${formatMoney(d.data.value)}`;
            }else {
                return ".";
            }
        })
        .attr("font-size", "0px")
        .attr("fill", "white")
        .merge(text).transition().duration(500)
        .attr("font-size", "15px")
        .attr("x", d => d.x0 + 0.1*(d.x1 - d.x0))
        .attr("y", d => d.y0 + 0.5*(d.y1 - d.y0));


        svg.selectAll("rect")
        .data(root.leaves(), d => d.data.id).enter()
        .on("mouseover", mouseover)
        .on("mousemove", mousemove )
        .on("mouseleave", mouseleave )

        node.transition().duration(500)
        .attr("x", d => d.x0)
        .attr("y", d => d.y0)
        .attr("width", d => d.x1 - d.x0)
        .attr("height", d => d.y1 - d.y0)
        .attr("id", d => `rect_${d.data.id}`)
        .text("testing")
        .style("fill", d => colorMapping[d.data.id] || '#999');

        node.exit()
        .transition().duration(500)
        .attr("width", 0) // Transition to 0 width and height for a shrink effect
        .attr("height", 0)
        .remove();

    }

    // Function to generate a color scale based on the parent category's color
    function generateColorScale(parentColor, childId) {
        const startColor = parentColor || '#999'; // Default to gray if parentColor is not defined
        const endColor = 'white';

        const colorScale = d3.scaleLinear()
            .domain([0, 1])
            .range([startColor, endColor]);

        // Calculate the position of the child within its parent's hierarchy
        const positionInHierarchy = getParentHierarchyPosition(childId);

        // Use the color scale to get the color based on the position
        return colorScale(positionInHierarchy);
    }

    // Function to calculate the position of a child within its parent's hierarchy
    function getParentHierarchyPosition(childId) {
        const parentIndex = data.findIndex(item => item.id === childId);
        const totalCategories = Object.keys(colorMapping).length;

        return parentIndex / (totalCategories - 1); // Normalize to [0, 1]
    }

    onMount(() => {
        createTreemap(data);
    });

    function updateTreemap() {
        if (currstate === 1) {
            const categoryData = filteredData.filter(d => d.Category === currcat);
            createTreemap(convertToCompanyFundingWithinCategory(categoryData));
        } else {
            createTreemap(convertToSummedCategoriesWithNullHandling(filteredData));
        }
    }

    function getCompanyInfo(companyName, data) {
        for (let i = 0; i < data.length; i++) {
            const companyData = data[i]; // Assuming the keys are numbers
            if (companyData['Company Name'] === companyName) {
                return companyData;
            }
        }
        //return null; // Company not found
    }

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

    function convertToCompanyFundingWithinCategory(dataList) {
        const companySums = {};
        dataList.forEach(item => {
            const fundingToAdd = item.Funding === null ? 0 : item.Funding;
            const companyName = item["Company Name"]; // Assuming you have a CompanyName field
            if (companySums[companyName]) {
                companySums[companyName] += fundingToAdd;
            } else {
                companySums[companyName] = fundingToAdd;
            }
        });
        const result = Object.keys(companySums).map(key => {
            return { id: key, value: companySums[key] };
        });
        return result;
    }
    
    function displayCompany(compinfo) {
        if (compinfo) {
            return compi
        }
    }


    $: if(filteredData && currstate === 0) {
        createTreemap(convertToSummedCategoriesWithNullHandling(filteredData));
    } else if(filteredData && currstate === 1) {
        // Filter data for the selected category and aggregate by company
        const categoryData = filteredData.filter(d => d.Category === currcat);
        createTreemap(convertToCompanyFundingWithinCategory(categoryData));
    }

</script>

<div class="slider-container">
    
    <label id="b1" for="maxBatch">Newest Batch: {batchNames[values[1]]}</label>
    <label for="minBatch">Oldest Batch: {batchNames[values[0]]}</label>
    <RangeSlider formatter={v => batchNames[v]} range min={0} max={29} pips all="label" bind:values/>
</div>



<svg id="treemap"></svg>
<div 
    class={currstate === 1 ? "company-card": ""}
    style="left: {recorded_mouse_position.x + 40}px; top: {recorded_mouse_position.y + 40}px"


>
    {#if currstate === 1}
        <h2>{ currcompany ? currcompany["Company Name"] : "nothing" }</h2>
        <p>Batch: { currcompany["Batch"] ? currcompany["Batch"] : "Not Available"}</p>
        <p>Funding Amount: { currcompany["Funding"] ? formatMoney(currcompany["Funding"]) : "Not Available"}</p>
        <p>Location: { currcompany["Location"] ? currcompany["Location"] : "Not Available"}</p>
        <p>Status: { currcompany["Status"] ? currcompany["Status"] : "Not Available"}</p>
    {/if}
</div>



<style>
    .slider-container {
        display: flex;
        flex-direction: column;
        gap: 10px;
    }

    /* Add your styling here */
  .company-card {
    border: 1px solid #ddd;
    padding: 10px;
    margin: 10px;
    border-radius: 8px;
    width: 300px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }

  h2 {
    margin-bottom: 8px;
    margin: 5px;
    color: #333;
  }

  p {
    margin: 5px;
    color: #777;
  }
</style>
