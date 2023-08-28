<script>
    let hours = Array.from({ length: 24 }, (_, i) => (i < 10 ? '0' + i : i) + ':00');
  
    let startTime = "00:00";
    let endTime = "01:00";
  
    export let selectedRange = null;
  
    function updateRange() {
      selectedRange = `${startTime}-${endTime}`;
    }
  </script>
  
  <div>
    <select bind:value={startTime} on:change={updateRange}>
        {#each hours as hour, index (index)}
          {#if index < hours.length - 1}
            <option value={hour}>{hour}</option>
          {/if}
        {/each}
      </select>
    to
    <select bind:value={endTime} on:change={updateRange}>
        {#each hours as hour, index (index)}
          {#if index < hours.length - 1}
            <option value={hour}>{hour}</option>
          {/if}
        {/each}
      </select>
  </div>