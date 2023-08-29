<script lang="ts">
    import { onMount } from 'svelte';
    import { db, user } from "$lib/firebase";  // Assuming you've initialized firebase in this module
    import { doc, getDoc } from "firebase/firestore";
    
    let availabilityData = {};

    onMount(async () => {
        const docRef = doc(db, 'users', $user!.uid);  // Using the authenticated user's uid as the document ID
        const docData = await getDoc(docRef);

        if (docData.exists()) {
            availabilityData = docData.data() || {};
        } else {
            console.log("No such document!");
        }
    });
</script>

<div>
    <h1>Your Availability</h1>
    {#if Object.keys(availabilityData).length > 0}
        <ul>
            {#each Object.keys(availabilityData) as day}
                <li>
                    <strong>{day}:</strong> {#each availabilityData[day] as timeWindow}<span>{timeWindow}</span>{/each}
                </li>
            {/each}
        </ul>
    {:else}
        <p>No availability data found.</p>
    {/if}
</div>