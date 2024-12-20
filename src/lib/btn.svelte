<script>
  import { Button } from 'flowbite-svelte';
  export let icon;
  export let alt = '';
</script>

<Button
  on:click
  class="btn nav-btn bg-transparent hover:bg-zinc-100 active:bg-zinc-200 h-7 w-7 !p-2"
>
  <img draggable="false" {alt} src={icon} />
</Button>
