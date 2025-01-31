<script lang="ts">
  import { faBookmark as farBookmark } from '@fortawesome/free-regular-svg-icons';
  import {
    faBookmark as fasBookmark,
    faCrosshairs,
    faExpand,
    faList
  } from '@fortawesome/free-solid-svg-icons';
  import { mergeEntries } from '$lib/components/merged-header-icon/merged-entries';
  import MergedHeaderIcon from '$lib/components/merged-header-icon/merged-header-icon.svelte';
  import Popover from '$lib/components/popover/popover.svelte';
  import {
    baseHeaderClasses,
    baseIconClasses,
    nTranslateXHeaderFa,
    translateXHeaderFa
  } from '$lib/css-classes';
  import { customReadingPointEnabled$, viewMode$ } from '$lib/data/store';
  import { ViewMode } from '$lib/data/view-mode';
  import { dummyFn, isMobile$ } from '$lib/functions/utils';
  import { createEventDispatcher } from 'svelte';
  import Fa from 'svelte-fa';

  export let hasChapterData: boolean;
  export let autoScrollMultiplier: number;
  export let hasCustomReadingPoint: boolean;
  export let showFullscreenButton: boolean;
  export let isBookmarkScreen: boolean;

  const dispatch = createEventDispatcher<{
    tocClick: void;
    bookmarkClick: void;
    showCustomReadingPoint: void;
    setCustomReadingPoint: void;
    resetCustomReadingPoint: void;
    fullscreenClick: void;
    bookManagerClick: void;
    settingsClick: void;
  }>();

  const customReadingPointMenuItems: {
    label: string;
    action: any;
  }[] = [
    ...(hasCustomReadingPoint ? [{ label: 'Show Point', action: 'showCustomReadingPoint' }] : []),
    { label: 'Set Point', action: 'setCustomReadingPoint' },
    ...(hasCustomReadingPoint ? [{ label: 'Reset Point', action: 'resetCustomReadingPoint' }] : [])
  ];

  let customReadingPointMenuElm: Popover;

  function dispatchCustomReadingPointAction(action: any) {
    dispatch(action);
    customReadingPointMenuElm.toggleOpen();
  }
</script>

<div class="flex justify-between bg-gray-700 px-4 md:px-8 {baseHeaderClasses}">
  <div class="flex transform-gpu {nTranslateXHeaderFa}">
    {#if hasChapterData}
      <div
        role="button"
        class={baseIconClasses}
        on:click={() => dispatch('tocClick')}
        on:keyup={dummyFn}
      >
        <Fa icon={faList} />
      </div>
    {/if}
    <div
      role="button"
      class={baseIconClasses}
      on:click={() => dispatch('bookmarkClick')}
      on:keyup={dummyFn}
    >
      <Fa icon={isBookmarkScreen ? fasBookmark : farBookmark} />
    </div>
    {#if !$isMobile$}
      <div class="flex items-center px-4 text-xl xl:px-3 xl:text-lg">{autoScrollMultiplier}x</div>
    {/if}
  </div>

  <div class="flex transform-gpu {translateXHeaderFa}">
    {#if $customReadingPointEnabled$ || $viewMode$ === ViewMode.Paginated}
      <div class="flex">
        <Popover
          placement="bottom"
          fallbackPlacements={['bottom-end', 'bottom-start']}
          yOffset={0}
          bind:this={customReadingPointMenuElm}
        >
          <div slot="icon" class={baseIconClasses}>
            <Fa icon={faCrosshairs} />
          </div>
          <div class="w-40 bg-gray-700 md:w-32" slot="content">
            {#each customReadingPointMenuItems as actionItem (actionItem.label)}
              <div
                role="button"
                class="px-4 py-2 text-sm hover:bg-white hover:text-gray-700"
                on:click={() => dispatchCustomReadingPointAction(actionItem.action)}
                on:keyup={dummyFn}
              >
                {actionItem.label}
              </div>
            {/each}
          </div>
        </Popover>
      </div>
    {/if}
    {#if showFullscreenButton}
      <div
        role="button"
        class={baseIconClasses}
        on:click={() => dispatch('fullscreenClick')}
        on:keyup={dummyFn}
      >
        <Fa icon={faExpand} />
      </div>
    {/if}
    <MergedHeaderIcon
      items={[mergeEntries.SETTINGS, mergeEntries.MANAGE]}
      on:action={({ detail }) => {
        if (detail === mergeEntries.SETTINGS.label) {
          dispatch('settingsClick');
        } else if (detail === mergeEntries.MANAGE.label) {
          dispatch('bookManagerClick');
        }
      }}
    />
  </div>
</div>
