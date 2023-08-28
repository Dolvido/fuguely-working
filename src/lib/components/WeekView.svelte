<script lang="ts">
  import { onMount } from 'svelte';
  import { db, user } from "$lib/firebase";  // Assuming you've initialized firebase in this module
  import { doc, setDoc, getDoc, updateDoc } from "firebase/firestore";
  import TimeRangeSelector from "./TimeRangeSelector.svelte";


  let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
  let selectedDays: { [key: string]: number } = {};
  let timeRanges = {};

  function setTimeRange(day, range){
    timeRanges[day] = range;
  }

  days.forEach(day => {
    selectedDays[day] = 0;
  });

  
  function addDaySlotToScheduler(day: string) {
    selectedDays[day]++;
    updateFirestore(day, selectedDays[day]);
  }

  function removeDaySlotFromScheduler(day: string) {
      if (selectedDays[day] > 0) {
        selectedDays[day]--;
        updateFirestore(day, selectedDays[day]);
      }
    }

  /* async function updateFirestore(day: string, count: number) {
    const docRef = doc(db, 'users', $user!.uid);  // Replace 'someUserId' with appropriate user ID or doc ID
    await setDoc(docRef, { [day]: count }, { merge: true });
  } */

  async function updateFirestore(day: string, timeRange: string) {
    console.log("updateFirestore", day, timeRange);
    const docRef = doc(db, 'users', $user!.uid); // Replace 'someUserId' with the appropriate user ID
    
    // Fetch the current data for the user
    const userData = await getDoc(docRef);
    const currentData = userData.data() || {};
    
    // Check if the day already has time ranges
    if (currentData[day]) {
      // If the time range doesn't exist, add it
      if (!currentData[day].includes(timeRange)) {
        currentData[day].push(timeRange);
      }
    } else {
      currentData[day] = [timeRange];
    }

    // Save the updated data back to Firestore
    await setDoc(docRef, currentData, { merge: true });
}

</script>
<div class="btn-group">
  {#each days as day}
    <button 
      class="btn btn-outline {selectedDays[day] > 0 ? 'active' : ''}" 
      on:click={() => addDaySlotToScheduler(day)}>
      {day}
    </button>
  {/each}
</div>

<!-- Display the table of selected days -->
<table class="table mt-4">
  <thead>
      <tr>
          <th>Selected Days</th>
          <th>Time Range</th>
          <th>Action</th>
      </tr>
  </thead>
  <tbody>
      {#each days as day (day)}
          {#if selectedDays[day] > 0}
              {#each Array(selectedDays[day]).fill(day) as _, index}
                  <tr>
                      <td>{day}</td>
                      <td>
                        <TimeRangeSelector bind:selectedRange={timeRanges[day + index]} on:change={() => setTimeRange(day + index, timeRanges[day + index])} />
                      </td>
                      <td><button class="btn btn-danger" on:click={() => removeDaySlotFromScheduler(day)}>-</button></td>
                  </tr>
              {/each}
          {/if}
      {/each}
  </tbody>
</table>
