<script lang="ts">
  import AuthCheck from "$lib/components/AuthCheck.svelte";
  import { db, user, userData } from "$lib/firebase";
  import { doc, getDoc, writeBatch } from "firebase/firestore";
  let username = "";
  let loading = false;
  let isAvailable = false;
  let debounceTimer: NodeJS.Timeout;
  let selectedRole = "";

  const re = /^(?=[a-zA-Z0-9._]{3,16}$)(?!.*[_.]{2})[^_.].*[^_.]$/;

  $: isValid =
    username?.length > 2 && username.length < 16 && re.test(username);
  $: isTouched = username.length > 0;
  $: isTaken = isValid && !isAvailable && !loading;
  $: isRoleSelected = selectedRole === "Student" || selectedRole === "Teacher";

  function checkAvailability() {
    isAvailable = false;
    clearTimeout(debounceTimer);
    if (!isValid) {
      loading = false;
      return;
    }

    loading = true;

    debounceTimer = setTimeout(async () => {
      console.log("checking availability of", username);

      const ref = doc(db, "usernames", username);
      const exists = await getDoc(ref).then((doc) => doc.exists());

      isAvailable = !exists;
      loading = false;
    }, 500);
  }

  async function confirmUsername() {
    console.log("confirming username", username);
    const batch = writeBatch(db);
    console.log("setting username");
    batch.set(doc(db, "usernames", username), { uid: $user?.uid });
    batch.set(doc(db, "users", $user!.uid), {
      username,
      photoURL: $user?.photoURL ?? null,
      published: false,
      bio: "",
      profileType: selectedRole,
    });
    console.log("setting profile type")
    if(selectedRole === "Teacher") {
      batch.set(doc(db, "teachers", $user!.uid), {
        username,
        photoURL: $user?.photoURL ?? null,
        published: false,
        bio: "",
        availabilities: [],
      });
      console.log("setting avail");
      batch.set(doc(db, "availabilities", $user!.uid), {
        availabilities: [],
      }); 
    }
    if(selectedRole === "Student") {
      console.log("setting student");

      batch.set(doc(db, "students", $user!.uid), {
        username,
        photoURL: $user?.photoURL ?? null,
        published: false,
        teachers: [],
      });
    }
    try{
      console.log("committing");
      await batch.commit();
    } catch (e) {
      console.log(e);
    }

    username = "";
    isAvailable = false;
  }
</script>

<AuthCheck>
  {#if $userData?.username}
    <p class="text-lg">
      Your username is <span class="text-success font-bold"
        >@{$userData.username}</span
      >
    </p>
    <p class="text-sm">(Usernames cannot be changed)</p>
    <a class="btn btn-primary" href="/login/photo">Upload Profile Image</a>
  {:else}
    <form class="w-2/5" on:submit|preventDefault={confirmUsername}>
      <input
        type="text"
        placeholder="Username"
        class="input w-full"
        bind:value={username}
        on:input={checkAvailability}
        class:input-error={!isValid && isTouched}
        class:input-warning={isTaken}
        class:input-success={isAvailable && isValid && !loading}
      />
      <div class="my-4 min-h-16 px-8 w-full">
        <label class="label" for="roleDropdown">Select your role</label>
        <p class="text-sm text-secondary">This cannot be changed later</p>
        <select id="roleDropdown" bind:value={selectedRole} class="form-select w-full mt-2">
          <option value="Student">Student</option>
          <option value="Teacher">Teacher</option>
        </select>
      </div>
      <div class="my-4 min-h-16 px-8 w-full">
        {#if loading}
          <p class="text-secondary">Checking availability of @{username}...</p>
        {/if}

        {#if !isValid && isTouched}
          <p class="text-error text-sm">
            must be 3-16 characters long, alphanumeric only
          </p>
        {/if}

        {#if isValid && !isAvailable && !loading}
          <p class="text-warning text-sm">
            @{username} is not available
          </p>
        {/if}
        {#if isAvailable && !isRoleSelected}
          <p class="text-warning text-sm">
            Please select a role
          </p>
        {/if}
        {#if isAvailable && isRoleSelected} <!-- Update this line -->
          <button class="btn btn-success">Confirm username @{username} </button>
        {:else}
          <!-- You can add an additional message or styling here to indicate that role selection is required -->
          <button class="btn btn-success" disabled>Confirm username @{username} </button>
        {/if}
      </div>
     
    </form>
  {/if}
  
</AuthCheck>
