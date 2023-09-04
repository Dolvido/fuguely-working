<script lang="ts">
  import { page } from "$app/stores";
  import { db, user, userData } from "$lib/firebase";
  import { doc, updateDoc, writeBatch } from "firebase/firestore";
  import Logout from "$lib/components/Logout.svelte";
  // teacher imports
  import TeacherWindowPicker from "$lib/components/teacher/TeacherWindowPicker.svelte";
  import TeacherViewAvailabilities from "$lib/components/teacher/TeacherViewAvailabilities.svelte";
  //student imports
  import TeacherTableView from "$lib/components/student/TeacherTableView.svelte";
  import TeacherSearch from "$lib/components/student/TeacherSearch.svelte";
  // common imports
  import Navbar from "$lib/components/common/Navbar.svelte";

  let reloadAvailabilities = false;

  // Function to toggle the profile status
  async function toggleProfileStatus() {
    if (!$userData) {
      console.error("userData is undefined");
      return;
    }

    const userRef = doc(db, "users", $user!.uid);

    try {
      await updateDoc(userRef, {
        published: !$userData?.published,
      });
    } catch (error) {
      console.error("Failed to update document:", error);
    }
  }
  // Get profileType (not provided in pseudocode how to handle this, so adding a simple fetch)
  let profileType = "";
  async function fetchProfileType() {
    const userRef = doc(db, "users", $userData!.uid);
    const docData = await getDoc(userRef);
    profileType = docData.data().profileType;
  }
 

  
</script>
<Navbar title="Fuguely" />
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

    <!-- Teacher/Student mode display-->
    {#if $userData?.profileType === "Teacher"}
      <div class="text-center my-4">
        <a class="btn btn-outline btn-xs" href={`/${$userData.username}/edit`}>Teacher Mode</a>
        <!-- Teacher Schedule -->
        <TeacherWindowPicker  />
        <TeacherViewAvailabilities/>

      </div>
    {:else if $userData?.profileType === "Student"}
      <div class="text-center my-4">
        <a class="btn btn-outline btn-xs" href={`/${$userData.username}/edit`}>Student Mode</a>
      </div>
      <!-- Student teachers -->
      <TeacherSearch />
      <TeacherTableView />
      
    {/if}
    
    <Logout />
  {/if}
</main>
