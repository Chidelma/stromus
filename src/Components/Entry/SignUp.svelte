<script lang="ts">
    import { admin, cognito, dynamo } from '../../Scripts/Init';
    import type { _admin, _part_sort } from '../../Scripts/Interface';
    import Code from './Code.svelte';
    import { v5 as uuidv5 } from 'uuid';

    let showCode:boolean = false;

    let new_user:_admin = {
        id: '',
        first_name: '',
        last_name: '',
        birthday: '',
        address: '',
        city: '',
        state: '',
        country: '',
        code: '',
        email: '',
        photo: ''
    }

    let password:string = '';
    let retyped_password:string = '';

    async function adminUser() {

        if(password == retyped_password) {
 
            new_user.id = uuidv5(new_user.email, uuidv5.URL);

            let key_value:_part_sort = { 
                part_key: 'last_name', 
                part_value: new_user.last_name, 
                sort_key: 'id', 
                sort_value: new_user.id
            }

            let result:any = await $dynamo.getItem('ADMINS', key_value);

            if(result == undefined) {

                new_user.photo = 'https://api.hello-avatar.com/adorables/' + new_user.id;

                await $admin.set_admin(new_user);

                let added:boolean = await $cognito.signUp(password, $admin);

                if(added) {
                    showCode = true;
                }
            }
        }
    }
</script>

<div id="signUp"> 
    {#if showCode}
        <Code/>
    {:else}
        <input class="signup left-half" placeholder="First Name" bind:value="{new_user.first_name}" required />
        <input class="signup right-half" placeholder="Last Name" bind:value="{new_user.last_name}" required />

        <input class="signup" type="date" placeholder="birthday" bind:value="{new_user.birthday}" />
        <input class="signup" placeholder="Address" bind:value="{new_user.address}" />
        <input class="signup left-half" placeholder="City" bind:value="{new_user.city}"/>
        <input class="signup right-half" placeholder="State/province" bind:value="{new_user.state}" />
        <input class="signup left-half" placeholder="Country" bind:value="{new_user.country}" />
        <input class="signup right-half" placeholder="Postal/Zip Code" bind:value="{new_user.code}" />

        <hr/>

        <input class="signup" type="email" placeholder="Email" bind:value="{new_user.email}" required />
        <input class="signup left-half" type="password" placeholder="Password" bind:value="{password}" required />
        <input class="signup right-half" type="password" placeholder="Retype Password" bind:value="{retyped_password}" required />

        
        <button class="btn btn-info" type="submit" on:click="{adminUser}">Sign Up</button>
    {/if}
</div>


<style>
    hr {
        margin-bottom:20px;
    }

    #signUp {
        margin: 0 auto;
        width:500px;
        padding:10px;
    }

    .signup {
        margin-bottom:20px;
        border:none;
        outline: none;
        border-radius: 0.2rem;
        background-color: #350d22;
        color:white;
        width:100%;
    }

    .left-half {
        width:48%;
        float:left;
    }

    .right-half {
        width:48%;
        float:right;
    }

    .btn {
        width:50%;
        margin-left:25%;
    }
</style>