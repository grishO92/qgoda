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
  <div class="wrapper">
    <p class="user">
      Signed in as {$currentUser.username}
    </p>
    <img
      src={`https://avatars.dicebear.com/api/identicon/${$currentUser.username}.svg`}
      alt="avatar"
      class="img"
      width="40px"
    />
    <button on:click={signOut}>SignOut</button>
  </div>
{:else}
  <form class="form" on:submit|preventDefault>
    <div class="inputs">
      <input type="text" placeholder="Username" bind:value={username} />
      <input type="password" placeholder="Password" bind:value={password} />
    </div>
    <div class="btns">
      <button on:click={signUp}>Sign Up</button>
      <button on:click={login}>Login</button>
    </div>
  </form>
{/if}

<style>
  .wrapper {
    display: flex;
    align-items: center;
    gap: 20px;
  }

  .img {
    border-radius: 50%;
    scale: 1.3;
  }

  .user {
    font-weight: 600;
    font-size: 20px;
  }

  .form {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }
  .inputs {
    display: flex;
    flex-direction: column;
    gap: 20px;
  }
  .btns {
    display: flex;
    gap: 20px;
  }
</style>
