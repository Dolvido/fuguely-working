<script lang="ts">
    import { writable } from 'svelte/store';
    import { db, user, studentData } from "$lib/firebase";
    import { doc, getDoc, updateDoc, arrayRemove } from "firebase/firestore";
    import { onMount } from "svelte";

    const loading = writable(false);

    $: teachers = $studentData?.teachers || [];

    async function fetchTeachers() {
        console.log("fetching teachers");

        $loading = true;
        try {
            const studentRef = doc(db, "students", $user.uid);
            const studentDoc = await getDoc(studentRef);

            if (studentDoc.exists() && studentDoc.data()) {
                const teacherData = studentDoc.data().teachers || [];
                teachers = teacherData;
            } else {
                console.error(`Document doesn't exist.`);
                teachers = [];
            }
        } catch (error) {
            console.error("Error fetching teachers:", error);
            teachers = [];
        } finally {
            $loading = false;
        }
    }

    async function deleteTeacher(teacherName: string) {
        console.log(`Deleting teacher: ${teacherName}`);

        try {
            const studentRef = doc(db, "students", $user.uid);
            await updateDoc(studentRef, {
                teachers: arrayRemove(teacherName)
            });

            // Reload the teachers after deleting one
            fetchTeachers();
        } catch (error) {
            console.error("Error deleting teacher:", error);
        }
    }

    onMount(() => {
        fetchTeachers();
    });
</script>

<style>
    .empty-cell {
        min-width: 50px;
    }
</style>

<div>
    {#if $loading}
        <div>Loading...</div>
    {/if}

    <table>
        <thead>
            <tr>
                <th>Name</th>
                <th class="empty-cell"></th>
                <th>Schedule</th>
                <th>Action</th> <!-- Header for the delete button -->
            </tr>
        </thead>
        <tbody>
            {#each teachers as teacher}
                <tr>
                    <td>{teacher}</td>
                    <td class="empty-cell"></td>
                    <td>
                        <a href={`/${teacher}`} class="btn-class">View Schedule</a>
                    </td>
                    <td>
                        <!-- Delete button -->
                        <button on:click={() => deleteTeacher(teacher)} class="btn-class">Delete</button>
                    </td>
                </tr>
            {/each}
        </tbody>
    </table>
</div>
