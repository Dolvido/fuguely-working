<script lang="ts">
    import { auth } from "$lib/firebase"  // Update the path accordingly
    import { user } from "$lib/firebase";  // Update the path accordingly

    function logout() {
        console.log
        auth.signOut()
            .then(() => {
                // You can handle any post-logout logic here
                console.log("Logged out successfully");
                location.href = "/";

            })
            .catch((error) => {
                console.error("Error logging out: ", error);
            });

    }
</script>

{#if $user}
    <div class="text-center">
        <button class="btn btn-primary" on:click|preventDefault={logout}>Logout</button>
    </div>
{:else}
    <p>You are already logged out.</p>
{/if}
