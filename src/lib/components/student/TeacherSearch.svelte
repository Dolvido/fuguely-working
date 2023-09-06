<script>
  import { writable } from 'svelte/store';
  import { collection, getDocs, query, where, doc, updateDoc, arrayUnion } from "firebase/firestore";
  import { db } from "$lib/firebase"; // Import your Firestore instance
  import { studentData } from "$lib/firebase"; // Import your Svelte store

  let username = "";
  let suggestions = [];
  let isExactMatch = false;
  let studentId = null;
 // Assuming username serves as the ID

  // Function to search for teachers based on the username
  const searchTeacher = async (username) => {
    // Reset states
    suggestions = [];
    isExactMatch = false;

    const q = query(
      collection(db, "teachers"),
      where("username", ">=", username),
      where("username", "<=", username + "\uf8ff")
    );

    const querySnapshot = await getDocs(q);
    const newSuggestions = [];
    querySnapshot.forEach((doc) => {
      newSuggestions.push(doc.data().username);
    });

    // Check for an exact match and update states
    if (newSuggestions.length === 1 && newSuggestions[0] === username) {
      suggestions = [];
      isExactMatch = true;
    } else {
      suggestions = newSuggestions;
    }
  };

  // Function to add the teacher to the student's teachers array in Firestore
  const addTeacherToStudent = async () => {
    console.log("Adding teacher to student");
    console.log("studentId", studentId);
    console.log("username", username);
    const studentRef = doc(db, "students", studentId);
    await updateDoc(studentRef, {
      teachers: arrayUnion(username)
    });

    // Update the studentData Svelte store
    studentData.update(data => {
      data.teachers = [...data.teachers, username];
      return data;
    });
  };

  // Watch for changes in the username
  $: if (username) {
    searchTeacher(username);
  }
  $: if ($studentData) {
    studentId = $studentData.uid;
  }
</script>

<div class="relative flex items-center">
  <input type="text" bind:value={username} placeholder="Enter username" />
  {#if isExactMatch}
    <button class="ml-2 bg-green-500 text-white px-4 py-2 rounded" on:click={addTeacherToStudent}>Add</button>
  {/if}
  {#if suggestions.length > 0}
    <div class="absolute z-10 mt-2 w-full bg-white border border-gray-300 rounded">
      {#each suggestions as suggestion}
        <div 
          class="p-2 hover:bg-gray-200 cursor-pointer"
          on:click={() => { username = suggestion; suggestions = []; }}
        >
          {suggestion}
        </div>
      {/each}
    </div>
  {/if}
</div>
