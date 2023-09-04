<script>
    import { onMount } from "svelte";
    import { collection, getDocs, query, where } from "firebase/firestore"; 
    import { db } from "$lib/firebase"; // Import your Firestore instance
  
    let username = "";
    let teacherFound = false;
  
    // Function to search for a teacher based on the username
    const searchTeacher = async (username) => {
      const q = query(
        collection(db, "teachers"),
        where("username", "==", username)
      );
  
      const querySnapshot = await getDocs(q);
      teacherFound = !querySnapshot.empty;
    };
  
    // Watch for changes in the username and trigger a search
    $: if (username) {
      searchTeacher(username);
    }
  </script>
  
  <input type="text" bind:value={username} placeholder="Enter username" />
  
  {#if username}
    {#if teacherFound}
      <p>Teacher found</p>
    {:else}
      <p>Teacher not found</p>
    {/if}
  {/if}
  