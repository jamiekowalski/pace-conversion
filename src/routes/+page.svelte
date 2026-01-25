<script lang="ts">
    import Input from "../lib/input.svelte"

    const KM_PER_MILE = 1.60934
    const HOUR_IN_SECONDS = 60 * 60
    const MINUTE_IN_SECONDS = 60

    function parseTime(time: string) {
        const parts_low_to_high = time.split(":").reverse()
        let current_factor = 1
        let total = 0
        for (let part of parts_low_to_high) {
            total += part * current_factor
            current_factor *= 60
        }
        return total
    }

    function formatSeconds(seconds: number) {
        const hours = Math.floor(seconds / HOUR_IN_SECONDS)
        let remaining_seconds = seconds - hours * HOUR_IN_SECONDS
        const minutes = Math.floor(remaining_seconds / MINUTE_IN_SECONDS)
        remaining_seconds = Math.round(remaining_seconds - minutes * MINUTE_IN_SECONDS)

        return [hours, minutes, remaining_seconds]
    }

    function formatSecondsString(seconds: number) {
        return formatSeconds(seconds).map(s => s.toString().padStart(2, "0")).join(":")
    }

    function formatDistance(distance: number) {
        return Math.round(distance * 100) / 100
    }

    function deriveTime(input_miles: number, output_miles: number, input_time: number) {
        return input_time * (output_miles / input_miles)**(1.06)
    }

    // function reverseDeriveTime(t1_seconds: number, d1_miles: number, d2_miles: number) {
    //     // given d1_miles, d2_miles, and t2_seconds
    //     return t1_seconds * (d2_miles / d1_miles)**(1.06)
    // }

    // only three values are given
    let distance1miles = $state(10)
    let time1seconds = $state(10)
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
<input bind:value={
    () => formatDistance(distance1miles),
    (v) => distance1miles = v
} />

km
<input bind:value={
    () => formatDistance(distance1km),
    (v) => distance1miles = v / KM_PER_MILE
} />

time
<input bind:value={
    () => formatSecondsString(time1seconds),
    (v) => time1seconds = parseTime(v)
} />

pace miles
<input bind:value={
    () => formatSecondsString(pace1miles),
    (v) => time1seconds = parseTime(v) * distance1miles
} />

pace km
<input bind:value={
    () => formatSecondsString(pace1km),
    (v) => time1seconds = parseTime(v) * distance1km
} />

<h2>Race 2</h2>

miles
<input bind:value={
    () => formatDistance(distance2miles),
    (v) => distance2miles = v
} />

km
<input bind:value={
    () => formatDistance(distance2km),
    (v) => distance2miles = v / KM_PER_MILE
} />

time
<input bind:value={
    () => formatSecondsString(time2seconds),
    (v) => time1seconds = deriveTime(parseTime(v), distance2miles, distance1miles)
} />

pace miles
<input bind:value={
    () => formatSecondsString(pace2miles),
    (v) => time1seconds = deriveTime(parseTime(v) * distance2miles, distance2miles, distance1miles)
} />

pace km
<input bind:value={
    () => formatSecondsString(pace2km),
    (v) => time1seconds = deriveTime(parseTime(v) * distance2km, distance2miles, distance1miles)
} />
