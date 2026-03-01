<script lang="ts">
  let isFocused = $state();

	let {
    value = $bindable(),
    fromString,
    toString,
    name,
  } = $props();

  let textValue = $state(toString(value))

  $effect(() => {
    if (!isFocused) {
  		textValue = toString(value)
    }
	});
</script>

<span>
  <label for={name}>{name}</label>
  <input
    id={name}
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
</span>

<style>
  input {
    border: 1px solid #ccc;
    padding: 0rem 0.3rem;
    width: 10rem;
  }
  label {
    font-weight: bold;
    padding-right: 0.2rem;
  }
</style>
