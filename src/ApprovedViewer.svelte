<script>
import Paybutton from './Paybutton.svelte'
import Dropzone from "svelte-file-dropzone";

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
export let serviceType 
let totalApprovedAmtDue = 0
let totalCompletedAmtDue = 0
let containerArray = []
let rejectedArray = []
let paidArray = []
let currentFilesHolder = []
let initialValue = 0
let serviceFee = 0
let serviceFeeCompleted = 0

let tab 
if(serviceType=='fullservice')
  tab="Completed"
else
  tab="Approved"

let showRetransactModal = false 

let plateNumHolder = ''


function displayLists(){
    if(serviceType=='selfservice'){ //we don't need to display this in full service
    listFolders()
    listRejected()
}

listCompleted()
}

displayLists()

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

async function updateRejectedStatusToPending() {
    const { data, error } = await supabase 
        .from('uploads')
        .update({status: 'pending'})
        .match({platenumber: plateNumHolder})
    
    if(error)
        alert("Error! "+ error.message)
    
}

async function listRejected() {
    const { data, error } = await supabase 
        .from('uploads')
        .select()
        .match({branch: branchName, status: 'rejected'})

        if(error)
            alert('Error! '+error.message)
        if(data)
            rejectedArray = data 
}

async function listCompleted() {
    const { data, error } = await supabase 
        .from('uploads')
        .select()
        .match({branch: branchName, status: 'completed'})

        if(error)
            alert('Error! '+error.message)
        if(data) {
            paidArray = data 
            totalCompletedAmtDue = getTotalAmountDueCompleted(paidArray)
        }

}

async function listFolders() {


    const { data, error } = await supabase 
        .from('uploads')
        .select()
        .match({branch: branchName, status: 'approved'})


    if(error)
        alert('Error! '+error.message)
    if(data) {  
        containerArray = data
        console.log(data)
        let tempVar
        totalApprovedAmtDue = getTotalAmountDue(containerArray)
        //console.log(totalApprovedAmtDue)
    }

}

function getTotalAmountDue(arrayHolder){
let fee = 750

let sum = arrayHolder.reduce(function (previousValue, currentValue) {
    return previousValue + currentValue.total
}, initialValue)

serviceFee = fee * parseFloat(arrayHolder.length)
sum = parseFloat(sum) + parseFloat(serviceFee)

console.log(arrayHolder.length)

return sum
}

function getTotalAmountDueCompleted(arrayHolder){
let fee = 750

let sum = arrayHolder.reduce(function (previousValue, currentValue) {
    return previousValue + currentValue.total
}, initialValue)

serviceFeeCompleted = fee * parseFloat(arrayHolder.length)
sum = parseFloat(sum) + parseFloat(serviceFeeCompleted)


return sum
}



// const mySubscription = supabase
//     .from('uploads')
//     .on('*', ()=>{
//         displayLists()
//     })
//     .subscribe()


    const mySubscription = supabase
    .channel('public:user')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'uploads' }, () =>
    displayLists()
  )
  .subscribe()

function downloadBlob(blob, name) { //download blob as file 

    const blobUrl = URL.createObjectURL(blob)

    const link = document.createElement('a')

    link.href=blobUrl
    link.download=name 

    document.body.appendChild(link)

    link.dispatchEvent(
    new MouseEvent('click', {
        bubbles: true,
        cancelable: true,
        view: window 
    })
    )

    document.body.removeChild(link)


}

async function downloadOR(platenum) {
    const { data, error } = await supabase
        .storage
        .from('transactions')
        .download(platenum+'/'+platenum+'-LTMS-OR')

        if(error)
            alert("Error! "+error.message)
        if(data)
            downloadBlob(data, platenum+'-LTMS-OR')
}

function cleanup() {

    showRetransactModal = false 
        files.accepted.length = 0
        currentFilesHolder.length = 0
}

async function retransactApplication() {

    if(files.accepted.length == 0) {    //meaning there are no files to upload
        cleanup()
        return 
    }
        //console.log("upload additional files")
        files.accepted.forEach(item=>{
            uploadAdditionalFiles(item.name, item)
            updateRejectedStatusToPending()
        })
        cleanup()
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
</script>

<div class="box">
<div class="block">
    <p class="is-size-4 has-text-left">
        Completed Applications
    </p>
</div>

<div class="tabs">
    <ul>

    {#if serviceType=='selfservice'}

      <!-- svelte-ignore a11y-missing-attribute -->
      <li class:is-active={tab=='Approved'}>
        <a on:click={()=>{tab="Approved";listFolders()}}>Approved
            <span class="is-size-7"> ({containerArray.length})</span>
        </a>
    </li>

      <!-- svelte-ignore a11y-missing-attribute -->
      <li class:is-active={tab=='Rejected'}>
        <a on:click={()=>{tab="Rejected";listRejected()}}>Rejected
            <span class="is-size-7"> ({rejectedArray.length})</span>
        </a>
    </li>

    {/if}

      <!-- svelte-ignore a11y-missing-attribute -->
      <li class:is-active={tab=='Completed'}>
        <a on:click={()=>{tab="Completed";listCompleted()}}>Completed
            <span class="is-size-7"> ({paidArray.length})</span>
        </a>
    </li>

    </ul>
  </div>

        <table class="table is-fullwidth fluid">
            {#if tab=="Approved"}
            <thead>
                <th>Date</th>
                <th>Plate #</th>
                <th>Total</th>
            </thead>
                {#each containerArray as item}
                    <tr>
                        <td>
                            {new Date(item.created_at).toDateString()}
                        </td>
                        <td>
                            <i class="fa fa-check-circle"></i> {item.platenumber}
                        </td>

                        <td>
                            {item.total}
                        </td>
                    </tr>
                {/each}
  
            <tfoot>
                <tr>
                    <td>

                    </td>
                    <td>
                        Total Service Fee:
                    </td>
                    <td class="has-text-weight-bold">
                        {serviceFee}
                    </td>
                </tr>
                <tr>
                    <td>

                    </td>
                    <td>
                        Total amount due:
                    </td>
                    <td class="has-text-weight-bold">
                        {totalApprovedAmtDue}
                    </td>
                </tr>
 
            </tfoot>

            {/if}  

            {#if tab=="Rejected"}
            <thead>
                <th>Plate #</th>
                <th>Remarks</th>
            
            </thead>
            {#each rejectedArray as item}
            <tr>
                <td>
                    <i class="fa fa-times-circle"></i> {item.platenumber}
                </td>

                <td>
                    {item.remarks}
                </td>
                <td class="has-text-centered">
                    <i class="fa fa-recycle is-clickable" on:click={()=>{
                        showRetransactModal=true
                        plateNumHolder = item.platenumber 
                        listCurrentFiles()
                    }}></i>
                </td>
            </tr>
             {/each}
            {/if}

            {#if tab=="Completed"}
            <thead>
                <th>Date Completed</th>              
                <th>Plate #</th>
                <th>Date Paid</th>
                <th>Service Fee</th>
                <th>Total Paid</th>
                <th></th>
             
            </thead>
            {#each paidArray as item}
            <tr>
                <td>
                    {new Date(item.completed_at).toDateString()}
                </td>              
                <td>
                    <i class="fa fa-check"></i> {item.platenumber}
                </td>
                <td>
                    {new Date(item.paid_at).toDateString()}
                </td>
                <td>
                     {item.servicefee}
                </td>

                <td>
                    {item.actual_amount}
                </td>
                <td class="is-clickable has-text-centered is-size-5" title="Click to download OR" on:click={()=>downloadOR(item.platenumber)}>
                    <i class="fa fa-download"></i>
                </td>
            </tr>
             {/each}
             <tfoot>
                <!-- <tr>
                    <td>

                    </td>
                    <td>
                        Total Service Fee:
                    </td>
                    <td class="has-text-weight-bold">
                        {serviceFeeCompleted}
                    </td>
                </tr> -->
  
                    <!-- <td colspan="3">

                    </td>
                    <td>
                        Total amount
                    </td>
                    <td class="has-text-weight-bold">
                        {totalCompletedAmtDue}
                    </td> -->

            </tfoot>
            {/if}

        </table>
        {#if serviceType == "selfservice"}
            {#if tab=="Approved"}
            <div class="block mt-6">
                <Paybutton branchName = {branchName} amtDue = {totalApprovedAmtDue}/>
            </div>
            {/if}
        {/if}
<!--RETRANSACT MODAL-->
<!--################-->

<div class="modal" class:is-active={showRetransactModal}>
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">Retransact Application</p>
        <button class="delete" aria-label="close" on:click={()=>{
            cleanup()
            showRetransactModal=false
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
                        <td>
                            {item.name}
                        </td>
                        <td>
                            <i class="fa fa-times-circle is-clickable" on:click={()=>{
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
        
        <button class="button is-success" on:click={retransactApplication}>OK</button>
        <button class="button" on:click={()=>{
            showRetransactModal=false
            cleanup()
            }}>Cancel</button>
      </footer>
    </div>
  </div>
</div>

<style>
    table{
        overflow-wrap: anywhere;
    }
</style>