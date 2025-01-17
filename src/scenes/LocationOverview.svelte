<script>
  import {fade} from 'svelte/transition';
  import {
    location,
    brands,
    cars,
  } from 'stores';
  import {clickOutside} from 'actions';
  import {
    getEvents,
    getRecords,
  } from 'api';
  import {
    formatTime,
    getPIClass,
    PICLASS_FILTERS,
  } from 'common';

  $: hasLocation = !!$location.uuid;
  let events = [];
  let filters = {
    car__brand__uuid: '',
    car__name: '',
    class: '',
  };
  let activeEventIndex;
  $: activeEvent = events[activeEventIndex] || {};
  let records = [];

  const formatRecordValue = (event_kind, value) => {
    switch (event_kind) {
      case 'road_circuit': return formatTime(value)
      default: return value;
    }
  }

  const getLeaderboardEvents = () => {
    getEvents({
      query: {
        'location__uuid': $location.uuid,
      }
    }).then(res => {
      if (!res.ok) {
        console.error('failed to fetch events for location');
        return;
      }

      events = res.body;
      activeEventIndex = 0;
    });
  }

  $: $location.uuid && getLeaderboardEvents();

  const getActiveLeaderboard = () => {

    if (!events[activeEventIndex]) {
      return;
    }

    const queryFilter = Object.entries(filters).reduce((out, [key, value]) => {

      if (value !== '') {
        key === 'class'
          ? Object.assign(out, PICLASS_FILTERS[value])
          : out[key] = value;
      }

      return out;
    }, {})

    getRecords({
      query: {
        'event__uuid': events[activeEventIndex].uuid,
        'ordering': 'value',
        'limit': 100,
        'offset': 0,
        ...queryFilter,
      }
    }).then(res => {
      if (!res.ok) {
        console.error('failed to fetch leaderboard for event');
        return;
      }

      records = res.body.results;
    });
  }

  $: activeEventIndex !== undefined && getActiveLeaderboard();
  $: filters, getActiveLeaderboard();

</script>

<style>
  .layer {
      position: absolute;
      display: flex;
      justify-content: center;
      align-items: center;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
  }

  .card_group {
      display: flex;
      width: 80vw;
  }

  .header_card {
      margin: 0 10px;
      pointer-events: auto;
  }

  .card {
      height: 80vh;
      background-color: white;
      pointer-events: auto;
      margin: 0 10px;
      overflow: hidden;
  }

  h3 {
      margin: 10px;
  }

  .event_card {
      width: 15vw;
  }

  .leaderboard_card {
      flex: 1;
  }

  .details_card {
      width: 15vw;
  }

  .filters {
      background-color: white;
      margin: 0 10px 10px;
      padding: 10px;
  }

  .row {
      display: flex;
      cursor: pointer;
      background: white;
      color: black;
      transition: all 0.2s;
      padding: 10px;
  }

  .active_event {
      background: #ff00a3;
      color: #fff;
  }

  .event:hover {
      cursor: pointer;
      background: #D10086;
      color: #fff;
  }

  .record_grid {
      display: grid;
      grid-template-columns: fit-content(9ch) 2fr 2fr 10ch 100px 100px 50px;
      grid-gap: 10px;
      margin: 5px;
      word-break: break-word;
  }

  .lb_video {
      text-align: right;
  }

  .event_details {
      margin: 10px;
  }
</style>

{#if hasLocation}
  <div class='layer'>
    <div
      use:clickOutside
      on:clickOutside={() => location.set({})}
    >
      <div
        class='card_group'
      >
        <div class='event_card header_card'></div>
        <div class='leaderboard_card header_card filters'>
          Brand
          <select bind:value={filters.car__brand__uuid}>
            <option value="">All</option>
            {#each $brands as brand}
              <option value={brand.uuid}>{brand.name}</option>
            {/each}
          </select>
          Car
          <select bind:value={filters.car__name}>
            <option value="">All</option>
            {#each $cars as car}
              <option value={car.uuid}>{car.name}</option>
            {/each}
          </select>
          Class
          <select bind:value={filters.class}>
            <option value="">All</option>
            <option value='D'>D</option>
            <option value='C'>C</option>
            <option value="B">B</option>
            <option value="A">A</option>
            <option value="S1">S1</option>
            <option value="S2">S2</option>
            <option value="X">X</option>
          </select>
        </div>
        <div class='details_card header_card'></div>
      </div>
      <div
        class='card_group'
      >
        <div
          class='card event_card'
          transition:fade={{duration: 150}}
        >
          <h3>Events</h3>
          {#each events as event, i}
            <div
              class='row event'
              class:active_event={i === activeEventIndex}
              on:click={() => activeEventIndex = i}
            >
              {event.name}
            </div>
          {/each}
        </div>

        <div
          class='card leaderboard_card'
          transition:fade={{duration: 150}}
        >
          <h3>Leaderboard</h3>
          <div class='record_grid'>
            <b>#</b>
            <b>Username</b>
            <b>Car</b>
            <b>Sharecode</b>
            <b>PI</b>
            <b>Time</b>
            <b>Video</b>
            {#each records as record, i}
              <div>{i + 1}</div>
              <div>{record.user}</div>
              <div>{record.car}</div>
              <div>{record.share_code || ''}</div>
              <div>{getPIClass(record.pi)} {record.pi}</div>
              <div>{formatRecordValue($location.kind, record.value)}</div>
              <div class='lb_video'>
                {#if !record.video}
                  <a target="_blank" href='https://www.youtube.com/watch?v=JRjwF8OWNck'>link</a>
                {/if}
              </div>
            {/each}
          </div>
        </div>

        <div
          class='card details_card'
          transition:fade={{duration: 150}}
        >
          <h3>Top tunes</h3>
          <div class='event_details'>
            &#128736 WIP &#128736
          </div>
        </div>

      </div>
    </div>
  </div>
{/if}
