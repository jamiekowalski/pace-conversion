<script lang="ts">
  const { name, children } = $props();
</script>

<div class="input-block">
  <div class="name">{name}</div>
  <div>{@render children()}</div>
</div>

<style>
  .input-block {
    margin: 0 0 0.6rem;
  }
  .name {
    font-weight: bold;
  }
</style>
