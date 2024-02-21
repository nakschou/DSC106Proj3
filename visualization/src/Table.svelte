<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import RangeSlider from 'svelte-range-slider-pips';

    let data = [];
    let filteredData = [];
    let minBatchIndex = 0;
    let maxBatchIndex = 29; // Set this based on your data
    let selectedMinBatch = minBatchIndex;
    let selectedMaxBatch = maxBatchIndex;
    let values = [0,29];

    onMount(async () => {
        data = await d3.json('data/yc_data.json');
        // Dynamically determine the min and max batch numbers
        const batchIndices = data.map(d => d['Batch Number']);
        minBatchIndex = Math.min(...batchIndices);
        maxBatchIndex = Math.max(...batchIndices);
        selectedMinBatch = values[0];
        selectedMaxBatch = values[1];
        filteredData = data;
    });

    $: filteredData = data.filter(d => d['Batch Number'] >= values[0] && d['Batch Number'] <= values[1]);

</script>

<div class="slider-container">
    
    <label for="maxBatch">Max Batch: {values[1]}</label>
    <label for="minBatch">Min Batch: {values[0]}</label>
    <RangeSlider range min={0} max={29} pips all="label" bind:values/>
</div>

<table>
    <thead>
        <tr>
            <th>Company Name</th>
            <th>Status</th>
            <th>Alexa Rank</th>
            <th>Batch Number</th>
            <!-- Add more headers as needed -->
        </tr>
    </thead>
    <tbody>
        {#each filteredData as company}
            <tr>
                <td>{company['Company Name']}</td>
                <td>{company.Status}</td>
                <td>{company['Alexa Rank']}</td>
                <td>{company['Batch Number']}</td>
                <!-- Add more cells as needed -->
            </tr>
        {/each}
    </tbody>
</table>

<style>
    .slider-container {
        display: flex;
        flex-direction: column;
        gap: 10px;
    }
    table {
        width: 100%;
        border-collapse: collapse;
    }
    th, td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
    }
    th {
        background-color: #f4f4f4;
    }
    input[type="range"] {
        width: 100%;
        margin: 10px 0;
    }
</style>
