<script>
    import { onMount } from "svelte";
    import { db, user } from "$lib/firebase";  // adjust the path as needed
    import { doc, getDoc } from "firebase/firestore";

    let availabilities = {};
    let teacherID = "some_teacher_id"; // Assume this is passed as a prop or fetched from somewhere

    onMount(async () => {
        // Fetch availabilities data for this teacher from Firestore
        const docRef = doc(db, "teachers", user.uid);
        const docSnap = await getDoc(docRef);
        
        if (docSnap.exists()) {
            availabilities = docSnap.data().availabilities || [];
        } else {
            console.log("No such document!");
        }
        console.log("availabilities", availabilities);
    });
</script>


  
  <style>
    /* Add your styles here */
    .lesson-card {
      border: 1px solid #ccc;
      padding: 16px;
      margin: 8px;
      border-radius: 4px;
    }
  </style>
  
  <div class="available-lessons">
  
    {#each Object.entries(availabilities) as lesson}
      <div class="lesson-card">
        <h3>{lesson[0]}</h3>
        <p>{lesson[1]}</p>
      </div>
    {/each}
  </div>
  