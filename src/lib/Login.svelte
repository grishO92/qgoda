<script lang="ts">
  import { currentUser, pb } from './pocketbase';

  let username: string;
  let password: string;

  async function login() {
    try {
      await pb.collection('users').authWithPassword(username, password);
    } catch (err) {
      console.log(err.message);
    }
  }

  async function signUp() {
    try {
      const data = {
        username,
        password,
        passwordConfirm: password,
        name: 'dai mi piene',
      };
      await pb.collection('users').create(data);
      await login();
    } catch (err) {
      console.log(err.message);
    }
  }

  function signOut() {
    pb.authStore.clear();
  }
</script>

{#if $currentUser}
  <p>
    Signed in as {$currentUser.username}
    <button on:click={signOut}>SignOut</button>
  </p>
{:else}
  <form on:submit|preventDefault>
    <input type="text" placeholder="Username" bind:value={username} />
    <input type="password" placeholder="Password" bind:value={password} />
    <button on:click={signUp}>Sign Up</button>
    <button on:click={login}>Login</button>
  </form>
{/if}
