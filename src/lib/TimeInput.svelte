<script lang="ts">
	import InputNormalizeOnBlur from "./InputNormalizeOnBlur.svelte";

  const HOUR_IN_SECONDS = 60 * 60
  const MINUTE_IN_SECONDS = 60

	let {
    // number of seconds
    value = $bindable(),
    withHours = false
  } = $props();

  function splitSeconds(seconds: number) {
    // seconds to [hours, minutes, seconds]
    const hours = Math.floor(seconds / HOUR_IN_SECONDS)
    let remaining_seconds = seconds - hours * HOUR_IN_SECONDS
    const minutes = Math.floor(remaining_seconds / MINUTE_IN_SECONDS)
    remaining_seconds = Math.round(remaining_seconds - minutes * MINUTE_IN_SECONDS)

    return [hours, minutes, remaining_seconds]
  }

  function secondsToString(seconds: number) {
    return splitSeconds(seconds).map(s => `${s}`.padStart(2, "0")).join(":")
  }

  function parseTime(time: string) {
    const expectedLength = withHours ? 3 : 2
    let parts_low_to_high = time.split(":").reverse().slice(0, 3)
    if (parts_low_to_high.length < expectedLength) {
      parts_low_to_high = ["0"].concat(parts_low_to_high)
    }
    let current_factor = 1
    let total = 0
    for (let part of parts_low_to_high) {
        total += (parseInt(part, 10) || 0) * current_factor
        current_factor *= 60
    }
    return total
  }
</script>

<InputNormalizeOnBlur
	bind:value={value}
  fromString={parseTime}
  toString={secondsToString}
/>
