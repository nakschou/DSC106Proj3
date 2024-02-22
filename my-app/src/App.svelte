<!-- App.svelte -->
<script>
    import Table from './Table.svelte';
    import MapboxMap from './MapboxMap.svelte';

    let currentInterface = 'first';

    function switchInterface() {
        currentInterface = currentInterface === 'first' ? 'second' : 'first';
        if (currentInterface === 'first') {
            // Refresh the page when switching from the MapboxMap interface to Treemap
            window.location.reload();
        }
    }

</script>

<!-- <div>
    {#key currentInterface}
        {#if currentInterface === 'first'}
            <Table />
        {/if}

        {#if currentInterface === 'second'}
            <MapboxMap />
        {/if}
    {/key}
</div>



<button on:click={switchInterface}>Switch Interface</button> -->

<Table />