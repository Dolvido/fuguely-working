
<script lang="ts">
    import { db, user, userData } from "$lib/firebase";  // Assuming you've initialized firebase in this module
    import { doc, setDoc, getDoc } from "firebase/firestore";
    import { createEventDispatcher } from "svelte";
    import { availabilityData } from "$lib/firebase";

    const dispatch = createEventDispatcher();

    let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
    let halfHourIntervals = ["8:00 AM", "8:30 AM", "9:00 AM", "9:30 AM", "10:00 AM", "10:30 AM", "11:00 AM", "11:30 AM", 
                             "12:00 PM", "12:30 PM", "1:00 PM", "1:30 PM", "2:00 PM", "2:30 PM", "3:00 PM", "3:30 PM", 
                             "4:00 PM", "4:30 PM", "5:00 PM", "5:30 PM", "6:00 PM", "6:30 PM", "7:00 PM", "7:30 PM", 
                             "8:00 PM", "8:30 PM", "9:00 PM"];
    let selectedDay = null;
    let selectedStartTime = null;
    let selectedEndTime = null;
    let loading = false;

    function checkAvailability() {
        let isValid = false;
        
        // Check if selectedStartTime is before selectedEndTime
        if (selectedStartTime && selectedEndTime) {
            const startTime = new Date(`2000/01/01 ${selectedStartTime}`);
            const endTime = new Date(`2000/01/01 ${selectedEndTime}`);

            if (startTime < endTime) {
                console.log("Start time is before end time");
                isValid = true;
            }
        }
    }
    async function onSubmit() {
        if (selectedDay && selectedStartTime && selectedEndTime) {  
            if(selectedStartTime > selectedEndTime) {
                alert("Start time must be before end time");
                return;
            }
            const docRef = doc(db, 'users', $user!.uid); // Replace 'users' with the appropriate collection name
            const userData = await getDoc(docRef);
            const userDoc = userData.data() || {};

            const timeWindow = `${selectedStartTime} to ${selectedEndTime}`;

            console.log('selectedDay:', selectedDay);
            console.log('userDoc:', userDoc);

            // Create or update the 'availabilities' field with the key-value pair
            userDoc.availabilities = {
                ...userDoc.availabilities,
                [selectedDay]: timeWindow
            };

            console.log('updated userDoc:', userDoc);

            await setDoc(docRef, userDoc);
        }
        else    {
            alert("Please select a day, start time, and end time");
        }
    }
</script>

<!-- Grid Container -->
<div class="grid grid-cols-4 gap-4">
  
    <!-- Your Component -->
    <div class="col-span-4 row-span-1 flex flex-col items-center mx-auto">
  
      <div class="flex items-center mb-2 w-full justify-center">
        <label for="day-dropdown" class="mr-2">Select a day:</label>
        <select id="day-dropdown" bind:value={selectedDay} class="mx-1">
            <option disabled selected value={null}>-- Choose a day --</option>
            {#each days as day}
                <option value={day}>{day}</option>
            {/each}
        </select>
      </div>
      
      <div class="flex items-center mb-2 w-full justify-center">
        <label for="start-time-dropdown" class="mr-2">Start time:</label>
        <select id="start-time-dropdown" bind:value={selectedStartTime} class="mx-1">
            <option disabled selected value={null}>-- Choose a start time --</option>
            {#each halfHourIntervals as interval}
                <option value={interval}>{interval}</option>
            {/each}
        </select>
      </div>
  
      <div class="flex items-center mb-2 w-full justify-center">
        <label for="end-time-dropdown" class="mr-2">End time:</label>
        <select id="end-time-dropdown" bind:value={selectedEndTime} class="mx-1">
            <option disabled selected value={null}>-- Choose an end time --</option>
            {#each halfHourIntervals as interval}
                <option value={interval}>{interval}</option>
            {/each}
        </select>
      </div>
  
      <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" on:click|preventDefault={onSubmit}>
        Add
      </button>
  
    </div>
    <!-- End of Your Component -->
  
  </div>
  
