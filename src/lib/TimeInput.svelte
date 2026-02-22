<script lang="ts">
  const HOUR_IN_SECONDS = 60 * 60
  const MINUTE_IN_SECONDS = 60

  let isFocused = $state();

	let {
    // number of seconds
    value = $bindable(),
    withHours = false
  } = $props();

  let textValue = $state(formatSecondsToString(value))

  $effect(() => {
    if (!isFocused) {
  		textValue = formatSecondsToString(value)
    }
	});

  function formatString(t: string) {
    let expectedLength = withHours ? 3 : 2
    let segments = t.split(":").map(s => s.padStart(2, "0"))
    if (segments.length == expectedLength - 2) {
      segments = ["00"].concat(segments).concat(["00"])
    } else if (segments.length == expectedLength - 1) {
      segments = ["00"].concat(segments)
    } else if (segments.length > expectedLength) {
      segments = segments.slice(-expectedLength)
    }
    return segments.join(":")
  }

  function formatSeconds(seconds: number) {
      const hours = Math.floor(seconds / HOUR_IN_SECONDS)
      let remaining_seconds = seconds - hours * HOUR_IN_SECONDS
      const minutes = Math.floor(remaining_seconds / MINUTE_IN_SECONDS)
      remaining_seconds = Math.round(remaining_seconds - minutes * MINUTE_IN_SECONDS)

      return [hours, minutes, remaining_seconds]
  }

  function formatSecondsToString(seconds: number) {
    return formatString(formatSeconds(seconds).join(":"))
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

<input
	bind:value={
    () => textValue,
    (v) => {
      textValue = v
      value = parseTime(textValue)
    }
  }
	onfocus={() => { isFocused = true }}
	onblur={() => { isFocused = false }}
/>
