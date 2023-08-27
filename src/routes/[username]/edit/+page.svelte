<script lang="ts">
  import { page } from "$app/stores";
  import { db, userData } from "$lib/firebase";
  import { doc, updateDoc, writeBatch } from "firebase/firestore";
  import Logout from "$lib/components/Logout.svelte";

  // Function to toggle the profile status
  async function toggleProfileStatus() {
    const userRef = doc(db, "users", $userData.uid);
    await updateDoc(userRef, {
      published: !$userData?.published,
    });
  }

  // Function to toggle the teacher mode
  async function toggleTeacherMode() {
    const userRef = doc(db, "users", $userData.uid);
    await updateDoc(userRef, {
      teacherMode: !$userData?.teacherMode,
    });
  }
</script>

<main class="max-w-xl mx-auto">
  {#if $userData?.username === $page.params.username}
    <h1 class="mx-2 text-2xl font-bold mt-8 mb-4 text-center">Edit your Profile</h1>

    <div class="text-center mb-8">
      <p>
        Profile Link:
        <a href={`/${$userData?.username}`} class="link link-accent">
          https://fuguely-web-app.web.app/{$userData?.username}
        </a>
      </p>
    </div>

    <div class="text-center my-4">
      <a class="btn btn-outline btn-xs" href="/login/photo">Change photo</a>
      <a class="btn btn-outline btn-xs" href={`/${$userData.username}/bio`}>Edit Bio</a>
    </div>

    <!-- Toggle for Public/Private profile -->
    <form class="form-control">
      <label class="label cursor-pointer flex items-start justify-center">
        <span class="label-text mr-6">
          <div
            class="tooltip group-hover:tooltip-open"
            data-tip="If public, the world can see your profile"
          >
            {$userData?.published ? "Public profile" : "Private profile"}
          </div>
        </span>
        <input
          type="checkbox"
          class="toggle toggle-success"
          checked={$userData?.published}
          on:change={toggleProfileStatus}
        />
      </label>
    </form>

    <!-- Toggle for Teacher/Student mode -->
    <form class="form-control">
      <label class="label cursor-pointer flex items-start justify-center">
        <span class="label-text mr-6">
          {$userData?.teacherMode ? "Teacher Mode" : "Teacher Mode"}
        </span>
        <input
          type="checkbox"
          class="toggle toggle-success"
          checked={$userData?.teacherMode}
          on:change={toggleTeacherMode}
        />
      </label>
    </form>
    <Logout />
  {/if}
</main>
