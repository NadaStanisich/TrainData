<script lang="ts">
	import { onMount } from 'svelte';
    import { Train } from '$lib/train'; 

        let trainRoute: Train[] = [];

        onMount(async () => {
    try {
        console.log('Fetching data...');
        const response = await fetch('https://ptvapiwrapper.azurewebsites.net/trains/get-all-routes', {
            method: "GET",
            headers: { "Content-type": "application/json;charset=UTF-8",
            "X-Api-Key": "pDDuZJyKSfzVU1zRriSw4vWvzpRAoGIAtw0osQPqkwn9MOIpOVpGTeEoMM94jXZw"}
        })

        console.log('Response:', response);

        if (response.ok) {
            const responseData = await response.json();
            trainRoute = responseData.map((item: any) => new Train(item.route_type, item.route_id, item.route_name));
        } else {
            throw new Error(`Failed to fetch data: ${response.statusText}`);
        }
    } catch (error) {
        console.error(error);
    }
});

</script>

<main>

<h1>Train Routes</h1>

    {#if trainRoute.length > 0}
    {#each trainRoute as train (train.route_id)}
            <div>
                <p>Route Type: {train.route_type}</p>
                <p>Route ID: {train.route_id}</p>
                <p>Route Name: {train.route_name}</p>
            </div>
        {/each}
    {:else}
        <p>No train data available</p>
    {/if}
</main>

        <style>
            .logo {
                height: 6em;
                padding: 1.5em;
                will-change: filter;
                transition: filter 300ms;
            }
            .logo:hover {
                filter: drop-shadow(0 0 2em #646cffaa);
            }
            .logo.svelte:hover {
                filter: drop-shadow(0 0 2em #ff3e00aa);
            }
            .read-the-docs {
                color: #888;
            }
        </style>
       
