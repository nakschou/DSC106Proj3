<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    let data = [];
    let filteredData = [];
    let minBatchIndex = 0;
    let maxBatchIndex = 100; // Set this based on your data
    let selectedMinBatch = minBatchIndex;
    let selectedMaxBatch = maxBatchIndex;

    onMount(async () => {
        data = await d3.json('data/yc_data.json');
        // Dynamically determine the min and max batch numbers
        const batchIndices = data.map(d => d['Batch Number']);
        minBatchIndex = Math.min(...batchIndices);
        maxBatchIndex = Math.max(...batchIndices);
        selectedMinBatch = minBatchIndex;
        selectedMaxBatch = maxBatchIndex;
        filteredData = data;
    });

    $: filteredData = data.filter(d => d['Batch Number'] >= selectedMinBatch && d['Batch Number'] <= selectedMaxBatch);
</script>

<div class="slider-container">
    <div>
        <label for="minBatch">Min Batch: {selectedMinBatch}</label>
        <input type="range" min={minBatchIndex} max={maxBatchIndex} bind:value={selectedMinBatch} id="minBatch">
    </div>
    <div>
        <label for="maxBatch">Max Batch: {selectedMaxBatch}</label>
        <input type="range" min={minBatchIndex} max={maxBatchIndex} bind:value={selectedMaxBatch} id="maxBatch">
    </div>
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
