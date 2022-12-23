<script lang="ts">
  import Login from './lib/Login.svelte';
  import Messages from './lib/Messages.svelte';
</script>

<div class="wrapper">
  <h1 class="title">DrinkChat</h1>
  <div>
    <Login />
  </div>
  <div>
    <Messages />
  </div>
</div>

<style>
  .wrapper {
    background-color: #242424;
    box-shadow: 1px 1px 50px -30px black;
    display: flex;
    border: 1px solid rgba(255, 255, 255, 0.301);
    box-shadow: 0 0 75px -60px black;
    border-radius: 8px;
    padding: 20px 10px;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }

  .title {
    margin: 20px;
  }
</style>
