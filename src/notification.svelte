<script>
  let showNotif = true;
   export let notifMessage = " GeeksforGeeks is a computer science portal.i"
   export let isNotifSucess = true;
</script>

<div class="modal" class:is-active={showNotif}>
   {#if isNotifSucess}
   <div class='column is-6'>
    <div class='notification is-link'>
      <h1 class='title'>
        sucess
        <slot>

        </slot>
      </h1>

      <p class='is-family-monospace'>
         {notifMessage}
      </p>

    </div>
  </div>
  {:else}
  <div class='column is-6'>
    <div class='notification is-danger'>
      <h1 class='title'>
        fail
        <slot>

        </slot>
      </h1>

      <p class='is-family-monospace'>
         {notifMessage}
      </p>

    </div>
  </div>
   {/if}
   
</div>
  <style>
    .modal.is-active {
    align-items: center;
    justify-content: flex-start;
    }
    .title{
        display: flex;
    align-items: flex-end;
    justify-content: space-between;
    }
    .column{
        width: 350px;
    }
  
  </style>