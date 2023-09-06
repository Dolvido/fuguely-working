<script lang="ts">
    import { db, user, userData as existingUserData } from "$lib/firebase";
    import { doc, setDoc, getDoc, writeBatch } from "firebase/firestore";
    import { createEventDispatcher } from "svelte";
    
    const dispatch = createEventDispatcher();
    
    let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
    let halfHourIntervals = ["8:00 AM", "8:30 AM", "9:00 AM", "9:30 AM", "10:00 AM", "10:30 AM", "11:00 AM", "11:30 AM", "12:00 PM", "12:30 PM", "1:00 PM", "1:30 PM", "2:00 PM", "2:30 PM", "3:00 PM", "3:30 PM", "4:00 PM", "4:30 PM", "5:00 PM", "5:30 PM", "6:00 PM", "6:30 PM", "7:00 PM", "7:30 PM", "8:00 PM", "8:30 PM"];
    let selectedStartTime = "8:00 AM";
    let selectedEndTime = "8:30 AM";
    let selectedDay = "Sunday";
    
    async function onSubmit() {
        console.log("UID:", $user!.uid); // Debug log to check user ID
    
        const teacherDocRef = doc(db, 'teachers', $user!.uid);
        const lessonDocRef = doc(db, 'lessons', $user!.uid); // New collection
    
        const fetchedUserData = await getDoc(teacherDocRef);
        const userDoc = fetchedUserData.data() || {};
    
        const timeWindow = `${selectedStartTime} to ${selectedEndTime}`;
        console.log("Time Window:", timeWindow); // Debug log
    
        const transformedTimeFrames = generateTimeFrames(timeWindow, selectedDay);
        console.log("Transformed Time Frames:", transformedTimeFrames); // Debug log
    
        // Check if transformedTimeFrames is undefined or not
        if (!transformedTimeFrames) {
            console.error("Transformed Time Frames is undefined");
            return;
        }
    
        // Create a batch
        const batch = writeBatch(db);
    
        // For lessons collection, save the array of availabilities
        batch.set(lessonDocRef, {
            ...userDoc,
            availabilities: transformedTimeFrames
        });
    
        // For teachers collection, save the summarized time window for each day
        const timeWindowForDay = `${selectedStartTime} - ${selectedEndTime}`;
        userDoc.availabilities = {
            ...userDoc.availabilities,
            [selectedDay]: timeWindowForDay
        };
    
        batch.set(teacherDocRef, userDoc);
    
        // Commit the batch
        await batch.commit();
    }

function generateTimeFrames(timeWindow, selectedDay) {
    const [startTimeStr, endTimeStr] = timeWindow.split('to').map(str => str.trim());
    const [startHour, startMinute] = startTimeStr.split(' ')[0].split(':');
    const [endHour, endMinute] = endTimeStr.split(' ')[0].split(':');
    const startIsPM = startTimeStr.includes('PM');
    const endIsPM = endTimeStr.includes('PM');

    let currentTime = (startIsPM ? parseInt(startHour) + 12 : parseInt(startHour)) * 60 + parseInt(startMinute);
    const endTime = (endIsPM ? parseInt(endHour) + 12 : parseInt(endHour)) * 60 + parseInt(endMinute);
    
    const timeFrames = [];

    while (currentTime < endTime) {
        const nextTime = currentTime + 30;
        if (nextTime > endTime) {
            break;
        }
        
        const currentHour = Math.floor(currentTime / 60);
        const currentMinute = currentTime % 60;
        const nextHour = Math.floor(nextTime / 60);
        const nextMinute = nextTime % 60;

        const timeFrameStr = `${currentHour}:${currentMinute.toString().padStart(2, '0')} - ${nextHour}:${nextMinute.toString().padStart(2, '0')}`;
        timeFrames.push({ [selectedDay]: timeFrameStr });

        currentTime = nextTime;
    }
    
    return timeFrames;
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
</div>
