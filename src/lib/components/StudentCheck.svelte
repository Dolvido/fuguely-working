<script lang="ts">
  import { user, userData } from "$lib/firebase";
  import { onMount } from "svelte";

  let loading = true;

  $: href = `/${$userData?.username}/edit`;

  onMount(() => {
      console.log("userData", $userData);
      loading = false;
  });
</script>

{#if loading}
  <p>Loading...</p>
{:else if $userData?.profileType === "Student"}
  <slot />
{:else}
  <p class="text-error my-10">
      You must be a student to view this content.
      <a class="btn btn-primary" {href}>Profile home</a>
  </p>
{/if}
