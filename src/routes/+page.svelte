<script lang="ts">
	import { onMount } from 'svelte';
    import { Train } from '$lib/train'; 
    import { Trainstop } from '$lib/trainstop';

        let trainRoute: Train[] = [];
        let stopData: Trainstop[] = [];
        

        onMount(async () => {
    try {
        // Fetch all stops data
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
            throw new Error(`Failed to fetch train data: ${response.statusText}`);
        }

        // Fetch stops data
        const stopsResponse = await fetch('https://ptvapiwrapper.azurewebsites.net/trains/get-all-stops', {
            method: 'GET',
            headers: {
            'accept': 'application/json',
            'X-Api-Key': 'pDDuZJyKSfzVU1zRriSw4vWvzpRAoGIAtw0osQPqkwn9MOIpOVpGTeEoMM94jXZw',
            },
        });

        if (stopsResponse.ok) {
            const stopsData = await stopsResponse.json();
            stopData = stopsData.map((stop: any) =>
            new Trainstop(stop.stop_id, stop.stop_name, stop.stop_suburb, stop.stop_latitude, stop.stop_longitude, stop.route_type)
            );
        } else {
            throw new Error(`Failed to fetch stops data: ${stopsResponse.statusText}`);
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
                <p>Route Type: {train.route_type} Route ID: {train.route_id} Route Name: {train.route_name}</p>
            </div>
        {/each}
    {:else}
        <p>No train data available</p>
    {/if}

    <h1>Train Stops</h1>

  {#if stopData.length > 0}
    {#each stopData as stop (stop.stop_id)}
      <div>
        <p>Stop Name: {stop.stop_name} Stop Suburb: {stop.stop_suburb} Stop Latitude: {stop.stop_latitude} Stop Longitude: {stop.stop_longitude} Route Type: {stop.route_type}</p>
      </div>
    {/each}
  {:else}
    <p>No stop data available</p>
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
       
