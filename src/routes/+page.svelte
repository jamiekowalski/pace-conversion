<script lang="ts">
	import InputNormalizeOnBlur from "$lib/InputNormalizeOnBlur.svelte";
    import TimeInput from "$lib/TimeInput.svelte";

    const KM_PER_MILE = 1.60934

    function formatDistance(distance: number) {
        return Math.round(distance * 100) / 100
    }

    function deriveTime(input_miles: number, output_miles: number, input_time: number) {
        return input_time * (output_miles / input_miles)**(1.06)
    }

    // only three values are given
    let distance1miles = $state(10)
    let time1seconds = $state(60 * 60)
    let distance2miles = $state(10)

    // everything else is always derived
    let distance1km = $derived(distance1miles && (distance1miles * KM_PER_MILE))
    let pace1miles = $derived(distance1miles && time1seconds && (time1seconds / distance1miles))
    let pace1km = $derived(pace1miles && (pace1miles / KM_PER_MILE))

    let distance2km = $derived(distance2miles && (distance2miles * KM_PER_MILE))
    let time2seconds = $derived(distance1miles && distance2miles && time1seconds && deriveTime(distance1miles, distance2miles, time1seconds))
    let pace2miles = $derived(distance2miles && time2seconds && (time2seconds / distance2miles))
    let pace2km = $derived(pace2miles && (pace2miles / KM_PER_MILE))

</script>

<form class="race1">
<h2>Actual race</h2>

<div class="segment">

<InputNormalizeOnBlur
  name="miles"
  bind:value={distance1miles}
  toString={formatDistance}
  fromString={parseFloat}
/>

<InputNormalizeOnBlur
  name="km"
  bind:value={
  () => distance1km,
  (v) => distance1miles = v / KM_PER_MILE
  }
  toString={formatDistance}
  fromString={parseFloat}
/>
</div>

<div class="segment">
<TimeInput
name="time"
bind:value={
    () => time1seconds,
    (v) => time1seconds = v
} withHours />
</div>

<div class="segment">
<TimeInput
name="pace miles"
bind:value={
    () => pace1miles,
    (v) => {
        console.log(v)
        time1seconds = v * distance1miles
    }
} />

<TimeInput
name="pace km"
bind:value={
    () => pace1km,
    (v) => time1seconds = v * distance1km
} />
</div>
</form>

<form class="race2">
<h2>Predicted race</h2>

<div class="segment">
<InputNormalizeOnBlur
name="miles"
bind:value={distance2miles}
  toString={formatDistance}
  fromString={parseFloat}
/>

<InputNormalizeOnBlur
name="km"
bind:value={
    () => distance2km,
    (v) => distance2miles = v / KM_PER_MILE
  }
  toString={formatDistance}
  fromString={parseFloat}
/>
</div>

<div class="segment">
<TimeInput
name="time"
bind:value={
    () => time2seconds,
    (v) => time1seconds = deriveTime(v, distance2miles, distance1miles)
} withHours />
</div>

<div class="segment">
<TimeInput
name="pace miles"
bind:value={
    () => pace2miles,
    (v) => time1seconds = deriveTime(v * distance2miles, distance2miles, distance1miles)
} />

<TimeInput
name="pace km"
bind:value={
    () => pace2km,
    (v) => time1seconds = deriveTime(v * distance2km, distance2miles, distance1miles)
} />
</div>
</form>

<style>
form {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Open Sans', 'Helvetica Neue', sans-serif;
  font-size: 1.6rem;
  padding: 2rem;
}

.race1 {
  background-color: #f3f3f3;
}

.race2 {
  background-color: #333;
  color: #f3f3f3;
}

h2 {
  font-size: 2.5rem;
  margin-bottom: 1.6rem;
  border-bottom: 1px solid #bbb;
}

.segment {
  display: flex;
  gap: 0.8rem;
  margin: 0 0 0.5rem;
}
</style>
