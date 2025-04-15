<script lang="ts">
  export let icon;
  export let alt = '';
  export let disabled = false;
</script>

<button
  on:click
  on:mousedown
  on:mouseup
  on:mouseleave
  {disabled}
  class="btn nav-btn bg-transparent hover:bg-zinc-100 active:bg-zinc-200 disabled:bg-transparent disabled:opacity-30 h-7 w-7 p-2 rounded-md flex items-center justify-center"
>
  <!-- pointer-events-none is used to prevent the img emitting mouse events, so the browsing history menu can show on the button's position -->
  <img draggable="false" {alt} src={icon} class="pointer-events-none" />
</button>

<style>
  .nav-btn {
    border: none;
    cursor: pointer;
  }
</style>
