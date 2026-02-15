<script lang="ts">
    let node
    let { val = $bindable() } = $props();
</script>

<input bind:this={node} bind:value={
    () => val,
    (v) => val = v.toLowerCase()
} />
