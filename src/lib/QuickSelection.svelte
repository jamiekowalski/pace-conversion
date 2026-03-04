<script lang="ts">
    const MARATHON_KM = 42.195
    const HALF_KM = 21.0975
    const TEN_KM = 10
    const FIVE_KM = 5

    const {
        setKm,
    } = $props();
</script>

<div class="container">
    <div class="button" onclick={() => setKm(FIVE_KM)}>5k</div>
    <div class="button" onclick={() => setKm(TEN_KM)}>10k</div>
    <div class="button" onclick={() => setKm(HALF_KM)}>half</div>
    <div class="button" onclick={() => setKm(MARATHON_KM)}>full</div>
</div>

<style>
    .container {
        display: grid;
        width: 8rem;
        grid-template-columns: repeat(2, 1fr);
        gap: 0.8rem;
        float: right;
        margin-top: 1rem;
    }
    .button {
        border: 1px solid #aaa;
        border-radius: 0.6rem;
        padding: 0.3rem;
        cursor: pointer;
        text-align: center;
    }
</style>
