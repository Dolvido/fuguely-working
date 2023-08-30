<script lang='ts'>
    import { onMount } from "svelte";
    import { doc, getDoc, updateDoc } from "firebase/firestore";
    import { db, user, userData } from "$lib/firebase";
    let availabilities = [];
    let availabilitiesList = [];
    let loading = true;
  
    onMount(async () => {
        try {
            const userDoc = await getDoc(doc(db, 'users', $user!.uid)); // Replace 'yourUserIdHere' with the actual user ID
            if (userDoc.exists()) {
                availabilities = userDoc.data().availabilities;
            } else {
                console.log("User document doesn't exist.");
            }
            loading = false;
        } catch (error) {
            console.error("Error fetching availabilities:", error);
        }
        });
  
    $: if (!loading) {
    availabilitiesList = Object.entries(availabilities).map(([day, time]) => {
        return { day, time };
    });
    }

    // Function to delete an availability
    async function deleteAvailability(day: string) {
      // Your logic to delete the availability from Firestore
      const userDocRef = doc(db, 'users', $user!.uid);
      const userDoc = await getDoc(userDocRef);
      if (userDoc.exists()) {
        const userData = userDoc.data();
        if (userData && userData.availabilities && userData.availabilities[day]) {
          // Remove the availability
          delete userData.availabilities[day];
          // Update the document
          await updateDoc(userDocRef, { availabilities: userData.availabilities });
        }
      }
    }


  </script>
  
  {#if $userData && $userData.availabilities}
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {#each Object.entries($userData.availabilities) as [day, time]}
      <div class="p-6 rounded-lg shadow-lg bg-white">
        <h3 class="text-lg font-semibold mb-2">{day}</h3>
        <ul class="list-decimal list-inside">
          <li class="text-sm text-gray-700">{time}</li>
        </ul>
        <button class="mt-2 bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded" on:click={() => deleteAvailability(day)}>
          Delete
        </button>
      </div>
    {/each}
  </div>
{:else}
  <p class="text-lg font-medium text-gray-600">Loading...</p>
{/if}



  