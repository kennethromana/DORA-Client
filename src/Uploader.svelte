<script>
  import Dropzone from "svelte-file-dropzone";
  import Notif from "./notification.svelte"


let plateNo = ''
export let branchName 
export let serviceType 


let files = {
  accepted: [],
  rejected: []
};

function handleFilesSelect(e) {
  const { acceptedFiles, fileRejections } = e.detail;
  files.accepted = [...files.accepted, ...acceptedFiles];
  files.rejected = [...files.rejected, ...fileRejections];
}

    let tabOne = true
    let tabTwo = false
    let tabThree = false
    let tabFour = false
    let COCModal = false
    
    let SelectedCOC = "Select Insurance"
    let InsuranceList = [
		{ id: 1, text: `Paramount Life & General Insurance` },
		{ id: 2, text: `Malayan Insurance Company`},
		{ id: 3, text: `Sample` }
	];


    let SelectedMVType = "Select MV Type"
    let MVTypeList = [
		{ id: 1, text: `UV - Utility Vehicle` },
		{ id: 2, text: `TV - Truck Bus`},
		{ id: 3, text: `M - Motor Cycle` }
	];

    let SelectedVClassification = "Select Classification"
    let VClassificationList = [
		{ id: 1, text: `AC and Tourist Cars` },
		{ id: 2, text: `Private Cars`},
		{ id: 3, text: `Taxi ,  PUJ , BUS` }
	];

    let SelectedPremiumType = "Select Classification"
    let PremiumTypeList = [
		{ id: 1, text: `AC and Tourist Cars` },
		{ id: 2, text: `Private Cars`},
		{ id: 3, text: `Taxi ,  PUJ , BUS` }
	];

    let SelectedTaxType = "Select Tax Type"
    let TaxTypeList = [
		{ id: 1, text: `VAT Exempt` },
		{ id: 2, text: `VAT`},
		{ id: 3, text: `Non - VAT` }
	];

    let SelectedVUsage = "Select Usage"
    let VUsageList = [
		{ id: 1, text: `VAT Exempt` },
		{ id: 2, text: `VAT`},
		{ id: 3, text: `Non - VAT` }
	];

    let Requestparams = {
      mv_file_no:"",
      engine:"",
      chassis:"",
      inception_date:"",
      expiry_date:"",
      mv_type:"",
      premium_type:"",
      tax_type:"",
      assured_name:"",
      assured_tin:""
    }

    let EngineNo;
    let ChassisNo;
    let MVFileNo;
    let AssuredName;
    let AssuredTIN;
    let InceptionDate;
    let ExpiryDate;

    let Client_ID = "";
    let Business_ID = "";





    let renewalType = 'individual'
    let customerName = ''

    async function insertIntoRecords() {


        const { data, error } = await supabase 
            .from('uploads')
            .insert([
                {
                    platenumber: plateNo,
                    name: customerName,
                    branch: branchName,
                    renewaltype: renewalType,
                    servicetype: serviceType,
                    client_id:Client_ID,
                    business_id:Business_ID,
            }
        ])


        if(error){
            notifMessage = "Error! "+error.message;
            isNotifSucess = false
        }
        else 
        {
            notifMessage = "Plate "+plateNo+" Submitted!"
            isNotifSucess = true
        }
            // alert()
        
    }

    function falseEverything() {
        tabOne = tabTwo = tabThree= tabFour = false 
    }

    async function uploadFiles() {

        //prepare files for upload
        await prepareFilesForUpload()

        //insert record into database
        await insertIntoRecords()

        //variable cleanup
        falseEverything()
        renewalType = ''
        tabOne=true 
        files.accepted.length = 0
        plateNo = ''
        customerName = ''

        EngineNo= ''
        ChassisNo= ''
        MVFileNo= ''
        AssuredName= ''
        AssuredTIN= ''
        InceptionDate= ''
        ExpiryDate= ''

        Client_ID = "";
        Business_ID = "";
        
        showNotif = true;
        

    }


async function uploadFileToBucket(name, fileHandle, bucket) {
    //supabase upload to storage bucket
    const { data, error } = await supabase 
        .storage 
        .from('transactions/'+bucket)
        .upload(name, fileHandle, {cacheControl: '3600', upsert: false})
}

async function prepareFilesForUpload() {
        //actual upload starts here
        files.accepted.forEach((value, index)=>{
            uploadFileToBucket(value.name, value, plateNo)
        })  
}

// function keyforclient() {

// var value = Client_ID
// if (value.length == 3 || value.length == 7 || value.length == 11) {
//     Client_ID = (value + "-")
// }

// }

let value = "";
function isNumber(value) {
        return !isNaN(value);
    }

    function handleInput(e) {

    if(renewalType == "individual"){
        
        var foo = Client_ID.replace(/-/g, ""); // remove hyphens
    // You may want to remove all non-digits here
     foo = foo.replace(/\D/g, "");

        if (foo.length > 0) {
        foo = format(foo, [2, 6], "-");
       }
       Client_ID = foo;
    }
    else{
        
        var foo = Business_ID.replace(/-/g, ""); // remove hyphens
    // You may want to remove all non-digits here
     foo = foo.replace(/\D/g, "");

        if (foo.length > 0) {
        foo = format(foo, [3, 2], "-");
       }
       Business_ID = foo;


       var foo2 = Client_ID.replace(/-/g, ""); // remove hyphens
    // You may want to remove all non-digits here
    foo2 = foo2.replace(/\D/g, "");

        if (foo2.length > 0) {
            foo2 = format(foo2, [2, 6], "-");
       }
       Client_ID = foo2;
    }
     


    }


    function format(input, format, sep) {
    var output = "";
    var idx = 0;
    for (var i = 0; i < format.length && idx < input.length; i++) {
        output += input.substr(idx, format[i]);
        if (idx + format[i] < input.length) output += sep;
        idx += format[i];
    }

    output += input.substr(idx);

    return output;
}


function validateID() {
    // console.log(Client_ID.replace(/-/g, "").split('').every(isNumber))
    if(renewalType == "individual"){
        if(Client_ID.replace(/-/g, "").split('').every(isNumber) == false || Client_ID.length != 17)
        {
            showNotif = true;
            notifMessage = "Invalid Client ID"
            isNotifSucess = false
        }
    //    alert("Invalid Client ID")   
    }
    else{
        if(Business_ID.replace(/-/g, "").split('').every(isNumber) == false || Business_ID.length != 14)
        {
            showNotif = true;
            notifMessage = "Invalid Business ID"
            isNotifSucess = false
        }
        else if(Client_ID.replace(/-/g, "").split('').every(isNumber) == false || Client_ID.length != 17){
            showNotif = true;
            notifMessage = "Invalid Client ID"
            isNotifSucess = false

        }
}

   
}



function getCOC(COC) {


    SelectedCOC = InsuranceList.filter(x => x.id === COC).map(x => x.text)
    COCModal=true
     console.log()
}


const requestAPI = async () => {

const response = await fetch('http://localhost:52361//api/ECOC_API',{
        method: "post",
        headers: {
            'Accept': 'application/json, text/plain',
            'Content-Type': 'application/x-www-form-urlencoded',
        },
        body: JSON.stringify({
           mv_file_no:MVFileNo,
           engine:EngineNo,
           chassis:ChassisNo,
           inception_date:InceptionDate,
           expiry_date:ExpiryDate,
           mv_type:SelectedMVType,
           premium_type:SelectedPremiumType,
           tax_type:SelectedTaxType,
           assured_name:AssuredName,
           assured_tin:AssuredTIN
            }),
        
    })
    const character = await response.json();
    console.log(character)
};


let showNotif = false;
let notifMessage = "Message";
let isNotifSucess = false;


 function closeNotif(){
    showNotif = false;
 }
</script>
<div class="box">
<div class="block">
    <p class="is-size-4">Upload</p>
</div>
        <div class="block">
            <div class="tabs">
                <ul>
            
                  <!-- svelte-ignore a11y-missing-attribute -->
                  <li class:is-active={tabOne}><a>Choose</a></li>
                  <!-- svelte-ignore a11y-missing-attribute -->
                  <li class:is-active={tabTwo}><a>Encode</a></li>
                  <!-- svelte-ignore a11y-missing-attribute -->
                  <li class:is-active={tabThree}><a>ECOC</a></li>
                  <li class:is-active={tabFour}><a>Upload</a></li>
            
                </ul>
              </div>
              {#if tabOne}
              <div class="">
                This vehicle is registered to:
                <br>
                <div class="control">
                    <!-- svelte-ignore a11y-label-has-associated-control -->
                    <label class="is-size-5 radio">
                          <input  type="radio" bind:group={renewalType} value="individual">
                          Individual
                    </label>
                    <br />
                    <label class="is-size-5 radio">
                          <input type="radio" bind:group={renewalType} value="company">
                          Company
                    </label>
                </div>
            </div>
            <br>
              <div class="tab1">
            
              <div class="block">
                  <p>Please prepare the ff. requirements:</p>
                    <div class="block mt-3 req">
                        <ul class="is-size-7">
                            {#if renewalType=="individual"}
                                <li>OR & CR</li>
                                <li>MVIR</li>
                                <li>Emission Test</li>
                                <li>CTPL</li>
                                <li>T.I.N.</li>
                            {/if}
                            {#if renewalType=="company"}
                                <li>OR & CR</li>
                                <li>MVIR</li>
                                <li>Emission Test</li>
                                <li>CTPL</li>
                                <li>ID of authorized company representative</li>
                                <li>T.I.N.</li>
                            {/if}
                            </ul>
                    </div>

              </div>
            </div>
              <div class="block has-text-right">
                  <button on:click={()=>{falseEverything();tabTwo=true}} class="button is-info" disabled={renewalType==''?true:false}>Next</button>
              </div>
            
              {/if}
            
              {#if tabTwo}
                
                <div class="block">
                    <p class="is-size-5 is-capitalized">
                        {renewalType}
                    </p>
                </div>
                <div class="block">
                   
              
                    {#if renewalType=="individual"}
                    <div class="block">
                        <p class="is-size-7">Client ID</p>
                        <input type="text" maxlength="17" on:input|preventDefault={handleInput} class="input" bind:value={Client_ID}>
                    </div>
                {/if}
                {#if renewalType=="company"}
                <div class="block">
                    <p class="is-size-7">Business ID</p>
                    <input type="text" maxlength="14" class="input" on:input|preventDefault={handleInput} bind:value={Business_ID}>
                </div>
                <div class="block">
                    <p class="is-size-7">Client ID</p>
                    <input type="text" maxlength="17" on:input|preventDefault={handleInput} class="input" bind:value={Client_ID}>
                </div>
                {/if}
                    <div class="block">
                        <p class="is-size-7">Customer Name</p>
                        <input type="text"  on:mousedown={()=>validateID()} class="input" bind:value={customerName}>
                    </div>
                        <div class="columns">
                            <div class="column is-half">
                                <p class="is-size-7">Plate Number</p>
                                <input bind:value={plateNo}  on:mousedown={()=>validateID()} type="text" class="input" on:keyup={()=>plateNo=plateNo.toUpperCase()}>
                            </div>
                        </div>                       
                </div>
                <div class="btnPrev block has-text-right">
                    <button on:click={()=>{falseEverything();tabOne=true}} class="button">Prev</button>
                    <button on:click={()=>{falseEverything();tabFour=true}} class="button" disabled={plateNo=='' || customerName==''?true:false}>Next</button>
                </div>
              {/if}
            
              {#if tabThree}
                <label for="">Select Insurance</label>
                <div class="block">                 
                  <div class="select mb-4">
               
                      <select bind:value={SelectedCOC} on:change={()=>{
                      getCOC(SelectedCOC)        
                      }}>
                        <!-- <option disabled selected value> -- select an option -- </option> -->
                          {#each InsuranceList as item}
                              <option value="{item.id}">{item.text}</option>
                          {/each}
          
                      </select>
                  </div>
              </div>           
              {#if SelectedCOC == 1}
              <label for="" style="display:block">Select MV Type</label>
              <div class="select mb-2">
                <select bind:value={SelectedMVType}>
                    <!-- <option value="Default" selected>Select Insurance</option> -->         
                    {#each MVTypeList as item}
                        <option value="{item.id}">{item.text}</option>
                    {/each}
                </select>
            </div>
            <label for="" style="display:block">Premium</label>
              <div class="select">
                <select bind:value={SelectedVClassification}>
                    <!-- <option value="Default" selected>Select Insurance</option> -->         
                    {#each VClassificationList as item}
                        <option value="{item.id}">{item.text}</option>
                    {/each}
                </select>
            </div>
            {/if}
              {#if SelectedCOC == 2}
              <div class="block">
                <p class="is-size-7">COC Number</p>
                <input type="text" class="input" bind:value={customerName}>               
            </div>
            <div class="block">
                <p class="is-size-7">COC Number</p>
                <input type="text" class="input" bind:value={customerName}>               
            </div>
              {/if}
                <!-- <div class="block has-text-right">
                  <button on:click={()=>{falseEverything();tabFour=true}} class="button" disabled={plateNo=='' || customerName==''?true:false}>Next</button>
              </div> -->
              {/if}
              
              {#if tabFour}
              <div class="block">
                 Upload Documents Here
              </div>
              <div class="block">
                  <Dropzone multiple=true on:drop={handleFilesSelect} />
                  <ol class="is-size-7">
                  {#each files.accepted as item}
                      <li>{item.name}</li>
                  {/each}
                  </ol>
              </div>
              <div class="block has-text-right">
                  <button class="button is-primary" on:click={uploadFiles}>Upload</button>
              </div>
            {/if}   
        </div>
    </div>
    
<div class="modal" class:is-active={COCModal}>
    <div class="modal-background"></div>
    <div class="modal-card">
        <header class="modal-card-head">
            <p class="modal-card-title">E-CPTL</p>
            <button class="delete" aria-label="close" on:click={()=>{
                COCModal=false
                }}></button>
        </header>
        <section class="modal-card-body">
            <div class="block">
                <p class="is-size-3">{SelectedCOC}</p>

            </div>

            <div class="columns">
                <div class="column">
                    <div class="">
                        <p class="is-size-7">Engine Number</p>
                        <input type="text" bind:value={EngineNo} class="input">               
                    </div>                 
                </div>       
                <div class="column">
                    <div class="">
                        <p class="is-size-7">Chassis Number</p>
                        <input type="text" bind:value={ChassisNo} class="input">               
                    </div>
                </div>       
            </div>

            <div class="columns">
                <div class="column">
                    <div class="">
                        <p class="is-size-7">MV File Number</p>
                        <input type="text" bind:value={MVFileNo} class="input">               
                    </div>                 
                </div>       
                <div class="column">
                    <div class="">
                        <p class="is-size-7">MV File Number</p>
                        <input type="text" bind:value={MVFileNo} class="input">               
                    </div>                 
                </div>       
                <div class="column">
                    <div class="">
                        <p class="is-size-7">MV File Number</p>
                        <input type="text" bind:value={MVFileNo} class="input">               
                    </div>                 
                </div>       
                <div class="column">
                    <div class="">
                        <p class="is-size-7">MV File Number</p>
                        <input type="text" bind:value={MVFileNo} class="input">               
                    </div>                 
                </div>       
                <div class="column">
                    <div class="">
                        <p class="is-size-7">MV File Number</p>
                        <input type="text" bind:value={MVFileNo} class="input">               
                    </div>                 
                </div>       
                <div class="column">
                    <div class="">
                        <p class="is-size-7">MV File Number</p>
                        <input type="text" bind:value={MVFileNo} class="input">               
                    </div>                 
                </div>       
                <div class="column">
                    <div class="">
                        <p class="is-size-7">Assured Name</p>
                        <input type="text" bind:value={AssuredName} class="input">               
                    </div>
                </div>       
                <div class="column">
                    <div class="">
                        <p class="is-size-7">Assured TIN</p>
                        <input type="text" bind:value={AssuredTIN} class="input">               
                    </div>
                </div>    
            </div>

         
            <div class="columns ">
                <div class="column">
                    <div class="block">
                        <label for="" style="display:block">Select MV Type</label>
                        <div class="select">
                          <select bind:value={SelectedMVType}>
                              <!-- <option value="Default" selected>Select Insurance</option> -->         
                              {#each MVTypeList as item}
                                  <option value="{item.id}">{item.text}</option>
                              {/each}
                          </select>
                      </div>
                    </div>
                </div>
                <div class="column">
                    <div class="block">
                        <label for="" style="display:block">Select Premium Type</label>
                        <div class="select">
                          <select bind:value={SelectedPremiumType}>
                              <!-- <option value="Default" selected>Select Insurance</option> -->         
                              {#each PremiumTypeList as item}
                                  <option value="{item.id}">{item.text}</option>
                              {/each}
                          </select>
                      </div>
                    </div>      
                </div>
                <div class="column">
                    <div class="block">
                        <label for="" style="display:block">Select Tax Type</label>
                        <div class="select">
                          <select bind:value={SelectedTaxType}>
                              <!-- <option value="Default" selected>Select Insurance</option> -->         
                              {#each  TaxTypeList as item}
                                  <option value="{item.id}">{item.text}</option>
                              {/each}
                          </select>
                      </div>
                    </div>
                </div>
              </div>

              <div class="columns">
                <div class="column is-4">
                    <label  style="display:block" for="dateofbirth">Inception Date</label>
                    <input bind:value={InceptionDate} class="calendar is-size-5" type="date" name="dateofbirth" id="calen">
                </div>
                <div class="column is-4">
                    <label  style="display:block" for="dateofbirth">Expiry Date</label>
                    <input bind:value={ExpiryDate} class="calendar is-size-5" type="date" name="dateofbirth" id="calen">
                </div>
            </div>


        </section>
        <footer class="modal-card-foot">
             <button class="button is-link" on:click={()=>{
                // requestAPI();
                falseEverything();
                tabFour=true;
             }}>Next</button>
             
             <!-- <button class="button" on:click={()=>{
                COCModal=false;
             }}></button> -->


        </footer>
    </div>
</div>


{#if showNotif}
<Notif {isNotifSucess} {notifMessage}>
    <button class="delete" on:click={()=>closeNotif()} button>
</Notif>
{/if}


<style>
   .req li{
        font-size: 15px;
    }
    .tab1{
        display: flex;
        flex-direction: column;
        align-items: center;
        align-content: center;
    }
    /* .select select:not([multiple]){
      padding-right: 17.5em;
    } */

    [type="date"] {
  background:#fff url(https://cdn1.iconfinder.com/data/icons/cc_mono_icon_set/blacks/16x16/calendar_2.png)  97% 50% no-repeat ;
}
[type="date"]::-webkit-inner-spin-button {
  display: none;
}
[type="date"]::-webkit-calendar-picker-indicator {
  opacity: 0;
}
#calen {
  border: 1px solid #c4c4c4;
  border-radius: 5px;
  background-color: #fff;
  padding: 3px 5px;
  box-shadow: inset 0 3px 6px rgba(0,0,0,0.1);
  width: 190px;
}


.modal-card, .modal-content {
    width: 666px;
}

.modal-card-foot, .modal-card-head{
    justify-content: right;
}

.btnPrev{
    display: flex;
    justify-content: space-between;
}
</style>