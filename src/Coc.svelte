<script>
    import { createEventDispatcher, onDestroy } from 'svelte'
    
    const dispatch = createEventDispatcher()
    const close = () => dispatch('close') //call this if you wanna close the modal
    const url = "https://coc-worker.dbasia.workers.dev"   //middleman API that I did.  
                                                        //You can probably directly call Jona's API if/when
                                                        //the source is transferred to the databridge server
    let registrationType = 'R'
    let taxTypeID = '1'
    let dealerName = "LTO Mobile Pilot"
    let dealerBranch = "Mobile 1"
    let vehicleClassificationId, vehicleCode, mvFileNumber, yearOfMake
    let make, model, variant, bodyType, color, chassisNumber, engineNumber
    let authCapacity, grossVehicleWeight 
    let firstName, middleName, lastName, plateNumber, companyName 
    let tin, address 
    let cocLink = '#'
    let hiddenProgress = true 
    let hiddenAnchor = true 

    async function saveToCocTable() {
        const { data, error } = await supabase
        .from('ecoc')
        .insert([{
            fullname: `${firstName} ${middleName} ${lastName}`,
            plate_number: plateNumber,
            issuer: dealerName
        }])

        if(error)
            alert("error saving to eCOC database! " + error.message)
    }

    function showPostData() {

    hiddenProgress = false 

    let postData = {
    "RegistrationType" : registrationType,
    "TaxTypeID": taxTypeID,
    "Dealer" : {
        "DealerName" : dealerName,
        "DealerBranch" : dealerBranch
    },
    "Vehicle" : {
        "VehicleClassificationID": vehicleClassificationId,
        "PlateNumber" : plateNumber,
        "MVFileNumber" : mvFileNumber,
        "YearOfMake" : yearOfMake,
        "Make" : make,
        "Model" : model,
        "Variant" : variant,
        "BodyType" : bodyType,
        "Color" : color,
        "ChassisNumber" : chassisNumber,
         "VehicleCode" : vehicleCode,
        "EngineNumber" : engineNumber,
        "AuthCapacity" : authCapacity,
        "GrossVehicleWeight" : grossVehicleWeight
    },
    "Customer":{
        "FirstName" : firstName,
        "MiddleName" : middleName,
        "LastName" : lastName,
         "Address" : address,
        "CompanyName" : companyName,
        "TIN": tin
        }
    }

    postTheData(JSON.stringify(postData, null, 2))

}
  
async function postTheData(jsonData) {

    if(hiddenAnchor==false) {   //check if we've already generated an eCOC.  Don't generate again!
        alert("eCOC already generated.")
        hiddenProgress = true 
         hiddenAnchor = false 
        return
    }

    axios({
      method: 'post',
      url: url,
      data: jsonData,
      headers: {'Content-Type': 'application/json', 'Authentication-hash' : '8447ed86f9a24921af2d17f0f91c535c'}
  })
    .then((response)=> {
      
    hiddenProgress = true 
    hiddenAnchor = false 
    //console.log(response.data.Status)
    if(response.data.Status=='Success'){
        convertToBlob(response.data.Message)
        //saveToCocTable()  if you wanna use this (save eCOC transactions), create the table in supabase first.
    }
        
    else{
        alert("Error! " + response.data.Message)
        hiddenAnchor = true
        hiddenProgress = true
    }
            
    })
}

async function convertToBlob(msg) {
  const resp = await fetch('data:application/pdf;base64,' + msg)
  const blb = await resp.blob()
  blb.lastModifiedDate = new Date()
  blb.name = 'test.pdf'
  console.log(blb)
  uploadToBucket(blb) //upload to bucket
  let blbUrl = URL.createObjectURL(blb)
  cocLink = blbUrl

}

async function uploadToBucket(file){
    const { data, error } = await supabase
    .storage
    .from(plateNumber.replaceAll(' ',''))
    .upload(`${plateNumber.replaceAll(' ','')}/Paramount-eCOC.pdf`,file)

    if(error) {
        hiddenAnchor = true 
        alert('eCOC upload error!' + error.message)
    }
        
}

</script>


<div class="modal is-active">
    <div class="modal-background"></div>
    <div class="modal-card">
        <header class="modal-card-head has-background-info">
            <p class="modal-card-title has-text-light">Paramount eCOC</p>
            <button class="delete" aria-label="close" on:click={close}></button>
        </header>    
        <section class="modal-card-body has-background-info-light">
            <!-- Content goes here -->
            <div class="block">
                <span class="is-size-6">Customer Info</span>
            </div>
            
            <div class="columns">
                <div class="column is-5">
                    <p class="is-size-7">Last Name</p>
                    <input type="text" placeholder="Last Name" class="input" bind:value={lastName}>
                </div>
                <div class="column is-5">
                    <p class="is-size-7">First Name</p>
                    <input type="text" placeholder="First Name" class="input" bind:value={firstName}>
                </div>
                <div class="column is-2">
                    <p class="is-size-7">MI</p>
                    <input type="text" placeholder="M.I." class="input" bind:value={middleName}>
                </div>
            </div>

            <div class="columns">
                <div class="column">
                    <p class="is-size-7">Address</p>
                    <input type="text" placeholder="Address" class="input" bind:value={address}>
                </div>
            </div>

            <div class="columns">
                <div class="column is-6">
                    <p class="is-size-7">Company Name</p>
                    <input type="text" placeholder="Company" class="input" bind:value={companyName}>
                </div>
                <div class="column is-6">
                    <p class="is-size-7">TIN Number</p>
                    <input type="text" placeholder="TIN: XXX-XXX-XXX-XXX" class="input" bind:value={tin}>
                </div>
            </div>
            
            <div class="block">
                <span class="is-size-6">Vehicle Info</span>
            </div>
            
            <div class="columns">
                <div class="column is-half">
                    <p class="is-size-7">Vehicle Classification</p>
                    <div class="select">
                        <select bind:value={vehicleClassificationId}>
                            <option value="0">Vehicle Classification</option>
                            <option value="1">Private Car</option>
                            <option value="2">Light/Medium Trucks</option>
                            <option value="3">Heavy Trucks & Private Buses</option>
                            <option value="4">AC and Tourist Cars</option>
                            <option value="5">Taxi, PUJ and Mini Bus</option>
                            <option value="6">PUB and Tourist Bus</option>
                            <option value="7">Motorcycles/Tricycles/Trailers</option>
                        </select>
                    </div>
                </div>
                <div class="column is-half">
                    <p class="is-size-7">Vehicle Code</p>
                    <div class="select">
                        <select bind:value={vehicleCode}>
                            <option value="0">Vehicle Code</option>
                            <option value="C">Car - C</option>
                            <option value="HB">Shuttle Bus - HB</option>
                            <option value="LE">Light Electric Vehicle - LE</option>
                            <option value="M">Motorcycle without Side Car - M</option>
                            <option value="MO">Mopeds - MO</option>
                            <option value="MS">Motorcycle with Side Car - MS</option>
                            <option value="NC">Non Conventional MC (Car) - NC</option>
                            <option value="NV">Non Conventional MV (UV) - NV</option>
                            <option value="OB">Tourist Bus - OB</option>
                            <option value="SB">School Bus - SB</option>
                            <option value="SV">Sports Utility Vehicle - SV</option>
                            <option value="UV">Utility Vehicle - UV</option>
                            <option value="TB">Truck Bus - TB</option>
                            <option value="TC">Tricycle - TC</option>
                            <option value="TK">Truck - TK</option>
                            <option value="TL">Trailer - TL</option>
                        </select>
                    </div>
                </div>
            </div>

            <div class="columns">
                <div class="column">
                    <p class="is-size-7">Make</p>
                    <input type="text" placeholder="Make" class="input" bind:value={make}>
                </div>
                <div class="column">
                    <p class="is-size-7">Model</p>
                    <input type="text" placeholder="Model" class="input" bind:value={model}>
                </div>
            </div>

            <div class="columns">
                <div class="column is-one-third">
                    <p class="is-size-7">Variant</p>
                    <input type="text" placeholder="Variant" class="input" bind:value={variant}>
                </div>
                <div class="column is-one-third">
                    <p class="is-size-7">Body Type</p>
                    <input type="text" placeholder="Body Type" class="input" bind:value={bodyType}>
                </div>
                <div class="column is-one-third">
                    <p class="is-size-7">Color</p>
                    <input type="text" placeholder="Color" class="input" bind:value={color}>
                </div>
            </div>

            <div class="columns">
                <div class="column is-one-third">
                    <p class="is-size-7">Plate Number</p>
                    <input type="text" placeholder="Plate No" class="input" bind:value={plateNumber}>
                </div>
                <div class="column is-one-third">
                    <p class="is-size-7">MV File No.</p>
                    <input type="text" placeholder="MV File No" class="input" bind:value={mvFileNumber}>
                </div>
                <div class="column is-one-third">
                    <p class="is-size-7">Year of Make</p>
                    <input type="text" placeholder="Year of Make" class="input" bind:value={yearOfMake}>
                </div>
            </div>

            <div class="columns">
                <div class="column is-half">
                    <p class="is-size-7">Engine Number</p>
                    <input type="text" placeholder="Engine No" class="input" bind:value={engineNumber}>
                </div>
                <div class="column">
                    <p class="is-size-7">Chassis Number</p>
                    <input type="text" placeholder="Chassis No" class="input" bind:value={chassisNumber}>
                </div>
                
            </div>

            <div class="columns">
                <div class="column">
                    <p class="is-size-7">Capacity</p>
                    <input type="text" placeholder="Capacity" class="input" bind:value={authCapacity}>
                </div>
                <div class="column">
                    <p class="is-size-7">Vehicle Weight</p>
                    <input type="text" placeholder="Gross Vehicle Weight" class="input" bind:value={grossVehicleWeight}>
                </div>
            </div>
            
            <div class="block">
                <span class="is-size-6">
                    Dealer Info
                </span>
            </div>

            <div class="columns">
                <div class="column is-half">
                    <p class="is-size-7">Dealer Name</p>
                    <input type="text" placeholder="Dealer Name" class="input" bind:value={dealerName}>
                </div>
                <div class="column is-half">
                    <p class="is-size-7">Dealer Branch</p>
                    <input type="text" placeholder="Dealer Branch" class="input" bind:value={dealerBranch}>
                </div>
            </div>

            <div class="block has-text-right">
                <button class:is-hidden={!hiddenProgress} class="button has-background-primary has-text-light" on:click={showPostData}>Generate eCOC</button>
            </div>
        </section>
        <footer class="modal-card-foot">
            <progress class:is-hidden={hiddenProgress} class="progress is-primary" max="100"></progress>
            <span class:is-hidden={hiddenAnchor} class="is-size-6">Your eCOC has been uploaded. <a class="is-size-6" href={cocLink} target="_blank">Click here</a> to view.</span>
            
        </footer>
        
    </div>
</div>