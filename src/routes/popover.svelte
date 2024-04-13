<!-- Popover.svelte -->
<script>
  export let content;
  export let x;
  export let y;
</script>

<div class="popover" style="left: {x}px; top: {y}px;">
  {content}
</div>

<style>
  .popover {
    position: absolute;
    z-index: 9999;
    background-color: rgba(255, 255, 255, 0.8);
    padding: 4px;
    border: 1px solid black;
    border-radius: 4px;
    pointer-events: none; /* Ensures that the popover does not block mouse events */
  }
</style>
