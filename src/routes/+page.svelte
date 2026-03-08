<script lang="ts">
  import { onMount } from 'svelte';
  import InputBlock from '$lib/InputBlock.svelte';
  import InputNormalizeOnBlur from '$lib/InputNormalizeOnBlur.svelte';
  import TimeInput from '$lib/TimeInput.svelte';
  import { browser } from '$app/environment';
    import QuickSelection from '$lib/QuickSelection.svelte';

  const KM_PER_MILE = 1.60934;
  const MIN_MI = 'min/mi';
  const MIN_KM = 'min/km';

  function formatDistance(distance: number) {
    return Math.round(distance * 100) / 100;
  }

  function parseDistance(distance: string) {
    return parseFloat(distance) || 0;
  }

  function deriveTime(input_miles: number, output_miles: number, input_time: number) {
    return input_time * (output_miles / input_miles) ** 1.06;
  }

  let paceUnits = $state(MIN_MI);

  onMount(() => {
    if (browser) {
      paceUnits = localStorage.getItem('paceUnits') || paceUnits;
    }
  });

  // only three values are given
  let distance1miles = $state(10 / KM_PER_MILE);
  let time1seconds = $state(60 * 60);
  let distance2miles = $state(10 / KM_PER_MILE);

  // everything else is always derived
  let distance1km = $derived(distance1miles && distance1miles * KM_PER_MILE);
  let pace1miles = $derived(distance1miles && time1seconds && time1seconds / distance1miles);
  let pace1km = $derived(pace1miles && pace1miles / KM_PER_MILE);

  let distance2km = $derived(distance2miles && distance2miles * KM_PER_MILE);
  let time2seconds = $derived(
    distance1miles &&
      distance2miles &&
      time1seconds &&
      deriveTime(distance1miles, distance2miles, time1seconds),
  );
  let pace2miles = $derived(distance2miles && time2seconds && time2seconds / distance2miles);
  let pace2km = $derived(pace2miles && pace2miles / KM_PER_MILE);

  $effect(() => {
    localStorage.setItem('paceUnits', paceUnits);
  });

  function swap() {
    time1seconds = deriveTime(distance1miles, distance2miles, time1seconds)
    const distance1milesCopy = distance1miles
    distance1miles = distance2miles
    distance2miles = distance1milesCopy
  }
</script>

<form class="race1">
  <h2>Race A</h2>

  <InputBlock name="distance">
    <InputNormalizeOnBlur
      name="miles"
      bind:value={distance1miles}
      toString={formatDistance}
      fromString={parseDistance}
      unit="mi"
      width="7rem"
    />

    <InputNormalizeOnBlur
      name="km"
      bind:value={() => distance1km, (v) => (distance1miles = v / KM_PER_MILE)}
      toString={formatDistance}
      fromString={parseDistance}
      unit="km"
      width="7rem"
    />
  </InputBlock>

  <QuickSelection setKm={(v: number) => distance1miles = v / KM_PER_MILE} />

  <InputBlock name="time">
    <TimeInput
      name="time"
      bind:value={() => time1seconds, (v) => (time1seconds = v)}
      withHours
      unit=""
    />
  </InputBlock>

  <InputBlock name="pace">
    {#if paceUnits == MIN_KM}
      <TimeInput
        name="pace_km"
        bind:value={() => pace1km, (v) => (time1seconds = v * distance1km)}
        unit={MIN_KM}
      />
    {:else}
      <TimeInput
        name="pace_mi"
        bind:value={
          () => pace1miles,
          (v) => {
            time1seconds = v * distance1miles;
          }
        }
        unit={MIN_MI}
      />
    {/if}
  </InputBlock>

  <div class="swap" onclick={swap}>
    ↑↓
  </div>
</form>

<form class="race2">
  <h2>Race B</h2>

  <InputBlock name="distance">
    <InputNormalizeOnBlur
      name="mi"
      bind:value={distance2miles}
      toString={formatDistance}
      fromString={parseDistance}
      unit="mi"
      width="7rem"
    />

    <InputNormalizeOnBlur
      name="km"
      bind:value={() => distance2km, (v) => (distance2miles = v / KM_PER_MILE)}
      toString={formatDistance}
      fromString={parseDistance}
      unit="km"
      width="7rem"
    />
  </InputBlock>

  <QuickSelection setKm={(v: number) => distance2miles = v / KM_PER_MILE} />

    <InputBlock name="time">
    <TimeInput
      name="time"
      bind:value={
        () => time2seconds, (v) => (time1seconds = deriveTime(distance2miles, distance1miles, v))
      }
      withHours
      disabled
    />
  </InputBlock>

  <InputBlock name="pace">
    {#if paceUnits == MIN_KM}
      <TimeInput
        name="pace_km"
        bind:value={
          () => pace2km,
          (v) => (time1seconds = deriveTime(distance2miles, distance1miles, v * distance2km))
        }
        unit={MIN_KM}
        disabled
      />
    {:else}
      <TimeInput
        name="pace_mi"
        bind:value={
          () => pace2miles,
          (v) => (time1seconds = deriveTime(distance2miles, distance1miles, v * distance2miles))
        }
        unit={MIN_MI}
        disabled
      />
    {/if}
  </InputBlock>
</form>

<div class="footer">
  Show pace in <span class="unit-selection"
    ><span class={paceUnits == MIN_MI ? 'selected' : ''} onclick={(paceUnits = MIN_MI)}>{MIN_MI}</span
    >
    <span class={paceUnits == MIN_KM ? 'selected' : ''} onclick={(paceUnits = MIN_KM)}>{MIN_KM}</span
    ></span
  >
</div>

<style>
  form {
    font-family:
      system-ui,
      -apple-system,
      BlinkMacSystemFont,
      'Open Sans',
      'Helvetica Neue',
      sans-serif;
    font-size: 1.6rem;
  }

  form,
  .footer {
    padding: 1.6rem 2rem 2.6rem;
    position: relative;
  }

  .footer {
    padding-top: 0;
  }

  .race1 {
    background-color: #f3f3f3;
  }

  .race2,
  .footer {
    background-color: #333;
    color: #f3f3f3;
  }

  h2 {
    font-size: 2.5rem;
    margin-bottom: 1.2rem;
    border-bottom: 1px solid #bbb;
  }

  .swap {
    position: absolute;
    left: calc(50% - 1.555rem);
    bottom: -1.5rem;
    z-index: 1;
    border: 1px solid #bbb;
    background-color: #f3f3f3;
    border-radius: 0.5rem;
    padding: 0.3rem;
    letter-spacing: -0.1rem;
    cursor: pointer;
  }

  .footer {
    font-size: 1.2rem;
    text-align: right;
  }

  .unit-selection {
    font-size: 0;
  }

  .unit-selection span {
    font-size: 1.2rem;
    color: #aaa;
    border: 1px solid #bbb;
    border-radius: 0.5rem;
    padding: 0.2rem 0.4rem;
    cursor: pointer;
  }

  .unit-selection span.selected {
    color: #f3f3f3;
    background-color: #777;
  }

  .unit-selection span:first-child {
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
    border-right: none;
  }

  .unit-selection span:last-child {
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
  }
</style>
