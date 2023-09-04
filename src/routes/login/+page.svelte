<script lang="ts">
  import { auth, user, userData} from "$lib/firebase";

  import { GoogleAuthProvider, signInWithPopup, signOut } from "firebase/auth";

  // get dynamic data from server of username for routing to edit page
  $: users_href = `/${$userData?.username}/edit`;
  // show next button if user but no userData
  $: show_next =  $user && !$userData;

  async function signInWithGoogle() {
    const provider = new GoogleAuthProvider();
    const credential = await signInWithPopup(auth, provider);

    const idToken = await credential.user.getIdToken();

    const res = await fetch("/api/signin", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        // 'CSRF-Token': csrfToken  // HANDLED by sveltekit automatically
      },
      body: JSON.stringify({ idToken }),
    });

  }

  async function signOutSSR() {
    const res = await fetch("/api/signin", { method: "DELETE" });
    await signOut(auth);
  }

</script>

<h2>Login</h2>

{#if $user}
  <h2 class="card-title">Welcome, {$user.displayName}</h2>
  <p class="text-center text-success">You are logged in</p>
  {#if $userData}
    <a href={users_href} class="btn btn-primary"> Edit Profile </a>
  {:else}
    {#if show_next}
      <a href="/login/username" class="btn btn-primary">Next</a>
    {:else}
      <p class="text-center text-error">You do not have a profile</p>
    {/if}
  {/if}
  <button class="btn btn-warning" on:click={signOutSSR}
    >Sign out</button
  >
{:else}
  <button class="btn btn-primary" on:click={signInWithGoogle}
    >Sign in with Google</button
  >
{/if}
