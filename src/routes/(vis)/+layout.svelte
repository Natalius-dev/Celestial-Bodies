<script>
    import "../../app.css";
    import Settings from "../../lib/Settings.svelte";
</script>

<div class="flex flex-col h-screen">
    <Settings />
    <slot />
</div>