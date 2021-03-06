<script>
  import { createEventDispatcher, onMount } from "svelte";
  import scroll from "./scroll.js";
  import utils from "./utils.js";
  import BodyRow from "./BodyRow.svelte";
  import SideMenu from "./SideMenu.svelte";
  import HeaderGroup from "./HeaderGroup.svelte";
  import HeaderRow from "./HeaderRow.svelte";

  const today = new Date();

  export let endTime;
  export let formatHeader = (item, zoom) => {
    const formatters = {
      year: () => {
        const content = `${item.startDate.getFullYear()}`;
        return {
          ...item,
          content: content,
          title: content,
        };
      },
      month: () => {
        const content = `${item.startDate.getFullYear()}`;
        return {
          ...item,
          content: content,
          title: content,
        };
      },
      day: () => {
        const content = `${
          item.startDate.getMonth() + 1
        }/${item.startDate.getFullYear()}`;
        return {
          ...item,
          content: content,
          title: content,
        };
      },
      hour: () => {
        const content = `${
          item.startDate.getMonth() + 1
        }/${item.startDate.getDate()}/${item.startDate.getFullYear()}`;
        return {
          ...item,
          content: content,
          title: content,
        };
      },
    };
    return formatters[zoom]();
  };
  export let formatSlice = (slice, zoom) => {
    const day = slice.startDate.getDay();
    const isWeekend = day === 6 || day === 0;
    const todayTime = today.getTime();
    const isToday = todayTime >= slice.startTime && todayTime <= slice.endTime;

    const classes = [];

    if (isWeekend) {
      classes.push("weekend");
    }
    if (isToday) {
      classes.push("today");
    }

    const formatter = {
      year: function () {
        const content = slice.startDate.getFullYear();
        return {
          ...slice,
          header: {
            ...slice.header,
            content: content,
            title: content,
          },
          body: {
            ...slice.body,
            title: content,
            class: classes.join(" "),
          },
        };
      },
      month: function () {
        const content = (slice.startDate.getMonth() + 1)
          .toString()
          .padStart(2, "0");
        return {
          ...slice,
          header: {
            ...slice.header,
            content: content,
            title: content,
          },
          body: {
            ...slice.body,
            title: content,
            class: classes.join(" "),
          },
        };
      },
      day: function () {
        const content = slice.startDate.getDate().toString().padStart(2, "0");
        return {
          ...slice,
          header: {
            ...slice.header,
            content: content,
            title: content,
          },
          body: {
            ...slice.body,
            title: content,
            class: classes.join(" "),
          },
        };
      },
      hour: function () {
        const content = slice.startDate.getHours().toString().padStart(2, "0");
        return {
          ...slice,
          header: {
            ...slice.header,
            content: content,
            title: content,
          },
          body: {
            ...slice.body,
            title: content,
            class: classes.join(" "),
          },
        };
      },
    };
    return formatter[zoom]();
  };
  export let getHeader = {
    year: (slices) => {
      const groups = utils.groupBy(slices, (slice) =>
        slice.startDate.getFullYear()
      );
      const items = [];

      for (let i in groups) {
        const first = groups[i][0];
        const last = groups[i][groups[i].length - 1];
        const item = {
          startDate: first.startDate,
          startTime: first.startTime,
          endDate: last.startDate,
          endTime: last.startTime,
        };
        items.push(formatHeader(item, zoom));
      }
      return items;
    },
    month: (slices) => {
      const groups = utils.groupBy(slices, (slice) =>
        slice.startDate.getFullYear()
      );
      const items = [];

      for (let i in groups) {
        const first = groups[i][0];
        const last = groups[i][groups[i].length - 1];
        const item = {
          startDate: first.startDate,
          startTime: first.startTime,
          endDate: last.startDate,
          endTime: last.startTime,
        };
        items.push(formatHeader(item, zoom));
      }
      return items;
    },
    day: (slices) => {
      const groups = utils.groupBy(
        slices,
        (slice) =>
          `${slice.startDate.getFullYear()}-${slice.startDate.getMonth()}`
      );
      const items = [];

      for (let i in groups) {
        const first = groups[i][0];
        const last = groups[i][groups[i].length - 1];
        const item = {
          startDate: first.startDate,
          startTime: first.startTime,
          endDate: last.startDate,
          endTime: last.startTime,
        };
        items.push(formatHeader(item, zoom));
      }
      return items;
    },
    hour: (slices) => {
      const groups = utils.groupBy(slices, (slice) =>
        slice.startDate.getDate()
      );
      const items = [];

      for (let i in groups) {
        const first = groups[i][0];
        const last = groups[i][groups[i].length - 1];
        const item = {
          startDate: first.startDate,
          startTime: first.startTime,
          endDate: last.startDate,
          endTime: last.startTime,
        };
        items.push(formatHeader(item, zoom));
      }
      return items;
    },
  };
  export let getSlices = {
    year: (startTime, endTime) => {
      const _slices = [];
      let date = new Date(startTime.getFullYear(), 0, 1);
      const end = new Date(endTime.getFullYear(), 11, 31);

      while (date <= end) {
        const current = utils.addYears(date, 1);
        const _endDate = new Date(current.getFullYear(), 12, 31);
        _endDate.setHours(23, 59, 59, 999);

        const _slice = getSlice(date, _endDate);
        _slices.push(formatSlice(_slice, zoom));
        date = current;
      }
      return _slices;
    },
    month: (startTime, endTime) => {
      const _slices = [];
      let date = new Date(startTime);
      date.setDate(1);
      date.setHours(0, 0, 0, 0);
      const end = new Date(endTime);
      end.setDate(0);
      end.setHours(23, 59, 59, 999);

      while (date <= end) {
        const current = utils.addMonths(date, 1);
        const _endDate = utils.getLastDayOfMonth(date);
        _endDate.setHours(23, 59, 59, 999);

        const _slice = getSlice(date, _endDate);
        _slices.push(formatSlice(_slice, zoom));
        date = current;
      }
      return _slices;
    },
    day: (startTime, endTime) => {
      const _slices = [];
      let date = new Date(startTime);
      date.setHours(0, 0, 0, 0);
      const end = new Date(endTime);
      end.setHours(23, 59, 59, 999);

      while (date <= end) {
        const current = utils.addDays(date, 1);
        const _startTime = date.getTime();
        const _endDate = new Date(_startTime);
        _endDate.setHours(23, 59, 59, 999);

        const _slice = getSlice(date, _endDate);
        _slices.push(formatSlice(_slice, zoom));
        date = current;
      }
      return _slices;
    },
    hour: (startTime, endTime) => {
      const _slices = [];
      let date = new Date(startTime);
      date.setMinutes(0, 0, 0);
      const end = new Date(endTime);
      end.setMinutes(59, 59, 999);

      while (date <= end) {
        const current = utils.addHours(date, 1);
        const _startTime = date.getTime();
        const _endDate = new Date(_startTime);
        _endDate.setMinutes(59, 59, 999);

        const _slice = getSlice(date, _endDate);
        _slices.push(formatSlice(_slice, zoom));
        date = current;
      }
      return _slices;
    },
  };
  export let headers = [{ content: "" }];
  export let rows;
  export let scrollBehavior;
  export let scrollMoveIntervalTimeout = 50;
  export let slider;
  export let slices;
  export let startTime;
  export let zoom;

  const dispatch = createEventDispatcher();
  let headerGroupLoaded = false;
  let resizeCount = 0;
  let rowsContainer;

  $: updateSlices(slices, resizeCount, zoom);

  onMount(() => {
    if (!slider) {
      slider = window;
    }

    if (slider !== window) {
      slider.classList.add("svelte-gantt-slider");
    }
  });

  function getSlice(startDate, endDate) {
    return {
      startDate: startDate,
      startTime: startDate.getTime(),
      endDate: endDate,
      endTime: endDate.getTime(),
      body: {},
      header: {},
    };
  }

  function loadHeaderGroup() {
    headerGroupLoaded = true;
    return "";
  }

  function onClick(e) {
    dispatch("click", e.detail);
  }

  function onResize(e) {
    resizeCount++;
  }

  function updateSlices(_slices, resizeCount, zoom) {
    slices = getSlices[zoom](new Date(startTime), new Date(endTime));
  }

  //index = 0 equals to Header
  export function getCoordinates(index, startTime, endTime) {
    const startTimeRelative = getRelativeDate(startTime);
    const endTimeRelative = getRelativeDate(endTime);

    const startCell = rowsContainer.querySelector(
      `.slice[index="${index}"][starttime="${startTimeRelative}"]`
    );
    const endCell = rowsContainer.querySelector(
      `.slice[index="${index}"][starttime="${endTimeRelative}"]`
    );

    if (startCell && endCell) {
      const startCoords = utils.offset(startCell);
      const endCoords = utils.offset(endCell);
      const top = startCoords.top;
      const left = startCoords.left;

      return {
        startCell,
        endCell,
        top,
        left,
        height: startCoords.bottom - top,
        width: endCoords.right - left,
      };
    }
    return undefined;
  }

  export function getRelativeDate(date, _zoom) {
    if (!_zoom) {
      _zoom = zoom;
    }

    const result = {
      year: (date) => {
        let newDate = new Date(date);
        newDate = new Date(newDate.getFullYear(), 0, 1);
        return newDate.getTime();
      },
      month: (date) => {
        let newDate = new Date(date);
        newDate = new Date(newDate.getFullYear(), newDate.getMonth(), 1);
        return newDate.getTime();
      },
      day: (date) => {
        let newDate = new Date(date);
        newDate.setHours(0, 0, 0, 0);
        return newDate.getTime();
      },
      hour: (date) => {
        let newDate = new Date(date);
        return newDate.getTime();
      },
    };

    return result[_zoom](date);
  }

  export function goto(date, behavior) {
    const relativeDate = getRelativeDate(date);
    const coords = getCoordinates(0, relativeDate, relativeDate);
    if (coords) {
      scrollTo({
        left: coords.left - (slider.offsetWidth || slider.innerWidth) / 2,
        top: slider.offsetTop || slider.scrollY,
        behavior,
      });
    } else if (date < slices[0].startTime) {
      scrollTo({
        left: 0,
        top: slider.offsetTop,
        behavior,
      });
    } else if (date > slices[slices.length - 1].startTime) {
      scrollTo({
        left: slices[slices.length - 1].startTime,
        top: slider.offsetTop,
        behavior,
      });
    }
  }

  let scrollToTimeout;

  function scrollTo(coords) {
    clearTimeout(scrollToTimeout);
    scrollToTimeout = setTimeout(function () {
      slider.scrollTo(coords);
    }, 100); //prevent scrollTo method being called twice on first time (js bug)
  }

  function onLoaded() {
    setTimeout(() => {
      dispatch("load", { rows, slices });
    }, 1); //needs timeout otherwise it won't render right in case of goto method is called during this event
  }
</script>

<style>
  .header-side {
    height: 5em;
    top: 0;
  }

  .header-side-group {
    background-color: #f9fafb;
    flex: none;
    height: 100%;
    left: 0;
    z-index: 2;
  }

  .header-side-group :global(.slice) {
    float: left;
    width: 100px;
    height: 100%;
  }

  .header-slices {
    display: flex;
    position: sticky;
    height: 2.5em;
    top: 2.5em;
  }

  .header-top {
    display: flex;
    position: sticky;
    height: 2.5em;
    top: 0;
    z-index: 1;
  }

  .header-top-group {
    cursor: ew-resize;
    top: 0;
    background-color: #f9fafb;
    z-index: 1;
  }

  .noselect {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }

  .sticky {
    position: sticky;
  }

  .svelte-gantt {
    font-size: 14px;
    position: relative;
    display: flex;
    width: max-content;
    min-width: 100%;
  }

  .svelte-gantt :global(.align-center) {
    display: flex;
    align-items: center;
  }

  .svelte-gantt :global(.content) {
    padding: 0.4em;
  }

  .svelte-gantt :global(.flex) {
    flex: 1;
  }

  .svelte-gantt :global(.relative) {
    position: relative;
  }

  .svelte-gantt :global(.row-height) {
    height: 3em;
  }

  .svelte-gantt :global(.slice) {
    overflow: hidden;
    border-right: 1px solid #eee;
    border-top: 1px solid #eee;
  }

  .svelte-gantt :global(.weekend) {
    background-color: #f9fafb;
  }

  .svelte-gantt :global(.today) {
    background-color: #f1c40f1a;
  }

  :global(.svelte-gantt-slider) {
    position: relative;
    overflow: auto;
  }
</style>

<svelte:window on:resize={onResize} />

<div class="svelte-gantt">
  <div class="header-side-group sticky">
    <div class="header-side align-center">
      {#each headers as header}
        <div
          on:click={(e) => onClick({
              e,
              detail: { header, type: 'menuHeader' },
            })}
          class:align-center={true}
          class:slice={true}
          {...header}>
          <span class="content">
            {@html header.content}
          </span>
        </div>
      {/each}
    </div>
    {#each rows as row, index (row)}
      <SideMenu headers={row.headers} bind:row on:click={onClick} />
    {/each}
  </div>
  <div class="flex noselect">
    <div
      class="header-top-group sticky"
      use:scroll={{ slider, directions: ['x'], behavior: scrollBehavior, moveIntervalTimeout: scrollMoveIntervalTimeout }}>
      <div class="header-top">
        {#if headerGroupLoaded}
          <HeaderGroup
            {getCoordinates}
            {getHeader}
            {slices}
            {zoom}
            on:click={onClick} />
          <img
            style="display:none;"
            src="svelte-gantt:onload"
            alt="svelte-gantt:onload"
            on:error={onLoaded} />
        {/if}
      </div>
      <div class="header-slices">
        <HeaderRow {slices} on:click={onClick} />
      </div>
    </div>
    <div
      bind:this={rowsContainer}
      class="rows"
      use:scroll={{ slider, directions: ['y'], behavior: scrollBehavior, moveIntervalTimeout: scrollMoveIntervalTimeout }}>
      {#if rowsContainer}
        {#each rows as row, index (row)}
          <BodyRow
            {index}
            bind:row
            {slices}
            {getCoordinates}
            {getRelativeDate}
            on:click={onClick} />
        {/each}
        {loadHeaderGroup()}
      {/if}
    </div>
  </div>
</div>
