<script lang="ts" context="module">
  export interface SelectableProviderEvent {
    toggle: {
      id: string;
      selected: boolean;
    };
  }
</script>

<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { clearAllBodyScrollLocks, disableBodyScroll } from 'body-scroll-lock';
  import { getTouchArray } from '$lib/utils/touch';
  import LongTouchProvider from '$lib/components/LongTouchProvider.svelte';
  import { Selecting } from './selecting';
  import { idsWithAddedId, idsWithoutRemovedId } from './utils';

  const dispatch = createEventDispatcher<SelectableProviderEvent>();

  export let disabled = false;
  export let selectedIds: string[] = [];
  export let disabledIds: string[] = [];
  export let selecting = Selecting.NONE;
  export let isDisabled = (id: string) => disabledIds.includes(id);

  export const select = (id: string) => {
    if (isDisabled(id)) {
      return;
    }
    selectedIds = idsWithAddedId(selectedIds, id);
    dispatch('toggle', { id, selected: true });
  };

  export const deselect = (id: string) => {
    if (isDisabled(id)) {
      return;
    }
    selectedIds = idsWithoutRemovedId(selectedIds, id);
    dispatch('toggle', { id, selected: false });
  };

  export const toggle = (id: string) => {
    if (selectedIds.includes(id)) {
      deselect(id);
    } else {
      select(id);
    }
  };

  // STARTING SELECTIONS
  const startSelectionFrom = (target: HTMLElement) => {
    if (disabled) {
      return;
    }
    const { id } = target.dataset;
    if (id === undefined) {
      return;
    }
    if (selectedIds.includes(id)) {
      // TODO: provide interpolation.
      deselect(id);
      if (selecting === Selecting.NONE) {
        selecting = Selecting.DELETE;
      }
    } else {
      // TODO: provide interpolation.
      select(id);
      if (selecting === Selecting.NONE) {
        selecting = Selecting.CREATE;
      }
    }
  };

  const mousestart = ({ target }: MouseEvent) => {
    startSelectionFrom(target as HTMLElement);
  };

  let trackedTouches: Record<number, Touch> = {};

  const longtouchstart = ({ detail }: CustomEvent) => {
    const touchEvent = detail.event as TouchEvent;
    const changedTouches = getTouchArray(touchEvent.changedTouches);
    changedTouches.forEach((touch) => trackTouch(touch, touchEvent));
  };

  const trackTouch = (touch: Touch, { target }: TouchEvent) => {
    trackedTouches[touch.identifier] = touch;
    disableBodyScroll(target as HTMLElement);
    startSelectionFrom(target as HTMLElement);
  };

  // UPDATING SELECTIONS
  const updateSelectionOn = (target: HTMLElement) => {
    if (disabled) {
      return;
    }
    const { id } = target.dataset;
    if (id === undefined) {
      return;
    }
    if (selecting === Selecting.CREATE) {
      // TODO: provide interpolation.
      select(id);
    } else {
      // TODO: provide interpolation.
      deselect(id);
    }
  };

  const mousemoveinto = ({ target }: MouseEvent) => {
    if (selecting === Selecting.NONE) {
      return;
    }
    updateSelectionOn(target as HTMLElement);
  };

  const longtouchmove = ({ detail }: CustomEvent) => {
    const touchEvent = detail.event as TouchEvent;
    const changedTouches = getTouchArray(touchEvent.changedTouches);
    changedTouches.forEach(updateTouch);
  };

  const updateTouch = (touch: Touch) => {
    trackedTouches[touch.identifier] = touch;
    if (selecting === Selecting.NONE) {
      return;
    }
    const target = document.elementFromPoint(touch.clientX, touch.clientY);
    updateSelectionOn(target as HTMLElement);
  };

  // ENDING SELECTIONS
  const endSelection = () => {
    if (disabled) {
      return;
    }
    selecting = Selecting.NONE;
    trackedTouches = {};
  };

  const longtouchend = ({ detail }: CustomEvent) => {
    const touchEvent = detail.event as TouchEvent;
    const changedTouches = getTouchArray(touchEvent.changedTouches);
    changedTouches.forEach(untrack);
  };

  const untrack = (touch: Touch) => {
    delete trackedTouches[touch.identifier];
    clearAllBodyScrollLocks();
    endSelection();
  };
</script>

<LongTouchProvider
  on:longtouchstart={longtouchstart}
  on:longtouchmove={longtouchmove}
  on:longtouchend={longtouchend}
>
  <div
    on:mousedown={mousestart}
    on:mousemove={mousemoveinto}
    on:mouseleave={endSelection}
    on:mouseup={endSelection}
    class="contents"
  >
    <slot {selectedIds} {selecting} {isDisabled} />
  </div>
</LongTouchProvider>