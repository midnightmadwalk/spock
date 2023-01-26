<script>
  import { currentUser, pb } from "./pocketbase";
  let username = "snskrr";
  let password = "snskrrsnskrr";

  async function Login() {
    await pb.collection("users").authWithPassword(username, password);
  }
  async function SignUp() {
    try {
      const data = {
        username,
        password,
        passwordConfirm: password,
      };
      const createduser = await pb.collection("users").create(data);
      console.log(createduser);
      await Login();
    } catch (err) {
      console.error(err);
    }
  }

  async function SignOut() {
    await pb.authStore.clear();
  }
</script>

{#if $currentUser}
  <p>Signed in as {$currentUser.username}</p>
{:else}
  <form on:submit|preventDefault>
    <input type="text" placeholder="Username" bind:value={username} required />
    <input
      type="password"
      placeholder="Password"
      bind:value={password}
      required
      minlength="8"
      maxlength="72"
    />
    <button on:click={Login}>Login</button>
    <button on:click={SignUp}>Sign Up</button>
  </form>
{/if}
