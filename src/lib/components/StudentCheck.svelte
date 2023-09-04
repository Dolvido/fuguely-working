<script lang="ts">
    import { user, userData } from "$lib/firebase";

    $: href = `/${$userData?.username}/edit`;

  </script>
  
  {#if $userData?.profileType == "Teacher"}
    <slot />
  {:else}
      <p class="text-error my-10">
          You must be a student to view this content.
          <a class="btn btn-primary" {href}>Profile home</a>
      </p>
  {/if}