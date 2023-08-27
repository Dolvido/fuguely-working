<script lang="ts">
    import { page } from "$app/stores";
    import AuthCheck from "$lib/components/AuthCheck.svelte";
    import SortableList from "$lib/components/SortableList.svelte";
    import UserLink from "$lib/components/UserLink.svelte";
    import { db, userData, user } from "$lib/firebase";
    import {
      doc,
      getDoc,
      writeBatch,
    } from "firebase/firestore";
    import { writable } from "svelte/store";
    
    let userType = "";
    function handleSubmit() {
        if (userType === "") {
            alert("Please select a user type!");
            return;
        }
        
        alert(`You are a ${userType}`);
    }
    
</script>
<AuthCheck>
<main class="max-w-xl mx-auto">
    {#if $userData?.username == $page.params.username}
    <h1 class="mx-2 text-2xl font-bold mt-8 mb-4 text-center">Edit your Profile</h1>

    <form on:submit|preventDefault={handleSubmit}>
        <label class="mr-4">
            I am a:
            <select bind:value={userType} class="select select-bordered w-max">
                <option value="">Select...</option>
                <option value="teacher">Teacher</option>
                <option value="student">Student</option>
            </select>
        </label>
        
        <button type="submit" class="btn btn-primary mt-4">Submit</button>
    </form>
    
    <!-- ...[rest of your structure]... -->
{/if}

</main>
</AuthCheck>
