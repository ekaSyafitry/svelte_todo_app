<script>
  import { getDateRows, uuid, noop } from "./date_time.js";
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  // props
  export let date;
  export let month;
  export let year;
  export let isAllowed;

  // local vars to help in render
  const weekdays = ["Su", "Mo", "Tu", "We", "Th", "Fr", "Sa"];
  let cells;

  // function helpers
  const onChange = date => {
    dispatch("datechange", new Date(year, month, date));
  };

  const allow = (year, month, date) => {
    if (!date) return true;
    return isAllowed(new Date(year, month, date));
  };

  $: cells = getDateRows(month, year).map(c => ({
    value: c,
    allowed: allow(year, month, c)
  }));
</script>

<style>
  .container {
    margin-top: 8px;
    padding: 10px 2px;
    width: 242px;
    background: #fff;
  }
  .row {
    display: flex;
    margin: 2px 0px;
    flex-wrap: wrap;
  }

  .cell {
    color: #6D4C41;
    font-size: 12px;
    display: inline-block;
    width: 32px;
    height: 20px;
    text-align: center;
    padding: 2px;
    margin: 1px;
  }

  .selected {
    color: #fff;
    background: #EF5350;
  }

  .highlight {
    transition: transform 0.2s cubic-bezier(0.165, 0.84, 0.44, 1);
  }

  .disabled {
    /* background: #efefef; */
    cursor: not-allowed;
    color: #ccc;
  }

  .highlight:hover {
    background: rgb(238, 176, 60);
    color: #fff;
    cursor: pointer;
    transform: scale(1.3);
  }

  .selected.highlight:hover {
    background: #EF5350;
  }
</style>


<div class="container">
  <div class="row">
    {#each weekdays as day}
      <div class="cell">{day}</div>
    {/each}
  </div>

  <div class="row">
    {#each cells as { allowed, value } (uuid())}
      <div
        on:click={allowed && value ? onChange.bind(this, value) : noop}
        class:cell={true}
        class:highlight={allowed && value}
        class:disabled={!allowed}
        class:selected={new Date(date.getFullYear(), date.getMonth(), date.getDate()).getTime() === new Date(year, month, value).getTime()}>
        {value || ''}
      </div>
    {/each}
  </div>
</div>
