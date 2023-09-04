<script lang="ts">
    import { writable } from 'svelte/store';
    import { db, user, studentData } from "$lib/firebase";
    import { doc, getDoc } from "firebase/firestore";
    import { onMount } from "svelte";

    // Changed to Svelte store
    const loading = writable(false);

    // Using Svelte's reactive statements
    $: teachers = $studentData?.teachers || [];

    async function fetchTeachers() {
        console.log("fetching teachers");

        // Using Svelte store methods
        $loading = true;
        try {
            const studentRef = doc(db, "students", $user.uid);
            const studentDoc = await getDoc(studentRef);

            if (studentDoc.exists() && studentDoc.data()) {
                const teacherData = studentDoc.data().teachers || [];
                teachers = teacherData;  // Directly updating the variable
            } else {
                console.error(`Document doesn't exist.`);
                teachers = [];  // Directly updating the variable
            }
        } catch (error) {
            console.error("Error fetching teachers:", error);
            teachers = [];  // Directly updating the variable
        } finally {
            $loading = false;  // Using Svelte store methods
        }
    }

    onMount(() => {
        fetchTeachers();
    });

    function reloadTeachers() {
        fetchTeachers();
    }
</script>


<div>
    <!-- Using $loading to access the value of the loading store -->
    <button on:click={reloadTeachers} disabled={$loading}>Reload</button>
    
    {#if $loading}
        <div>Loading...</div>
    {/if}
    
    <table>
        <thead>
            <tr>
                <th>Name</th>
            </tr>
        </thead>
        <tbody>
            <!-- Using teachers directly since it's a reactive variable -->
            {#each teachers as teacher (teacher.id)}
                <tr>
                    <td>{teacher}</td>
                </tr>
            {/each}
        </tbody>
    </table>
</div>
