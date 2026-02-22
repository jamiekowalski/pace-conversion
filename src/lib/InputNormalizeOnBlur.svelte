<script lang="ts">
  let isFocused = $state();

	let {
    value = $bindable(),
    fromString,
    toString,
  } = $props();

  let textValue = $state(toString(value))

  $effect(() => {
    if (!isFocused) {
  		textValue = toString(value)
    }
	});
</script>

<input
	bind:value={
    () => textValue,
    (v) => {
      textValue = v
      value = fromString(textValue)
    }
  }
	onfocus={() => { isFocused = true }}
	onblur={() => { isFocused = false }}
/>
