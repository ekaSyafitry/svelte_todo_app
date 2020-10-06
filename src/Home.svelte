<script>
  import DatePicker from "./components/DatePicker.svelte";
  import Modal from "./components/Modal.svelte";
  import ModalConfirm from "./components/ModalConfirm.svelte"
  import { onMount } from 'svelte';
  let currentDate = new Date();

  const onDateChange = d => {
    currentDate = d.detail;
    formatDate()
    getData()
    btnAll()
  };

  let sel_date,day,isLoad,hasLoad, complete, totalComplete,num;
  const database = firebase.database();
  let todolist =[],
      todos = [],
      todolist_completed = [],
      todolist_incompleted = [],
      current = 'all',
      addActive = false,
      editActive = false,
      dataModal = [],
      completeActive = false,
      id_todo = '',
      incompleteActive = false,
      trashActive = false,
      percent = 1,
      totalIncomplete = 0,
      sum = 0,
      showMenu = false;

  const formatDate = () =>{
    let now = new Date()
    if (currentDate.toDateString() !== now.toDateString()){
    var months = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sept","Oct","Nov","Dec",];
    sel_date =  months[(currentDate.getMonth())] + " " + (currentDate.getDate()) + " " + currentDate.getFullYear()
    }
    else {
      sel_date = "Today's tasks"
    }
     }

  const getData = async () =>{
        isLoad = true
        let g_date = currentDate.getFullYear() + "-" +  (currentDate.getMonth() + 1) + "-" + currentDate.getDate() 
        database.ref('todolist').orderByChild('date').equalTo(g_date).on('value', (snapshot) => { realtimedata(snapshot) })     
  }

  const realtimedata = async (data) =>{
    // console.log(data)
    let reads = []
    data.forEach(function (val) {
      reads.push({...val.val(), id: val.key})
    })
    todolist = reads
    isLoad = false
    hasLoad = true
    todolist_completed = todolist.filter((todo)=> { return todo.complete  })
    todolist_incompleted = todolist.filter((todo)=> {return !todo.complete})
    if (current == 'all'){
      todos = todolist
    }
    else if( current == 'com'){
      todos = todolist_completed
    }
    else{
      todos = todolist_incompleted
    }
    // console.log(todos)
    countProgress()
  }

  const btnAll = () => {
    current = 'all'
    todos = todolist 
    setTimeout(() => {
      // console.log(document.getElementById("myBar"))
    countProgress()
    }, 500);
   showMenu = false
  }

  const btnCom = () => {
    current = 'com'
    todos = todolist_completed
    showMenu = false
  }

  const btnIncom = () => {
    current = 'incom'
    todos = todolist_incompleted
    showMenu = false
    // console.log(todolist_incompleted)
  }
  const editData = td =>{ 
    // console.log('sdfsd')
    editActive = true
    let tgl = new Date(td.date)
    td.date = tgl
    dataModal = td
  }

  const completeData = td =>{
    completeActive = true
    id_todo = td.id
    complete = td.complete
  }

  const incompleteData = td => {
    incompleteActive = true
    id_todo = td.id
    complete = td.complete
  }

  const confirmComplete = () => {
    // console.log(complete, 'complete')
    // if (complete == true) {
      database.ref('todolist/' + id_todo + '/complete').set(true);
      completeActive = false   
    // } else{
     
    // }
  }

  const confirmIncomplete = () => {
    // console.log(id_todo)
  database.ref('todolist/' + id_todo + '/complete').set(false);
    incompleteActive = false
    // console.log(incompleteActive)
}

  const deleteData = id => {
    id_todo = id
    trashActive = true
  }

  const confirmDelete = () =>{
    let userRef = database.ref('todolist/' + id_todo);
    userRef.remove()
    trashActive = false
  }

  const countProgress = () => {
    if (current == 'all'){
      totalComplete = todolist_completed.length
      totalIncomplete = todolist_incompleted.length
      sum = todos.length
      num = totalComplete / sum * 100
      percent = num.toFixed(1);
      // console.log(percent)
      var elem = document.getElementById("myBar");
      if(todos.length == 0){
        elem.style.width = "0%";  
      }
      else{
        elem.style.width = percent + "%";  
      }
    }
  }
  
  const toogleMenu = () => {
    showMenu = !showMenu;
    // console.log('czndsdjk')
  }

  const installModal = () => {
    if(promptInstall){
    promptInstall.prompt()
    promptInstall.userChoice.then(function(choiceResult){
      console.log(choiceResult.outcome);
      if(choiceResult.outcome==='dismissed'){
        console.log('user cancelled installation');
      }else{
        console.log('user add to home screen');
      }
    });
    promptInstall = null;
  }
  }

  formatDate()
  getData()
  // countProgress()
  
  // btnAll()

</script>


<slot>
  <div class="container home">
    <div class="calender-box">

      <div class="header">
        <h1>To-do</h1>
        <div class="box-date">
          <div on:click={installModal}> 
            <i class="fas fa-arrow-alt-circle-down" style="font-size:25px"></i>
          </div>
          <div class="cal">
            <DatePicker on:datechange={onDateChange} selected={currentDate} type="Home" />
          </div>
        </div>
      </div>
    </div>



    <div class="card">
      <div class="wrapTitle">
        {#if current == 'all'}
        <div class="wrapDay">
          <h2>{sel_date}</h2>
          <div class="btnFilter" on:click={toogleMenu}><i class="fas fa-filter"></i></div>
          <div class="{showMenu? "circle show" : "circle"}">
            <div id="all" on:click={btnAll} class="{current === 'all' ? 'active' : ''}">All</div>
            <div id="complete" on:click={btnCom} class="{current === 'com' ? 'active' : ''}">Complete</div>
            <div id="incomplete" on:click={btnIncom} class="{current === 'incom' ? 'active' : ''}">Incomplete</div>
          </div>
        </div> 
       <div class="unfinished">{totalIncomplete} unfinished tasks from total {sum} </div>
          <div class="meter">
            <div id="myBar"></div>
          </div>
        {:else if current == 'com'}
        <div class="wrapDay">
          <h2>Complete tasks</h2>
          <div class="btnFilter" on:click={toogleMenu}><i class="fas fa-filter"></i></div>
          <div class="{showMenu? "circle show" : "circle"}">
            <div id="all" on:click={btnAll} class="{current === 'all' ? 'active' : ''}">All</div>
            <div id="complete" on:click={btnCom} class="{current === 'com' ? 'active' : ''}">Complete</div>
            <div id="incomplete" on:click={btnIncom} class="{current === 'incom' ? 'active' : ''}">Incomplete</div>
          </div> 
        </div> 
        <div style="color: #60563d;">{totalComplete} tasks </div>
        {:else if current == 'incom'}
        <div class="wrapDay">
          <h2>Incomplete tasks</h2>
          <div class="btnFilter" on:click={toogleMenu}><i class="fas fa-filter"></i></div>
          <div class="{showMenu? "circle show" : "circle"}">
            <div id="all" on:click={btnAll} class="{current === 'all' ? 'active' : ''}">All</div>
            <div id="complete" on:click={btnCom} class="{current === 'com' ? 'active' : ''}">Complete</div>
            <div id="incomplete" on:click={btnIncom} class="{current === 'incom' ? 'active' : ''}">Incomplete</div>
          </div>
        </div> 
        <div style="color: #60563d;">{totalIncomplete} tasks </div>
        {/if}
      </div>
      <div class="card-box">
        {#if isLoad}
          <div class="wrapLoad">
            <div class="lds-roller"><div></div><div></div><div></div><div></div></div>
          </div>
        {/if}
        {#if hasLoad}
          <div >
            {#if todos.length == 0}
            <div style="padding-top: 25px;">
              <img src="undraw_empty_xct9.svg" alt="" width="100%">
              <div class="empty">No plan yet..</div>
            </div>
            {:else}
            <ul style="list-style-type: none;">
              {#each todos as todo}
              <li class="card-list">
                <div class="btn-check">
                  {#if todo.complete}
                   <i class="fas fa-check-circle" on:click={() => incompleteData(todo)}></i>
                  {:else}
                   <i class="far fa-circle"  on:click={() => completeData(todo)}></i>
                  {/if}
                </div>
                <div style="width: 73%; margin-right: 8px;">
                  <div class={todo.complete? 'title cross' : 'title' }>{todo.name}</div>
                  {#if todo.notes != ''}
                  <div class={todo.complete? 'card-desc cross' : 'card-desc' }>{todo.notes}</div>
                  {/if}
                </div>
               
                <div class="btn-group">
                  <div on:click={() => editData(todo)}><i class="far fa-edit"></i></div>
                  <div on:click={() => deleteData(todo.id)}><i class="far fa-trash-alt"></i></div>
                </div>
              </li>
              {/each}
            </ul>
            {/if}
            <div class="btn-float" on:click={() => (addActive = true)}>
              <button ><i class="fas fa-plus"></i></button>
            </div>
          </div>
        {/if}
      </div>  
    </div>
    <Modal modalActive={addActive} type="add" on:close={() => (addActive = false)} data={dataModal}/>
      <Modal modalActive={editActive} type="edit" data={dataModal} on:close={() => (editActive = false)} />
      <ModalConfirm modalActive={completeActive} type="complete" on:close={() => (completeActive = false)} on:yesBtn={confirmComplete} />
      <ModalConfirm modalActive={incompleteActive} type="incomplete" on:yesBtn={confirmIncomplete} on:close={() => (incompleteActive = false)} />
      <ModalConfirm modalActive={trashActive} type="trash" on:yesBtn={confirmDelete} on:close={() => (trashActive = false)} />
  </div>
</slot>

<style type="text/scss">
  .container {
    max-width: 500px;
    height: 100vh;
    overflow: hidden;
    margin: 0 auto;
    background: #fff;
    display: grid;
    // background: #fff;
    // grid-template-rows: 2fr 4fr;
    grid-template-rows: 13% 87%;
    background: linear-gradient(71deg, rgba(250, 94, 111, 1) 30%, rgba(251, 149, 97, 1) 70%);
  }

  .calender-box {
    background: linear-gradient(71deg, rgba(250, 94, 111, 1) 30%, rgba(251, 149, 97, 1) 70%);
    display: grid;
    // grid-template-columns: 2fr 1fr;
    // padding: 0 25px;
    // position: relative;

    // .capt {
    //   margin-top: 14px;
    //   font-size: 11px;
    //   width: 100px;
    //   color: #fff;
    //   font-style: italic;
    //   opacity: 0.7;
    // }

    h1 {
      color: #fff;
      align-self: center;
    }

    .box-date {
      align-self: flex-end;
      color: white;
      display: flex;
      width: 60px;
      justify-content: space-between;

      // .date {
      //   text-align: end;

      //   i {
      //     font-size: 30px;
      //     margin-bottom: 5px;
      //   }
      // }

      .cal {
        align-self: end;
        text-align: end;
      }
    }

    .header{
      margin:0 25px;
      align-self: center;
      display: flex;
      justify-content: space-between;
    }
  
  }

  .card {
    // padding: 25px 0;
    background-color: #fff;
    // overflow-y: scroll;
    border-radius: 50px 0 0 0;

    .wrapTitle{
    position: relative;
    padding: 25px;
    background-color: #fff;
    border-radius: 60px 0 0;
    // height: 130px;
    .wrapDay{
      display: flex;
      align-items: center;
      justify-content: space-between;
      .btnFilter{
        // position: relative;
        i{
        font-size: 20px;
        color: #60563d;
        }
      }
      .circle {
        opacity: 0;
        visibility: hidden;
        border-radius: 10px;
        width: 105px;
        display: block;
        position: absolute;
        top: 65px;
        right: 17px;
        z-index: 2;
        padding-left: 0px;
        background: #fff;
        box-shadow: -2px 6px 18px #ccc;
        transition: all 0.8 ease-in-out;

      div {
        padding: 5px 15px;
        // border-radius: 0;
        background: #cccccc14;
        font-size: 14px;
        color: #6D4C41;  
      }
      
      .active {
        background-color: #60563d;;
        color: #fff ;
        border-radius: 10px;
        }
    }
     }

     .circle.show{
       opacity: 1;
       visibility: visible;
     }

    h2, .unfinished{
      color: #60563d;
      margin-bottom: 10px;
    }

    .unfinished{
      font-size: 14px;
    }

    .meter { 
     height: 9px;
    position: relative;
    background: #e3e1de;
    -moz-border-radius: 25px;
    -webkit-border-radius: 25px;
    border-radius: 25px;
    // margin-bottom: 1.5rem;
    // padding: 5px;
    box-shadow: inset 0 -1px 1px rgba(255, 255, 255, 0.3);
      #myBar {
        border-radius: 25px;
        width: 0%;
        height: 9px;
        background-color: #fb756a;
        padding: 4px 0;
        transition: all 1s ease-in-out; 
    }
   }

  }
  
  .card-box {
    padding: 0 25px 25px 25px;
    max-height: 75%;
    overflow: auto;
  }
    .empty {
      margin-top: 30px;
      font-size: 18px;
      color: #EF5350;
    }

    .card-list {
      margin-bottom: 20px;
      margin-bottom: 20px;
      display: flex;
      align-items: center;
      border: 1px solid #ccc;
      padding: 16px 10px;
      border-radius: 20px;
      .btn-check{
        width: 10%;
        i {
          font-size: 18px;
          margin-right: 10px;
          color: #fb756a;
        }
      }

      .title {
        text-transform: capitalize;
        font-weight: 900;
        color: #60563d;
        overflow: hidden;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;  
      }

      .title.cross,
      .card-desc.cross {
        text-decoration: line-through;
        text-decoration-color: #6D4C41;
        opacity: 0.4;
      }

      .card-desc {
        overflow-wrap: anywhere;
        margin-top: 7px;
        font-size: 14px;
        color: #60563d;;
        line-height: 15px;
      }

      .btn-group {
        display: flex;
        width: 15%;
        justify-content: space-between;

        i {
          font-size: 14px;
          color: #EF5350;
        }
      }

    }

    .btn-float {
      position: fixed;
      bottom: 0;
      right: 0;
      margin-bottom: 15px;
      margin-right: 15px;

    }

    button {
      @extend %btn;
      padding: 14px 16px;

      i {
        font-size: 20px;
        color: #fff;
      }
    }
  }

  %btn {
    background: linear-gradient(71deg, #fa5e6f 30%, #fb9561 70%);
    border-radius: 50%;
    border: none;
    box-shadow: 2px 6px 9px 2px #ef535061;
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

  %btnConfirm {
    font-weight: 600;
    color: #fff;
    font-size: 16px;
  }

  button:focus {
    outline: none;
  }
  .wrapLoad{
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    left: 0;
    /* align-items: center; */
    position: absolute;
  }
  .lds-roller {
    width: 80px;
    height: 80px;
  }

  .lds-roller div {
    animation: lds-roller 1.2s cubic-bezier(0.5, 0, 0.5, 1) infinite;
    transform-origin: 40px 40px;
  }

  .lds-roller div:after {
    content: " ";
    display: block;
    position: absolute;
    width: 7px;
    height: 7px;
    border-radius: 50%;
    background: #EF5350;
    margin: -4px 0 0 -4px;
  }

  .lds-roller div:nth-child(1) {
    animation-delay: -0.036s;
  }

  .lds-roller div:nth-child(1):after {
    top: 63px;
    left: 63px;
  }

  .lds-roller div:nth-child(2) {
    animation-delay: -0.072s;
  }

  .lds-roller div:nth-child(2):after {
    top: 68px;
    left: 56px;
  }

  .lds-roller div:nth-child(3) {
    animation-delay: -0.108s;
  }

  .lds-roller div:nth-child(3):after {
    top: 71px;
    left: 48px;
  }

  .lds-roller div:nth-child(4) {
    animation-delay: -0.144s;
  }

  .lds-roller div:nth-child(4):after {
    top: 72px;
    left: 40px;
  }

  .lds-roller div:nth-child(5) {
    animation-delay: -0.18s;
  }

  .lds-roller div:nth-child(5):after {
    top: 71px;
    left: 32px;
  }

  .lds-roller div:nth-child(6) {
    animation-delay: -0.216s;
  }

  .lds-roller div:nth-child(6):after {
    top: 68px;
    left: 24px;
  }

  .lds-roller div:nth-child(7) {
    animation-delay: -0.252s;
  }

  .lds-roller div:nth-child(7):after {
    top: 63px;
    left: 17px;
  }

  .lds-roller div:nth-child(8) {
    animation-delay: -0.288s;
  }

  .lds-roller div:nth-child(8):after {
    top: 56px;
    left: 12px;
  }

  @keyframes lds-roller {
    0% {
      transform: rotate(0deg);
    }

    100% {
      transform: rotate(360deg);
    }
  }
</style>