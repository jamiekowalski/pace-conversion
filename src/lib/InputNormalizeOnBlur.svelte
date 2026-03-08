<script lang="ts">
  let isFocused = $state();

  let {
    value = $bindable(),
    fromString,
    toString,
    name,
    disabled = false,
    unit = null,
    width = '10rem',
  } = $props();

  let textValue = $state(toString(value));

  $effect(() => {
    if (!isFocused) {
      textValue = toString(value);
    }
  });
</script>

<span class="container" style:width>
  <input
    inputmode="decimal"
    id={name}
    bind:value={
      () => textValue,
      (v) => {
        textValue = v;
        value = fromString(textValue);
      }
    }
    onfocus={() => {
      isFocused = true;
    }}
    onblur={() => {
      isFocused = false;
    }}
    {disabled}
    style:width
  />
  {#if unit}
    <span class="unit">{unit}</span>
  {/if}
</span>

<style>
  .container {
    position: relative;
    display: inline-block;
    margin-right: 0.6rem;
  }

  input {
    border: 1px solid #ccc;
    border-radius: 0.6rem;
    padding-left: 0.3rem;
  }

  input:disabled {
    border-color: #484848;
  }

  .unit {
    position: absolute;
    right: 0.4rem;
    bottom: 0.3rem;
    color: #777;
    font-size: 0.8em;
  }
</style>
