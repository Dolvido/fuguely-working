<script lang="ts">
    import { auth } from "$lib/firebase"  // Update the path accordingly
    import { user } from "$lib/firebase";  // Update the path accordingly

    function logout() {
        auth.signOut()
            .then(() => {
                // You can handle any post-logout logic here
                console.log("Logged out successfully");
            })
            .catch((error) => {
                console.error("Error logging out: ", error);
            });
    }
</script>

{#if $user}
<a class="btn btn-primary" href="/">Logout</a>
    
{:else}
    <p>You are already logged out.</p>
{/if}
