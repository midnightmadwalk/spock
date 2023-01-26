<script>
  import Login from "./lib/Login.svelte";
  import Message from "./lib/Message.svelte";
  import { currentUser } from "./lib/pocketbase";
</script>

<main>
  {#if $currentUser}
    <Message />
  {:else}
    <Login />
  {/if}
</main>

<style>
  main {
    position: relative;
  }
</style>
