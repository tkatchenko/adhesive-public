<script>
  import Notes from './Notes.svelte';

  const hash = window.location.hash.substring(1);

  document.addEventListener('dragover', function(e) { e.preventDefault() })
</script>

{#if hash}
  <Notes board="{hash}"/>
{:else}
  No board selected, please add a hash to the url.
{/if}
