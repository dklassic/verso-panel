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
  import TabBar, {
    type Tab,
    type TabActivateRequest,
    type TabCloseRequest,
    type TabCreateResponse,
    type TabId,
  } from './tabbar.svelte';

  let navbarEl: HTMLDivElement | null = null;
  let navbarUrl: string = '';

  // Window Dragging
  let startDragging: boolean = false;

  // System information
  const isMac = window.navigator.userAgent.includes('Mac');

  // Inject navbar function into global window object. Expose functions to backend.
  Object.assign(window, {
    navbar: {
      setNavbarUrl: (newUrl: string) => {
        const tab = tabs[activeTabIdx];
        tab.url = newUrl;
        navbarUrl = newUrl;
      },
      addTab: (id: string) => {
        tabs.push({ name: 'new tab', id: JSON.parse(id), url: '' });
      },
      setTabTitle: (id_raw: string, title: string) => {
        let id: TabId = JSON.parse(id_raw);
        let tab = tabs.find((tab) => {
          return (
            tab.id.namespace_id === id.namespace_id && tab.id.index === id.index
          );
        });
        if (tab) {
          tab.name = title;
          tabs = tabs; // trigger svelte to re-render
        }
      },
    },
  });

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

  /* tabs */

  let activeTabIdx = 0;
  let tabs: Tab[] = [];

  // TODO: get tab name / set index;
  function onClickNewTab() {
    console.log('click new tab');

    let resp: TabCreateResponse = JSON.parse(
      window.prompt('NEW_TAB') ?? '{ "success": false, id: null }'
    );
    if (!resp.success || !resp.id) {
      return;
    }

    tabs.push({ name: 'new tab', id: resp.id, url: '' });
    tabs = tabs; // trigger svelte to re-render

    activeTabIdx = Math.max(0, tabs.length - 1);
  }
  function onCloseTab(idx: CustomEvent<number>) {
    let removeTab = tabs[idx.detail];
    let request: TabCloseRequest = {
      id: removeTab.id,
    };
    window.prompt(`CLOSE_TAB:${JSON.stringify(request)}`);

    tabs = tabs.filter((tab, _) => tab.id !== removeTab.id);
    let newActiveTabIdx = activeTabIdx;
    if (idx.detail < activeTabIdx) {
      newActiveTabIdx--;
    } else if (idx.detail >= tabs.length - 1) {
      newActiveTabIdx = tabs.length - 1;
    }
    activateTab(newActiveTabIdx);
  }
  function onActivateTab(idx: CustomEvent<number>) {
    activateTab(idx.detail);
  }
  function activateTab(idx: number) {
    if (activeTabIdx === idx) {
      return;
    }
    if (idx >= tabs.length || idx < 0) {
      console.error(`Invalid tab index: ${idx}`);
      return;
    }

    const tab = tabs[idx];
    let request: TabActivateRequest = {
      id: tab.id,
    };

    activeTabIdx = idx;
    // update navbar url
    navbarUrl = tab.url;

    window.prompt(`ACTIVATE_TAB:${JSON.stringify(request)}`);
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
        bind:value={navbarUrl}
        on:keydown={(e) => e.code === 'Enter' && onEnterNavigation(navbarUrl)}
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
