<script lang="ts">
    import { db, user, userData } from "$lib/firebase";
    import {
        doc,
        getDoc,
        writeBatch,
        updateDoc,
        arrayUnion,
        setDoc
    } from "firebase/firestore";
    import { writable } from 'svelte/store';

    let teacher_uname = "";
    let loading = false;
    let teacherExists = writable(false);

    async function searchTeacher() {
        loading = true;
        const ref = doc(db, "usernames", teacher_uname);
        const exists = await getDoc(ref).then((doc) => doc.exists);
        loading = false;

        if (exists) {
            teacherExists.set(true);
        } else {
            teacherExists.set(false);
        }
    }

    async function addTeacher() {
        if ($teacherExists) {
            const studentRef = doc(db, "students", $user.uid);
            const studentDoc = await getDoc(studentRef);

            if (studentDoc.exists) {
                const teacherUsername = teacher_uname.trim();
                if (teacherUsername.length === 0) {
                    return;
                }

                const teachers = studentDoc.data().teachers || [];
                const updatedTeachers = [...teachers, teacherUsername];
                console.log("teachers:",teachers);
                console.log("updatedTeachers:",updatedTeachers);
                try {
                    await updateDoc(studentRef, { teachers: updatedTeachers });
                } catch (error) {
                    console.error("Error adding teacher:", error);
                }

                teacher_uname = "";
            }
        }
    }
</script>

<div>
    <input type="text" placeholder="Enter teacher's username" bind:value={teacher_uname} />
    <button on:click={searchTeacher} disabled={loading}>Search</button>
    
    {#if loading}
        <div>Loading...</div>
    {/if}
    
    {#if $teacherExists}
        <div>
            Teacher found! 
            <button on:click={addTeacher}>Add Teacher</button>
        </div>
    {:else}
        <div>Teacher not found.</div>
    {/if}
</div>
