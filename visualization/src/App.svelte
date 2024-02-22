<script>
    import Table from './Table.svelte';
    import MapboxMap from './MapboxMap.svelte';
    import Treemap from './Treemap.svelte';

    let currentInterface = 'first';

    function switchInterface() {
        currentInterface = currentInterface === 'first' ? 'second' : 'first';
    }

    // Sample data
    let data = [
        {id: 'node1', value: 10},
        {id: 'node2', value: 15},
        // Add more nodes as needed
    ];

    // Function to simulate data updates
    function updateData() {
        data = [
            // Update with new data
            {id: 'node1', value: Math.random() * 100},
            {id: 'node2', value: Math.random() * 100},
            // More nodes...
        ];
    }
</script>

<Table />
<button on:click={updateData}>Update Data</button>

{#key currentInterface}
    {#if currentInterface === 'first'}
        <Treemap {data} />
    {/if}

    {#if currentInterface === 'second'}
        <MapboxMap />
    {/if}
{/key}

<button on:click={switchInterface}>Switch Interface</button>
