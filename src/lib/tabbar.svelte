<script lang="ts" context="module">
  export interface Tab {
    name: string;
    id: TabId;
  }
  export interface TabCreateResponse {
    success: boolean;
    id: TabId | null;
  }
  export interface TabActivateRequest {
    id: TabId;
  }
  export interface TabCloseRequest {
    id: TabId;
  }
  export interface TabId {
    namespace_id: number;
    index: number;
  }
</script>

<script lang="ts">
  import CloseIcon from '@images/close.png';
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  export let tabs: Tab[] = [];
  export let activeIdx = 0;

  function activateTab(idx: number) {
    dispatch('activate-tab', idx);
  }
  function closeTab(idx: number) {
    dispatch('close-tab', idx);
  }
</script>

<div id="tab-bar" class="tab-bar">
  {#each tabs as tab, index}
    <div
      class="tab bg-zinc-200 hover:bg-zinc-100 transition-[background-color] duration-100"
      class:active={index === activeIdx}
      on:click={() => {
        activateTab(index);
      }}
    >
      <p class="title text-sm">{tab.name}</p>
      <button
        on:click|stopPropagation={() => {
          closeTab(index);
        }}
        class="close-btn hover:bg-zinc-200 active:bg-zinc-300 hover:transition-[background-color] duration-100"
      >
        <img src={CloseIcon} draggable="false" />
      </button>
    </div>
  {/each}
</div>

<style lang="scss">
  .tab-bar {
    display: flex;
    flex-wrap: nowrap;
    height: 30px;
    background-color: #dfdfdf;
  }
  .tab {
    display: flex;
    align-items: center;
    justify-content: center;
    flex: 1;
    height: 30px;
    user-select: none;
    cursor: default;
    &.active {
      background-color: #ffffff;
      .close-btn {
        visibility: visible;
      }
    }
    &:hover .close-btn {
      visibility: visible;
    }
  }
  .title {
    flex: 1;
    text-align: center;
  }
  .close-btn {
    visibility: hidden;
    margin-left: auto;
    margin-right: 8px;
    width: 20px;
    height: 20px;
    padding: 4px;
    border-radius: 4px;
  }
</style>
