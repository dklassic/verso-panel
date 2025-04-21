<script lang="ts" context="module">
  export interface Bookmark {
    name: string;
    url: string;
  }
</script>

<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let bookmarks: Bookmark[] = [];
  function navigateBookmark(idx: number) {
    dispatch('navigate-bookmark', idx);
  }

  // context menu state
  let longPressTimer: number | null = null;
  let isLongPressing = false;

  function onNavBtnMouseDown(ev: MouseEvent, index: number) {
    if (ev.button !== 0) return; // Only handle left click

    isLongPressing = true;

    longPressTimer = window.setTimeout(() => {
      if (isLongPressing) {
        const rect = (ev.target as HTMLElement).getBoundingClientRect();
        const request = {
          index,
          position: {
            x: rect.left,
            y: rect.bottom + 5,
          },
        };
        window.prompt(`OPEN_BOOKMARK_MENU:${JSON.stringify(request)}`);
      }
    }, 500);
  }


  function onNavBtnMouseUp() {
    isLongPressing = false;
    if (longPressTimer) {
      clearTimeout(longPressTimer);
      longPressTimer = null;
    }
  }

  function onNavBtnMouseLeave() {
    isLongPressing = false;
    if (longPressTimer) {
      clearTimeout(longPressTimer);
      longPressTimer = null;
    }
  }
</script>

<div id="bookmark-bar" class="bookmark-bar">
  {#each bookmarks as bookmark, index}
    <div
      class="bookmark bg-zinc-200 hover:bg-zinc-100 transition-[background-color] duration-100"
      on:click={() => {
        navigateBookmark(index);
      }}
      on:mousedown={(e) => onNavBtnMouseDown(e, index)}
      on:mouseup={onNavBtnMouseUp}
      on:mouseleave={onNavBtnMouseLeave}      
    >
      {bookmark.name}
    </div>
  {/each}
</div>

<style lang="scss">
  .bookmark-bar {
    display: flex;
    background-color: inherit;
    height: 30px;
    flex-wrap: nowrap;
    gap: 1px;
  }
  .bookmark {
    display: flex;
    background-color: inherit;
    height: 30px;
    padding: 2px 16px;
    max-width: 20ch;
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
    align-items: center;
    cursor: default;
    user-select: none;
  }
  .bookmark:hover{
    display: flex;
    background-color: #dfdfdf;
  }
</style>
