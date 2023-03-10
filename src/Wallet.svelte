<script>
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
let balance = 0
let showModal = false 
let topupRequestAmount = 0

let tabs = "Top Up Request"
let historyHolder = []

async function getTopUpHistory() {
  const { data, error } = await supabase 
    .from('topups')
    .select()
    .eq('branch', branchName)

    if(error)
      alert("Error! "+error.message)
    
    if(data) {
      historyHolder = data 
    }
}

async function getBalance() {
    //retrieve wallet balance
    console.log(branchName)
    const { data, error } = await supabase
        .from('wallet_balance')
        .select()
        .eq('branch', branchName)

    if(error)
        alert('Error! ' + error.message)
    
    if(data)
        balance=data[0].balance
}

getBalance()

const mySubscription = supabase 
    .from('wallet_balance')
    .on('*', getBalance)
    .subscribe()

function cleanup(){
  showModal = false 
  files.accepted.length = 0 
  topupRequestAmount = 0
}

async function uploadProofOfDeposit() {
  const { data, error } = await supabase 
    .storage 
    .from('proof/'+ branchName)
    .upload(files.accepted[0].name, files.accepted[0], {
      cacheControl: '3600',
      upsert: false
    })
    if(error)
      alert('Error! '+error.message)
    if(data) {
      alert('Request sent')
      cleanup()
    }

}

async function sendTopupRequest() {
  const { data, error } = await supabase 
    .from('topups')
    .insert([
      {
        branch: branchName,
        amount: topupRequestAmount,
        status: 'requested',
        filename: files.accepted[0].name 
      }
    ])
    if(error)
      alert("Error! "+error.message)

    if(data){
      uploadProofOfDeposit()
    }
}    
</script>

<div class="column has-text-right">
<span class="pr-3 is-clickable" title="Click here to top up your wallet" on:click={()=>{showModal=true}}>
   <span class="is-size-3"> Your Wallet Balance Is: </span> 
   <span class="is-size-3" class:has-text-danger={parseInt(balance)<500}>{balance}</span>
</span>
</div>

<!--TOP UP REQUEST MODAL-->

<div class="modal" class:is-active={showModal}>
    <div class="modal-background"></div>
    <div class="modal-card">
      <header class="modal-card-head">
        <p class="modal-card-title">Top Up Request</p>
        <button class="delete" aria-label="close" on:click={cleanup}></button>
      </header>
      <section class="modal-card-body">
        <!-- Content ... -->

        <div class="tabs">
          <ul>
            <!-- svelte-ignore a11y-missing-attribute -->
            <li class="is-clickable" class:is-active={tabs=='Top Up Request'} on:click={()=>tabs='Top Up Request'}><a>Top Up Request</a></li>
            <!-- svelte-ignore a11y-missing-attribute -->
            <li class="is-clickable" class:is-active={tabs=='History'} on:click={()=>{
              tabs='History'
              getTopUpHistory()
              }}><a>History</a></li>
          </ul>
        </div>

        {#if tabs=='Top Up Request'}
        <div class="block">
          <p class="is-size-7">
            Step 1: Deposit your desired top up amount into the Databridge Bank Account
          </p>
          <ul class="is-size-7 ml-3 mt-3">
            <li>Account Name: Databridge Asia Inc</li>
            <li>Account Number: xxx-xxx-xxx</li>
          </ul>
        </div>
          <div class="block">
            <p class="is-size-7">Step 2: Enter The Deposited Amount</p>
            <input type="text" class="input is-fullwidth" bind:value={topupRequestAmount}>
          </div>

          <div class="block">
            <p class="is-size-7">Step 3: Attach Proof of Deposit</p>
            <Dropzone on:drop={handleFilesSelect} />
            <ul>
              {#each files.accepted as item}
                <li class="is-size-7">{item.name}</li>
              {/each}
            </ul>          
          </div>
        {/if}

        {#if tabs == 'History'}
            <div class="table-container">
              <table class="table">
                <thead>
                  <th>Request Date</th>
                  <th>Request Amount</th>
                  <th>Status</th>
                  <th>Remarks</th>
                </thead>
                {#each historyHolder as item}
                  <tr>
                    <td>{new Date(item.created_at).toDateString()}</td>
                    <td>{item.amount}</td>
                    <td>{item.status}</td>
                    <td>{item.remarks}</td>
                  </tr>
                {/each}
              </table>
            </div>
        {/if}

      </section>
      <footer class="modal-card-foot">
        {#if tabs=='Top Up Request'}
        <button class="button is-success" on:click={sendTopupRequest} >OK</button>
        <button class="button" on:click={cleanup}>Cancel</button>
        {:else}
          <button class="button is-success" on:click={cleanup}>OK</button>
        {/if}
      </footer>
    </div>
  </div>