<script>
    export let amtDue = 0
    export let branchName 
    let balance = 0
    let newBalance = 0
    let containerArray = []
    
async function updateRecordStatus() {
    const { data, error } = await supabase 
        .from('uploads')
            .update({
                status: 'pay-request'
            })  
            .match({
                branch: branchName, 
                status: 'approved'
            })

        if(error)
            alert("Error! " + error.message)    
}


async function updateBalance() {
    const { data, error } = await supabase
        .from('wallet_balance')
        .update({
            balance: newBalance
        })  
        .eq('branch', branchName)

        if(error)
            alert('Error! ' + error.message)

        if(data) {
            updateRecordStatus()
            alert('Payment request successful!')
        }  
        
}

async function retrieveCurrentBalance() {
    const { data, error } = await supabase
        .from('wallet_balance')
        .select()
        .eq('branch', branchName)
        
    if(error)
        alert('Error! ' + error.message)
    
    if(data) {
        balance=data[0].balance
        if(parseFloat(balance)<parseFloat(amtDue))
            alert("Not enough balance.  Please top up your wallet.")
        else {
                //subtrack amtDue from currentBalance 
                newBalance = parseFloat(balance) - parseFloat(amtDue)
                updateBalance()
        }
        
            
    }

}    
</script>
<button class="button is-fullwidth is-primary" on:click={retrieveCurrentBalance}>Process Payment</button>