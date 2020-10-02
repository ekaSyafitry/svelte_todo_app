<script>
    import { onMount, afterUpdate } from 'svelte';
    import { createEventDispatcher } from 'svelte';
    import DatePicker from "./DatePicker.svelte";  

    //props
    export let modalActive;
    export let type;
    export let data;
    export let tab;
   
    //state
    const dispatch = createEventDispatcher();
    const database = firebase.database();
    let currentDate = new Date();
    const onDateChange = d => {
        currentDate = d.detail;
    };
    const onDateModalChange = d => {
      data.date = d.detail
    }
    let name = '',
        notes = '';


    afterUpdate(() => {
		// console.log("after update name:", modalActive);
    // console.log(data)
	  })

    const closeHandler = () => dispatch('close');
    const handleSubmit = () => {
        if(type=='add'){
            console.log(name)
            let frmt_date = currentDate.getFullYear() + "-" + (currentDate.getMonth() + 1) + "-" + currentDate.getDate()
            const rootRef = database.ref('todolist')
            const autoId = rootRef.push().key
            rootRef.child(autoId).set({
            name: name,
            notes: notes,
            date: frmt_date,
            complete: false
            })
            alert('Success add data !!')
            name = ''
            notes = '',
            currentDate = new Date()
            closeHandler()
            console.log(tab, 'tabbbbbbbbbb')
        }
        else if(type=='edit'){
          let frmt_date = data.date.getFullYear() + "-" + (data.date.getMonth() + 1) + "-" + data.date.getDate()
          database.ref('todolist').child(data.id).update({
            name: data.name,
            notes: data.notes,
            date: frmt_date
          });
          alert('Success edit data !!')
          closeHandler()
        }
    }
   
</script>

<div>
    <div class="{modalActive ? 'bg-modal active' : 'bg-modal'}" on:click={closeHandler}></div>
    <form action="" on:submit|preventDefault={handleSubmit} class="{modalActive? 'add-modal active' : 'add-modal'}">
        <div style="height:100%; position:relative;">
        {#if type=='edit'}
        {#if data !== undefined}
        <h2 > Edit Task</h2>
        <div>
            <div>
                <label for="task" class="title-mdl"><i class="fas fa-tasks"></i>TASK</label>
                <input type="text" name="task" bind:value={data.name} required>  
            </div>
            <div>
                <label for="note" class="title-mdl"><i class="far fa-comment"></i>NOTES</label>
                <textarea name="note" id="note" cols="30" rows="5" width="100%" bind:value={data.notes}></textarea>
            </div>
            <div>
                <label for="time" class="title-mdl"> <i class="far fa-calendar-alt"></i>DATE</label>
                <DatePicker
                type="modal"
                on:datechange={onDateModalChange}
                selected={data.date}
                isAllowed={date => {
                    const millisecs = date.getTime();
                    if (millisecs + 25 * 3600 * 1000 < Date.now()) return false;
                   // if (millisecs > Date.now() + 3600 * 24 * 45 * 1000) return false;
                    return true;
                }} />
          </div> 
            </div>
        {/if}
        {/if}
     
      <div>
        {#if type=='add'}
        <h2 >Add Task</h2>
        <div>
          <label for="task" class="title-mdl"><i class="fas fa-tasks"></i>TASK</label>
          <input type="text" name="task" bind:value={name}  required>
        </div>
        <div>
          <label for="note" class="title-mdl"><i class="far fa-comment"></i>NOTES</label>
          <textarea name="note" id="note" cols="30" rows="5" width="100%"  bind:value={notes}></textarea>
        </div>
        <div>
          <label for="time" class="title-mdl"> <i class="far fa-calendar-alt"></i>DATE</label>
          <DatePicker
            type="modal"
            on:datechange={onDateChange}
            selected={currentDate}
            isAllowed={date => {
                const millisecs = date.getTime();
                if (millisecs + 25 * 3600 * 1000 < Date.now()) return false;
               // if (millisecs > Date.now() + 3600 * 24 * 45 * 1000) return false;
                return true;
            }} />
        </div>
        {/if}
      </div>
      <div class="btn-bg"></div>
      <div class="btn-modal">
        <button><i class="fas fa-check"></i></button>
      </div>

    </form>
  </div>

  <style type="text/scss">
  .bg-modal {
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

  .bg-modal.active,{
    opacity: 1;
    visibility: visible;
    transition: opacity .25s ease;
  }

  .add-modal.active {
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
  .add-modal {
    @extend %modal;
    height: 75%;
    overflow: hidden;

    h2 {
      margin-bottom: 19px;
      color: #6D4C41;
      text-align: center;
    }

    div {
      margin-bottom: 15px;

      input,
      textarea {
        font-size: 16px;
        display: inherit;
        margin-top: 10px;
        width: 100%;
        padding: 3px;
        border: 0;
        background: #d2691e17;
      }

      .title-mdl {
        color: #EF5350;
        font-weight: 900;
        margin-bottom: 10px;

        i {
          font-size: 12px;
          margin-right: 5px;

        }
      }
    }

    .btn-modal {
      position: absolute;
      bottom: -15px;
      right: 0;

      button {
        @extend %btn;
        padding: 14px;

        i {
          font-size: 20px;
          color: #fff;
        }
      }
    }
  }

  %btn {
    background: linear-gradient(71deg, #fa5e6f 30%, #fb9561 70%);
    border-radius: 50%;
    border: none;
    box-shadow: 2px 6px 9px 2px #ef535061;
  }

 </style>