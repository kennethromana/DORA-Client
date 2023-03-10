
<script>

export let branchName

function capitalizeFirstLetter(string) {
  return string.charAt(0).toUpperCase() + string.slice(1);
}

</script>
<style>
  .navbar-item {
    
    height: 55px;
    width: 102px;
    float: left;
    padding-left: 4%;
    margin: 2px;

}
.Navbar{
  /* height: 100%;
  width: 100%;
  max-height: 3.75rem; */
  background-color: #485fc77d;;
   
}
.logo{
  height: 96%;
    width: 82%;
    max-height: 2.75rem
}

</style>
<nav class="Navbar navbar is-link"  aria-label="main navigation">
    <div class="navbar-brand">
      <!-- svelte-ignore a11y-invalid-attribute -->
      <a class="navbar-item" href="#">
        <!-- svelte-ignore a11y-missing-attribute --> 
        {#if branchName.includes('honda')}
        <img class="logo" src="honda-logo11.png" alt="logo">
        {:else if branchName.includes('isuzu')}
        <img class="logo" src="isuzu-logo3.png" alt="logo">
        {:else if branchName.includes('toyota')}
        <img class="logo" src="toyota-logo3.png" alt="logo">
        {:else if branchName.includes('vespa')}
        <img class="logo" src="vespa-logo.png" alt="logo">
        {:else}
        <!-- <img class="logo" src="logo.png" alt="logo"> -->
        {/if}
      </a>
      <span class="is-size-3 mt-1">{capitalizeFirstLetter(branchName)}</span>
      <!-- svelte-ignore a11y-missing-attribute -->
      <a role="button" class="navbar-burger" aria-label="menu" aria-expanded="false" data-target="navbarBasicExample">
        <span aria-hidden="true"></span>  
        <span aria-hidden="true"></span>
        <span aria-hidden="true"></span>
      </a>
    </div>
  
    <div id="navbarBasicExample" class="navbar-menu">
      <div class="navbar-start">

  
        <div class="navbar-item has-dropdown is-hoverable">

        </div>
      </div>
  
      <div class="navbar-end">
        <div class="navbar-item">
          <div class="buttons">

            <slot>

            </slot>
          </div>
        </div>
      </div>
    </div>
  </nav>