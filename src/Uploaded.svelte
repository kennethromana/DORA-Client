<script>
import Dropzone from 'svelte-file-dropzone'

let files = {
    accepted: [],
    rejected: []
  };

  function handleFilesSelect(e) {
    const { acceptedFiles, fileRejections } = e.detail;
    files.accepted = [...files.accepted, ...acceptedFiles];
    files.rejected = [...files.rejected, ...fileRejections];
  }


export let branchName 


let containerArray = []
let currentFilesHolder = []

let showAddEditModal = false 
let plateNumHolder = ''

listFolders()

async function listFolders() {
    const { data, error } = await supabase 
        .from('uploads')
        .select()
        .match({branch: branchName, status: 'pending'})
        .order('created_at',{ascending:  false})

    if(error)
        alert('Error! '+error.message)
    if(data) {
    
        containerArray = data
        //console.log(data)
    }

}

async function listCurrentFiles() {
    const { data, error } = await supabase
        .storage
        .from('transactions')
        .list(plateNumHolder)

        if(error)
            alert('Error! '+ error.message)
        if(data){
            currentFilesHolder = data 
            //console.log(data)
        }
            
}

// const mySubscription = supabase
//     .from('*')
//     .on('*', listFolders)
//     .subscribe()


const mySubscription = supabase
  .channel('public:uploads')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'uploads' }, () =>
    listFolders()
  )
  .subscribe()

function cleanup() {

showAddEditModal = false 
    files.accepted.length = 0
    currentFilesHolder.length = 0
}

async function removeFile(fname) {
    const { data, error } = await supabase 
        .storage 
        .from('transactions')
        .remove([plateNumHolder+'/'+fname])
    
    if(error)
        alert("Error! "+error.message)
    if(data)
        listCurrentFiles()
}

async function uploadAdditionalFiles(fname, fHandle) {
    const { data, error } = await supabase 
        .storage 
        .from('transactions')
        .upload(plateNumHolder+'/'+fname, fHandle, {
            cacheControl: '3000',
            upsert: false 
        })

        if(error)
            alert("Error! " + error.message)
}


async function addFilesInApplication() {
    //add files to application
    if(files.accepted.length==0) {
        cleanup()
        return 
    }


        files.accepted.forEach(item=>{
            uploadAdditionalFiles(item.name, item)
            
        })
        cleanup() 
        listFolders()   
}

</script>
<div class="box">


<div class="block has-text-left">
    <p class="is-size-4">Submitted Applications</p>
</div>

<table class="table is-fullwidth">
    {#each containerArray as { branch, created_at, platenumber }}
        <tr class="is-clickable">
            <td title="Plate Number"><i class="fa fa-folder"></i> {platenumber}</td>
            <td title="Date Submitted" class="is-size-6">{new Date(created_at).toLocaleDateString()}</td>    
            <td>
                <i title="Click to edit" class="is-size-5 fa fa-edit is-clickable" on:click={()=>{
                        plateNumHolder = platenumber 
                        showAddEditModal = true 
                        listCurrentFiles()
                }}></i>
            </td> 
   
        </tr>
        {:else}                  
         <p class="has-text-grey-light  has-text-centered block">There are no data to show . . .</p>

    {/each}
</table>


<div class="modal" class:is-active={showAddEditModal}>
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">Edit Application</p>
        <button class="delete" aria-label="close" on:click={()=>{
            cleanup()
            showAddEditModal=false
            }}></button>
      </header>
      <section class="modal-card-body">
        <div class="block">
            <p class="is-size-2">{plateNumHolder}</p>
        </div>
        
        <div class="block">
            <p class="is-size-5">
                Existing Files 
            </p>
            <table class="table">
                {#each currentFilesHolder as item}
                    <tr>
                        <td class="is-size-6 pt-3">
                            {item.name}
                        </td>
                        <td>
                            <i class="fa fa-remove is-clickable" on:click={()=>{
                                removeFile(item.name)
                                
                            }}></i>
                        </td>
                    </tr>
                {/each}
            </table>

        </div>

        <div class="block">
            <p class="is-size-5">
                Upload Additional Files 
            </p>
            <Dropzone multiple=true accept="application/pdf,image/*" on:drop={handleFilesSelect} />
                <table class="table">
                {#each files.accepted as item}
                <tr>
                    <td class="is-size-7">{item.name}</td>
                </tr>    
                {/each}
                </table>
        </div>

      </section>

      <footer class="modal-card-foot">
        
        <button class="button is-success" on:click={addFilesInApplication}>OK</button>
        <button class="button" on:click={()=>{
            showAddEditModal=false
            cleanup()
            }}>Cancel</button>
      </footer>
    </div>
  </div>
</div>