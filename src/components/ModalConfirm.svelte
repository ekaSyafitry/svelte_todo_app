<script>
    import { onMount, afterUpdate } from 'svelte';
    import { createEventDispatcher } from 'svelte';

    export let type;
    export let modalActive;

    afterUpdate(() => {
	console.log("after update name:", type);
	})

    const dispatch = createEventDispatcher();

    const closeHandler = () => dispatch('close');
    const confirmComplete = () => dispatch('yesBtn');
</script>

<div>
    <div class="{modalActive? 'bg-modalComplete active' : 'bg-modalComplete'}" on:click={closeHandler} ></div>
    <div class="{modalActive? 'modalComplete active' : 'modalComplete'}">
        {#if type ==='complete'}
        <div>
            <div class="title" >Add to complete task?</div>
        </div>
        
        {:else if type==='incomplete'}
        <div>
            <div class="title"> Add to incomplete task?</div>
        </div>
    
        {:else if type==='trash'}
        <div >
            <div class="title"> Are you sure to delete data?</div>
        </div>
        {:else if type==='install'}
        <div >
          <div class="title"> Do you like this application?</div>
      </div>
        {/if}

        {#if type==='install'}
      <div style="display:flex; justify-content:space-between;">
        <button class="btnY" on:click={confirmComplete}>Download Now</button> 
        <button class="btnN" on:click={closeHandler}>Cancel</button>
      </div>
      {:else}
      <div>
        <button class="btnY" on:click={confirmComplete}>Yes</button> 
        <button class="btnN" on:click={closeHandler}>Cancel</button>
      </div>
      {/if}
    </div>
</div>

<style  type="text/scss">
  .bg-modalComplete {
    opacity: 0;
    visibility: hidden;
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    text-align: left;
    background: #000000ab;
    transition: opacity .25s ease;
  }

  .bg-modalComplete.active {
    opacity: 1;
    visibility: visible;
    transition: opacity .25s ease;
  }

  .modalComplete.active {
    opacity: 1;
    visibility: visible;
    top: 0;
    transition: top .25s ease;
  }

  %modal {
    visibility: hidden;
    background-color: #fff;
    transition: top .25s ease;
    position: absolute;
    top: 100%;
    right: 0;
    bottom: 0;
    left: 0;
    margin: auto;
    overflow: auto;
    border-radius: 5px;
    padding: 1em;
    opacity: 0;
    width: 93%;
  }

  %btn {
    background: linear-gradient(71deg, #fa5e6f 30%, #fb9561 70%);
    border-radius: 50%;
    border: none;
    box-shadow: 2px 6px 9px 2px #ef535061;
  }

  .modalComplete {
    @extend %modal;
    height: 130px;

    .title {
      margin-bottom: 30px;
      font-size: 18px;
      font-weight: 600;
      color: #EF5350;
    }

    .btnY {
      // @extend %btn;
      border: none;
      background: linear-gradient(71deg, #fa5e6f 30%, #fb9561 70%);
      @extend %btnConfirm;
      color: #fff;
      padding: 10px;
    }

    .btnN {
      border: 1px solid #EF5350;
      background-color: #fff;
      color: #EF5350;
      padding: 10px;
      margin-left: 20px;
      @extend %btnConfirm;
    }
  }

  %btnConfirm {
    border-radius: 10px;
    font-weight: 600;
    font-size: 14px;
  }

</style>