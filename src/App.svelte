<script>
import Navbar from './Navbar.svelte'
import Wallet from './Wallet.svelte'
import Uploader from './Uploader.svelte'
import Uploaded from './Uploaded.svelte'
import ApprovedViewer from './ApprovedViewer.svelte'
import Login from './Login.svelte'

const { createClient } = supabase

const SUPABASE_URL='https://rpojzmwqlwoltxvmtjuw.supabase.co'
const SUPABASE_ANON_KEY='eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InJwb2p6bXdxbHdvbHR4dm10anV3Iiwicm9sZSI6ImFub24iLCJpYXQiOjE2NjAxNDI4MDksImV4cCI6MTk3NTcxODgwOX0.6N204aZakp2x0M2NWxBBYNtgg931CTSdxcOrxRbgC88'

supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY)

let branchName = 'databridge-test'
let serviceType = 'selfservice'
let user = null

async function getBranchAndServiceType(email) {
	console.log(email)

	const { data, error } = await supabase 
	.from('usermap')
	.select()
	.eq('email', email)

	if(error)
		alert('Error! '+error.message)

	if(data){
		if(data==null){
			alert('Not assigned')
			user = null 
			return
		}

		else {
			
			branchName = data[0].branch
			serviceType = data[0].servicetype
			user = supabase.auth.getUser()
		}

	}
}
async function getCurrentUserEmail() {
const { data: { session },} = await supabase.auth.getSession()
const { user } = session
return user.email
}



supabase.auth.onAuthStateChange((event, session) => {
	if(user==null){
		//getBranchAndServiceType(supabase.auth.user().email)
		//getBranchAndServiceType(supabase.auth.getUser().email)
		getBranchAndServiceType(session.user.email)
	
		//console.log(session.user.email)

	}
		
	else	
	user=session

})

async function logOut() {
	const { error } =await supabase.auth.signOut()
}
</script>

{#if user==null}
	<Login />
{:else}
<div class="block has-text-right">
	<Navbar branchName={branchName}>
		<button id="logoutButton" class="button is-rounded" on:click={logOut}>Log Out</button>
		
	</Navbar>

</div>

{#if serviceType=='selfservice'}
	<div class="columns">
		<Wallet branchName={branchName}/>
	</div>
{/if}

<div class="columns" style="margin-top:50px;">

	
	<div class="column is-5 mx-2">

		<ApprovedViewer branchName = {branchName} serviceType={serviceType}/>
	</div>
	<div class="column is-4 ">
		<Uploader branchName = {branchName} serviceType = {serviceType}/>
</div>
	<div class=" c3 column mx-2 ">	
			<Uploaded branchName = {branchName} />
	</div>
</div>

{/if}
<style>
#logoutButton{

border: outset;
padding-right: 14px;
/* font-weight: 700; */
}
#logoutButton:hover{
background-color: #3232ee24;
color:aliceblue;
border: inset;
/* font-weight: 700; */
}
.c3{
	overflow-wrap: anywhere
}
</style>
