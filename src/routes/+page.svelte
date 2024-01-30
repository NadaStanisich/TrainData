<script lang="ts">
  import { onMount } from "svelte";
  import { Train } from "$lib/train";
  import { Trainstop } from "$lib/trainstop";
  import { TrainDeparture, Departures } from "$lib/traindeparture";
  import { TrainDirections } from "$lib/traindirections";


  let selectedStation: Trainstop | null = null;
  let trainRoute: Train[] = [];
  let stopData: Trainstop[] = [];
  let departureData: TrainDeparture[] = [];
  let directionData: TrainDirections[] = [];
  let departuresData: Departures | undefined = undefined;

  async function fetchData() {
    try {
      // Fetch train data
      const trainResponse = await fetch(
        "https://ptvapiwrapper.azurewebsites.net/trains/get-all-routes",
        {
          method: "GET",
          headers: {
            "Content-type": "application/json;charset=UTF-8",
            "X-Api-Key":
              "pDDuZJyKSfzVU1zRriSw4vWvzpRAoGIAtw0osQPqkwn9MOIpOVpGTeEoMM94jXZw",
          },
        },
      );

      if (trainResponse.ok) {
        const responseData = await trainResponse.json();
        trainRoute = responseData.map(
          (item: any) =>
            new Train(item.route_type, item.route_id, item.route_name),
        );
      } else {
        throw new Error(
          `Failed to fetch train data: ${trainResponse.statusText}`,
        );
      }

      // Fetch stops data
      const stopsResponse = await fetch(
        "https://ptvapiwrapper.azurewebsites.net/trains/get-all-stops",
        {
          method: "GET",
          headers: {
            accept: "application/json",
            "X-Api-Key":
              "pDDuZJyKSfzVU1zRriSw4vWvzpRAoGIAtw0osQPqkwn9MOIpOVpGTeEoMM94jXZw",
          },
        },
      );

      if (stopsResponse.ok) {
        const stopsDataResponse = await stopsResponse.json();
        stopData = stopsDataResponse.map(
          (stop: any) =>
            new Trainstop(
              stop.stop_id,
              stop.stop_name,
              stop.stop_suburb,
              stop.stop_latitude,
              stop.stop_longitude,
              stop.route_type,
            ),
        );
      } else {
        throw new Error(
          `Failed to fetch stops data: ${stopsResponse.statusText}`,
        );
      }

      if (selectedStation) {
        // Fetch train departure data
        const departureResponse = await fetch(
          `https://ptvapiwrapper.azurewebsites.net/trains/get-departures/${selectedStation.stop_id}`,
          {
            method: "GET",
            headers: {
              accept: "application/json",
              "X-Api-Key":
                "pDDuZJyKSfzVU1zRriSw4vWvzpRAoGIAtw0osQPqkwn9MOIpOVpGTeEoMM94jXZw",
            },
          },
        );

        console.log("sadfsdaf")
        

        if (departureResponse.ok) {
          const departureDataResponse = await departureResponse.json();
          departureData = departureDataResponse.map(
            (departure: any) =>
              new TrainDeparture(
                departure.stop_id,
                departure.route_id,
                departure.run_id,
                departure.direction_id,
                departure.scheduled_departure_utc,
                departure.estimated_departure_utc,
                departure.platform_number,
              ),
          );
          console.log(departureData);
        } else {
          throw new Error(
            `Failed to fetch departure data: ${departureResponse.statusText}`,
          );
        }

        // Fetch train direction data
        const directionResponse = await fetch(
          "https://ptvapiwrapper.azurewebsites.net/trains/get-directions",
          {
            method: "GET",
            headers: {
              accept: "application/json",
              "X-Api-Key":
                "pDDuZJyKSfzVU1zRriSw4vWvzpRAoGIAtw0osQPqkwn9MOIpOVpGTeEoMM94jXZw",
            },
          },
        );

        if (directionResponse.ok) {
          const directionDataResponse = await directionResponse.json();
          console.log("DirectionResponse: ", directionDataResponse);
          directionData = directionDataResponse.map(
            (direction: any) =>
              new TrainDirections(
                direction.direction_id,
                direction.route_direction_description,
                direction.direction_name,
                direction.route_id,
                direction.route_type,
              ),
          );
          console.log(directionData);
        } else {
          throw new Error(
            `Failed to fetch direction data: ${directionResponse.statusText}`,
          );
        }
      } //  if (selectedStation)
    } catch (error) {
      console.error(error);
    }
  }

  onMount(() => {
    fetchData(); // Initial data fetch
  });

  // Update data when selectedStation changes
  $: {
    if (selectedStation) {
      fetchNextTrains();
      //fetchData();
    }
    
  }

  async function fetchNextTrains() {
    try {
      if (selectedStation) {
        // Fetch departures for the selected station
        const departuresResponse = await fetch(
          `https://ptvapiwrapper.azurewebsites.net/trains/get-departures/${selectedStation.stop_id}`,
          {
            method: "GET",
            headers: {
              accept: "application/json",
              "X-Api-Key":
                "pDDuZJyKSfzVU1zRriSw4vWvzpRAoGIAtw0osQPqkwn9MOIpOVpGTeEoMM94jXZw",
            },
          },
        );

        if (departuresResponse.ok) {
          departuresData = await departuresResponse.json();
          console.log("DeparturesData: ", departuresData)
          // departureData = departuresData.map(
          //   (departure: any) =>
          //     new TrainDeparture(
          //       departure.stop_id,
          //       departure.route_id,
          //       departure.run_id,
          //       departure.direction_id,
          //       departure.scheduled_departure_utc,
          //       departure.estimated_departure_utc,
          //       departure.platform_number,
          //     ),
          // );
          console.log(departuresData);
        } else {
          throw new Error(
            `Failed to fetch departures data: ${departuresResponse.statusText}`,
          );
        }
      }
    } catch (error) {
      console.error(error);
    }
  }
</script>

<main>
  <h1>Train Station Selector</h1>
  <label for="stations">Select a Station:</label>
  <select id="stations" bind:value={selectedStation}>
    {#each stopData as stop (stop.stop_id)}
      <option value={stop}>{stop.stop_name}</option>
    {/each}
  </select>

  {#if departuresData != undefined}
  {#if departuresData.departures != undefined }
    <h1>Next Trains</h1> {new Date().toUTCString()}
    
    {#each departuresData.departures.slice(27, 32) as departure}
      <p>{departure.platform_number} {departure.scheduled_departure_utc}</p>
    {/each}

  {/if}
{/if}

  {#if selectedStation}
    <h2 class="text-xl font-bold mb-2">Selected Station</h2>
    <p class="font-bold">Station Name: {selectedStation.stop_name}</p>
    <p>Station Suburb: {selectedStation.stop_suburb}</p>
    <p>Route Type: {selectedStation.route_type}</p>
    <div class="container mx-auto p-4">
      <h1 class="text-3xl font-bold mb-4">Train Routes</h1>

      {#if trainRoute.length > 0}
        {#each trainRoute as train (train.route_id)}
          <div>
            <p>
              Route Type: {train.route_type} Route ID: {train.route_id} Route Name:
              {train.route_name}
            </p>
          </div>
        {/each}
      {:else}
        <p>No train data available</p>
      {/if}

      <h1 class="text-3xl font-bold my-4">Train Stops</h1>

      {#if stopData.length > 0}
      {#each stopData.slice(0, 4) as stop (stop.stop_id)}
        <div>
          <p>
            Stop Name: {stop.stop_name} Stop Suburb: {stop.stop_suburb} Route Type:
            {stop.route_type} Latitude: {stop.stop_latitude} Longitude: {stop.stop_longitude}
          </p>
        </div>
      {/each}
    {:else}
      <p>No stop data available</p>
    {/if}

      <h1 class="text-3xl font-bold my-4">Train Departures</h1>

      {#if departureData.length > 0}
        {#each departureData as departure (departure.stop_id)}
          <div>
            <p>
              Stop ID: {departure.stop_id} Route ID: {departure.route_id} Run ID:
              {departure.run_id} Direction ID: {departure.direction_id} Scheduled
              Departure: {departure.scheduled_departure_utc} Estimated Departure:
              {departure.estimated_departure_utc} Platform Number: {departure.platform_number}
            </p>
          </div>
        {/each}
      {:else}
        <p>No departure data available</p>
      {/if}

      <h1 class="text-3xl font-bold my-4">Train Directions</h1>

      {#if directionData.length > 0}
        {#each directionData as direction (direction.direction_id)}
          <div>
            <p>
              Direction ID: {direction.direction_id} Route Direction Description:
              {direction.route_direction_description} Direction Name: {direction.direction_name}
              Route ID: {direction.route_id} Route Type: {direction.route_type}
            </p>
          </div>
        {/each}
      {:else}
        <p>No direction data available</p>
      {/if}
    </div>
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
