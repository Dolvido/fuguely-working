<script lang="ts">
  import { db, user, teacherData } from "$lib/firebase";  // Assuming you've initialized firebase in this module
  import { doc, setDoc, getDoc } from "firebase/firestore";
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  // ... (existing variables and functions)

  function generateTimeSlots(startTime, endTime) {
    // Your code to generate 30-minute slots within a time window.
    // Return an array of these slots.
  }

  async function onSubmit() {
      if (selectedDay && selectedStartTime && selectedEndTime) {  
          if(selectedStartTime > selectedEndTime) {
              alert("Start time must be before end time");
              return;
          }

          // Generate time slots for the selected time window
          const timeSlots = generateTimeSlots(selectedStartTime, selectedEndTime);

          // Read the existing availabilities for the user if any
          const availabilitiesRef = doc(db, 'availabilities', $user!.uid);
          const existingData = await getDoc(availabilitiesRef);
          let existingAvailabilities = existingData.exists ? existingData.data().availabilities : [];

          // Create a new availability entry
          const newAvailability = {
              day: selectedDay,
              timeSlots: timeSlots
          };

          // Add the new availability to the existing list
          existingAvailabilities = [...existingAvailabilities, newAvailability];

          // Update the availabilities collection
          await setDoc(availabilitiesRef, { availabilities: existingAvailabilities });

          const docRef = doc(db, 'teachers', $user!.uid); // Replace 'teachers' with the appropriate collection name
          const userData = await getDoc(docRef);
          const userDoc = userData.data() || {};

          const timeWindow = `${selectedStartTime} to ${selectedEndTime}`;

          // Create or update the 'availabilities' field in the teachers collection
          userDoc.availabilities = [
              ...userDoc.availabilities || [],
              {
                  day: selectedDay,
                  timeWindow: timeWindow
              }
          ];

          await setDoc(docRef, userDoc);
      }
      else {
          alert("Please select a day, start time, and end time");
      }
  }
</script>

<!-- (Your existing HTML remains unchanged) -->

  
  {#if $teacherData && $teacherData.availabilities}
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    {#each Object.entries($teacherData.availabilities) as [day, time]}
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



  