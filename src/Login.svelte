<script>

let userEmail = ''
let passWord = ''

async function loginUser(){
    const { user, session, error } = await supabase.auth.signInWithPassword({
    email: userEmail,
    password: passWord
})
    if(error)
        alert("Error! "+error.message)
    
}



// const requestAPI = async () => {

//     const response = await fetch('http://localhost:52361//api/ECOC_API',{
//             method: "post",
//             headers: {
// 				'Accept': 'application/json, text/plain',
//                 'Content-Type': 'application/x-www-form-urlencoded',
//             },
// 			body: JSON.stringify({
// 					value:"hey"
// 				}),
            
//         })
//         const character = await response.json();
//         console.log(character)
// };

// requestAPI();




</script>

<style>
    p {
        font-family: 'Source Sans Pro', sans-serif;
        font-size:300px;
       
    }
    #loginButton{

        border: outset;
        /* font-weight: 700; */
    }
    #loginButton:hover{
        background-color: #3232ee24;
        color:aliceblue;
        border: inset;
        /* font-weight: 700; */
    }
    .loginForm {
       margin-top: 100px;
    }
    .hero-body{
        font-weight: 700;
        height: 120vh;
        
    }
    .hero{
        background-color: #485fc7f7;
    color: #fff;
    }
</style>

<section class="hero is-link">
    <div class="hero-body">
        
 
            <div class="loginForm column is-2 is-offset-5">

                <div class="block has-text-centered">
                    <!-- <img src="logo.png" alt="logo" height="25%" width="auto"> -->
                    <p class="pt-1 is-size-3">User Login</p>
                </div>

                <div class="block">
                    <input bind:value={userEmail} type="text" class="input" placeholder="Email">
                </div>
                <div class="block">
                    <input bind:value={passWord} type="password" class="input" placeholder="Password">
                </div>
                <div class="block has-text-centered">
                    <button id="loginButton" class="button" on:click={loginUser}>Log In</button>
                </div>
            </div>


    </div>
  </section>


