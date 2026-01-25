<script lang="ts">
    let { val = $bindable() } = $props();
</script>

<input bind:value={
    () => val,
    (v) => val = v.toLowerCase()
} />
