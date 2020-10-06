<script>
  import { createEventDispatcher } from "svelte";
  import Calender from "./Calender.svelte";
  import { getMonthName } from "./date_time.js";

  const dispatch = createEventDispatcher();

  // props
  export let isAllowed = () => true;
  export let selected = new Date();
  export let type = ""
  // console.log(type)

  // state
  let date, month, year, showDatePicker,dt;

  // so that these change with props
  $: {
    date = selected.getDate();
    month = selected.getMonth();
    year = selected.getFullYear();
  }

  // handlers
  const onFocus = () => {
    showDatePicker = true;
  };

  const next = () => {
    if (month === 11) {
      month = 0;
      year = year + 1;
      return;
    }
    month = month + 1;
  };

  const prev = () => {
    if (month === 0) {
      month = 11;
      year -= 1;
      return;
    }
    month -= 1;
  };

  const onDateChange = d => {
    showDatePicker = false;
    dispatch("datechange", d.detail);
    // console.log(d.detail)
    selected = d.detail
    getValDate()
  };

  const getValDate = () => {
    var months = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec",];
    dt = months[selected.getMonth()]+" "+selected.getDate()+" "+selected.getFullYear();
    // console.log(dt)
  }
 
 getValDate()
</script>

<style>
  .relative {
    position: relative;
  }

  input{
    font-size: 16px;
    display: inherit;
    margin-top: 10px;
    width: 100%;
    padding: 3px;
    border: 0;
    background: #d2691e17;
  }
  .box {
    background:#fff;
    position: absolute;
    z-index: 99;
    top: 40px;
    right: 0px;
    border: 1px solid #ccc;
    display: inline-block;
  }

  .box-mod{
    background: #fff;
    position: absolute;
    
    /* top: 40px; */
    left: 0px;
    border: 1px solid #ccc;
    display: inline-block;
    bottom: 0;
  }

  .month-name {
    /* background: #fff; */
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin: 6px 0;
  }

  .center {
    color: #6D4C41;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .center i{
    font-size: 18px;
  }

  .btn_cal i{
    font-size: 25px;
    /* margin-bottom: 30px; */
  }
</style>

<div class="relative">
  {#if type=='Home'}
  <div on:click={onFocus} class="btn_cal"><i class="far fa-calendar-alt"></i></div>
  {:else}
  <input type="text" on:focus={onFocus} value={dt} />
  {/if}
  {#if showDatePicker}
    <div class="{type=='Home'? 'box' : 'box-mod'}">
      <div class="month-name">
        <div class="center">
          <div on:click={prev}><i class="fas fa-angle-left"></i></div>
        </div>
        <div class="center">{getMonthName(month)} {year}</div>
        <div class="center">
          <div on:click={next}><i class="fas fa-angle-right"></i></div>
        </div>
      </div>
      <Calender
        {month}
        {year}
        date={selected}
        {isAllowed}
        on:datechange={onDateChange} />
    </div>
  {/if}
</div>
