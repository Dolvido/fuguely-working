
<script lang="ts">
    import { onMount } from 'svelte';
    import { db } from "$lib/firebase";
    import { doc, setDoc, getDoc } from "firebase/firestore";
    import TimeRangeSelector from './TimeRangeSelector.svelte';

    let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
    let selectedDayForRange = null;
    let timeRangeForSpecificDay = '';
    let showTimeSelector = false; // Controls the visibility of the TimeRangeSelector

    function addDaySlotToScheduler(day: string) {
      selectedDayForRange = day;
      showTimeSelector = true; // Show the time selector when a day is clicked
    }

    $: if (selectedDayForRange && timeRangeForSpecificDay) {
      updateFirestore(selectedDayForRange, timeRangeForSpecificDay);
      selectedDayForRange = null;
      showTimeSelector = false; // Hide the time selector once time range is selected
    }

    async function updateFirestore(day: string, timeRange: string) {
      const docRef = doc(db, 'userDays', 'someUserId');

      const userData = await getDoc(docRef);
      const currentData = userData.data() || {};

      if (currentData[day]) {
        currentData[day].push(timeRange);
      } else {
        currentData[day] = [timeRange];
      }

      await setDoc(docRef, currentData, { merge: true });
    }
</script>

<div class="btn-group">
    {#each days as day}
      <button 
        class="btn btn-outline {selectedDayForRange === day ? 'active' : ''}" 
        on:click={() => addDaySlotToScheduler(day)}>
        {day}
      </button>
    {/each}
</div>

<!-- Table displaying selected days and time ranges -->
<table class="table mt-4">
  <thead>
      <tr>
          <th>Selected Days</th>
          <th>Time Range</th>
      </tr>
  </thead>
  <tbody>
      {#each days as day}
          {#if selectedDayForRange === day}
              <tr>
                  <td>{day}</td>
                  <td>{timeRangeForSpecificDay}</td>
                  {#if showTimeSelector}
                    <TimeRangeSelector bind:selectedRange={timeRangeForSpecificDay} />
                  {/if}
              </tr>
          {/if}
      {/each}
  </tbody>
</table>

<!-- You can add the table and other UI elements below as per your requirement -->
