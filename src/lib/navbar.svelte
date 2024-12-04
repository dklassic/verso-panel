<script lang="ts">
  import PrevPageIcon from '@images/arrow-left.png';
  import NextPageIcon from '@images/arrow-right.png';
  import CloseIcon from '@images/close.png';
  import MaximizeIcon from '@images/maximize.png';
  import MinimizeIcon from '@images/minimize.png';
  import NewWindowIcon from '@images/new-window.png';
  import RefreshIcon from '@images/refresh.png';
  import { Input } from 'flowbite-svelte';
  import { onMount } from 'svelte';
  import NavBtn from './btn.svelte';
  import TabBar from './tabbar.svelte';

  let navbarEl: HTMLDivElement | null = null;
  let url = '';

  // Window Dragging
  let startDragging: boolean = false;

  // System information
  const isMac = window.navigator.userAgent.includes('Mac');

  // Inject navbar function into global window object. Expose functions to backend.
  Object.assign(window, {
    navbar: {
      setNavbarUrl: (newUrl: string) => {
        url = newUrl;
      },
    },
  });

  // Tabs
  let activeTabIdx = 0;
  let tabs = [{ name: 'example' }];

  onMount(() => {
    if (isMac) {
      navbarEl?.classList.add('macos');
    }
  });

  /* Event handlers */

  function onClickPrev() {
    console.log('click prev');
    window.prompt('PREV');
  }
  function onClickForward() {
    console.log('click forward');
    window.prompt('FORWARD');
  }
  function onClickRefresh() {
    console.log('click refresh');
    window.prompt('REFRESH');
  }
  function onClickNewWindow() {
    console.log('click new window');
    window.prompt('NEW_WINDOW');
  }
  function onClickClose() {
    console.log('close');
    window.close();
  }
  function onClickMinimize() {
    console.log('minimize');
    window.prompt('MINIMIZE');
  }
  function onClickMaximize() {
    console.log('maximize');
    window.prompt('MAXIMIZE');
  }
  function onEnterNavigation(url: string) {
    console.log(`navigate to url: ${url}`);
    window.prompt(`NAVIGATE_TO:${url}`);
  }
  function onPanelMouseDown(ev: MouseEvent) {
    /* Draggable window */
    if (ev.buttons === 1) {
      // on macOS, this drag window must trigger before native drag happened, otherwise, it will crash.
      if (isMac) {
        console.log('dragging window');
        window.prompt('DRAG_WINDOW');
        return;
      }

      // on Linux, mouse release event will not fire after dragging window,
      // so we need to ensure it's acatually dragging window. Otherwise, double click on panel will not work.
      console.log('start dragging window');
      startDragging = true;
    }
  }
  function onPanelMouseUp(ev: MouseEvent) {
    /* Draggable window */
    // on Linux and Windows, if left mouse button is released and dragging is started, reset dragging state
    if ((ev.buttons & 1) === 0 && startDragging) {
      console.log('cancel dragging window');
      startDragging = false;
    }
  }
  function onMouseMove(ev: MouseEvent) {
    /* Draggable window */
    // on Linux and Windows, we use mouse move event to detect if the user is actually dragging window.
    if (!isMac && startDragging) {
      console.log('dragging window');
      window.prompt('DRAG_WINDOW');
      startDragging = false;
    }
  }
  function onPanelDbClick() {
    console.log('double click on panel');
    window.prompt('DBCLICK_PANEL');
  }

  function onClickNewTab() {
    console.log('click new tab');
    tabs.push({ name: 'new tab' });
    tabs = tabs;
    activeTabIdx = tabs.length-1;
    window.prompt('NEW_TAB');
    // TODO: get tab name;
  }
  function onCloseTab(tab: CustomEvent<number>) {
    tabs = tabs.filter((_, i) => i !== tab.detail);
    if (tab.detail < activeTabIdx) {
      activeTabIdx--;
    } else if (tab.detail >= tabs.length-1) {
      activeTabIdx = tabs.length-1;
    }
    // TODO: Get new index from here
    window.prompt(`CLOSE_TAB:${tab.detail}`);
  }
  function onActivateTab(idx: CustomEvent<number>) {
    if (activeTabIdx === idx.detail) {
      return;
    }
    activeTabIdx = idx.detail;
    window.prompt(`ACTIVATE_TAB:${idx.detail}`);
  }
</script>

<div>
  <div
    class="navbar flex box-border w-full items-center gap-1"
    bind:this={navbarEl}
    on:mousemove={onMouseMove}
    on:mousedown|self={onPanelMouseDown}
    on:mouseup|self={onPanelMouseUp}
    on:dblclick|self={onPanelDbClick}
  >
    <div
      class="flex flex-1 justify-between gap-1"
      on:mousedown|self={onPanelMouseDown}
      on:mouseup|self={onPanelMouseUp}
      on:dblclick|self={onPanelDbClick}
    >
      <div>
        <NavBtn on:click={onClickPrev} icon={PrevPageIcon} />
        <NavBtn on:click={onClickForward} icon={NextPageIcon} />
      </div>
      <div>
        <NavBtn on:click={onClickNewWindow} icon={NewWindowIcon} />
        <NavBtn on:click={onClickNewTab} icon={NewWindowIcon} />
      </div>
    </div>
    <div class="flex-2 w-1/2">
      <Input
        type="text"
        placeholder="Search or enter website name"
        bind:value={url}
        on:keydown={(e) => e.code === 'Enter' && onEnterNavigation(url)}
      />
    </div>
    <div
      class="flex flex-1 justify-between gap-1"
      on:mousedown|self={onPanelMouseDown}
      on:mouseup|self={onPanelMouseUp}
      on:dblclick|self={onPanelDbClick}
    >
      <div>
        <NavBtn on:click={onClickRefresh} icon={RefreshIcon} />
      </div>
      <div class="window-actions">
        <NavBtn on:click={onClickMinimize} icon={MinimizeIcon} />
        <NavBtn on:click={onClickMaximize} icon={MaximizeIcon} />
        <NavBtn on:click={onClickClose} icon={CloseIcon} />
      </div>
    </div>
  </div>
  {#if tabs.length > 1}
    <TabBar
      {tabs}
      activeIdx={activeTabIdx}
      on:close-tab={onCloseTab}
      on:activate-tab={onActivateTab}
    />
  {/if}
</div>

<style lang="scss">
  .navbar {
    font-family: Arial, sans-serif;
    height: 50px;
    padding-left: 10px;
    padding-right: 10px;
  }

  :global(.macos) {
    &.navbar {
      padding-left: 80px;
    }
    .window-actions {
      display: none;
    }
  }
</style>
