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

<h1>Welcome to your library project</h1>
<p>Create your package using @sveltejs/package and preview/showcase your work with SvelteKit</p>
<p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p>

<h2>Race 1</h2>

miles
<InputNormalizeOnBlur
  bind:value={distance1miles}
  toString={formatDistance}
  fromString={parseFloat}
/>

km
<InputNormalizeOnBlur bind:value={
  () => distance1km,
  (v) => distance1miles = v / KM_PER_MILE
  }
  toString={formatDistance}
  fromString={parseFloat}
/>

time
<TimeInput bind:value={
    () => time1seconds,
    (v) => time1seconds = v
} withHours />

pace miles
<TimeInput bind:value={
    () => pace1miles,
    (v) => {
        console.log(v)
        time1seconds = v * distance1miles
    }
} />

pace km
<TimeInput bind:value={
    () => pace1km,
    (v) => time1seconds = v * distance1km
} />

<h2>Race 2</h2>

miles
<InputNormalizeOnBlur bind:value={distance2miles}
  toString={formatDistance}
  fromString={parseFloat}
/>

km
<InputNormalizeOnBlur bind:value={
    () => distance2km,
    (v) => distance2miles = v / KM_PER_MILE
  }
  toString={formatDistance}
  fromString={parseFloat}
/>

time
<TimeInput bind:value={
    () => time2seconds,
    (v) => time1seconds = deriveTime(v, distance2miles, distance1miles)
} withHours />

pace miles
<TimeInput bind:value={
    () => pace2miles,
    (v) => time1seconds = deriveTime(v * distance2miles, distance2miles, distance1miles)
} />

pace km
<TimeInput bind:value={
    () => pace2km,
    (v) => time1seconds = deriveTime(v * distance2km, distance2miles, distance1miles)
} />
