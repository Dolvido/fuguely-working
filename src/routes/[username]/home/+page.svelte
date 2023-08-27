<script lang="ts">
    // ...[rest of your imports]...
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

    function selectUserType(e) {
        userType = e.target.value;

        // Confirm user type and execute related logic
        confirmUserType();
    }

    async function confirmUserType() {
        console.log("confirming user type", userType);

        const batch = writeBatch(db);

        batch.set(doc(db, "users", $user!.uid), { 
            userType, 
        }, { merge: true });

        await batch.commit();

        console.log("userData", $userData);

        // You might need to define `href` to where you want to redirect.
        const href = `/${$userData?.username}/edit`; 
        location.href = href;
    }
</script>

<main class="max-w-xl mx-auto">
    {#if $userData?.username == $page.params.username}
        <h1 class="mx-2 text-2xl font-bold mt-8 mb-4 text-center">Edit your Profile</h1>
        
        <div class="my-4">
            <label class="mr-4">I am a:</label>
            <select bind:value={userType} on:change={selectUserType} class="select select-bordered w-max">
                <option value="">Select...</option>
                <option value="teacher">Teacher</option>
                <option value="student">Student</option>
            </select>
        </div>
        <!-- ...[rest of your structure]... -->

    {/if}
</main>
