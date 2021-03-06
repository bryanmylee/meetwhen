<script lang="ts">
  import type { Interval, ShallowUser } from '$lib/gql/types';
  import { createPopperActions } from 'svelte-popperjs';
  import { classes } from '$lib/utils/classes';
  import { getContext } from 'svelte';
  import type { CalendarState } from './state/core';

  const state = getContext<CalendarState>('state');
  const { getComplimentUsers } = state;

  export let interval: Interval;
  export let users: ShallowUser[];
  $: complimentUsers = $getComplimentUsers(users);

  const INDEX_COL_WIDTH = 40;
  const HEADER_HEIGHT = 64;
  const [ref, content, getInstance] = createPopperActions({
    strategy: 'absolute',
    placement: 'right',
    modifiers: [
      { name: 'arrow', options: { padding: 16 } },
      { name: 'eventListeners', options: { scroll: false, resize: false } },
      { name: 'offset', options: { offset: [0, 12] } },
      {
        name: 'preventOverflow',
        options: {
          altAxis: true,
          padding: { top: HEADER_HEIGHT, right: 12, left: 12 + INDEX_COL_WIDTH },
        },
      },
    ],
  });

  export let show = false;
  export let fixed = false;

  export let referenceElement: HTMLElement;
  $: if (referenceElement !== undefined) {
    ref(referenceElement);
  }

  export const updateRefPosition = (event: MouseEvent) => {
    const parent = event.target as HTMLDivElement;
    const parentRect = parent.getBoundingClientRect();
    const top = Math.max(0, event.clientY - parentRect.top);
    referenceElement.style.position = 'absolute';
    referenceElement.style.left = '0';
    referenceElement.style.top = `${top}px`;
    updatePopoverPosition();
  };

  export const updatePopoverPosition = () => {
    getInstance()?.update();
  };

  $: popperClass = classes([
    'popover card pointer-events-none border-3 w-40',
    fixed ? 'border-primary dark:border-primary-lighter z-20' : 'border-gray-400 z-30',
  ]);

  $: popoverArrowClass = classes([
    'popover--arrow w-4 h-4 bg-default transform rotate-45',
    fixed ? 'border-primary dark:border-primary-lighter' : 'border-gray-400',
  ]);
</script>

{#if show}
  <div use:content class={popperClass}>
    <div data-popper-arrow>
      <div class={popoverArrowClass} />
    </div>
    <h1 class="px-4 py-3 text-xs text-gray-400">
      {interval.beg.format('h:mma')} – {interval.end.format('h:mma')}
    </h1>
    <div class="px-4 py-3 border-t border-gray-200 dark:border-gray-600">
      <h2 class="text-sm font-bold">
        {users.length} attending
      </h2>
      <ul class="mt-1 text-sm">
        {#each users as user}
          <li>{user.name}</li>
        {/each}
      </ul>
    </div>
    {#if complimentUsers.length > 0}
      <div class="px-4 py-3 text-gray-400 border-t border-gray-200 dark:border-gray-600">
        <h2 class="text-sm font-bold">
          {complimentUsers.length} not attending
        </h2>
        <ul class="mt-1 text-sm">
          {#each complimentUsers as user}
            <li>{user.name}</li>
          {/each}
        </ul>
      </div>
    {/if}
  </div>
{/if}

<style lang="postcss">
  :global(.popover[data-popper-placement^='right']) {
    & [data-popper-arrow] {
      @apply left-0;
    }
    & .popover--arrow {
      @apply border-l-3 border-b-3;
      --tw-translate-x: calc(-50% - 2px);
    }
  }

  :global(.popover[data-popper-placement^='left']) {
    & [data-popper-arrow] {
      @apply right-0;
    }
    & .popover--arrow {
      @apply border-r-3 border-t-3;
      --tw-translate-x: calc(50% + 2px);
    }
  }
</style>
