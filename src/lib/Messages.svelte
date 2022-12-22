<script lang="ts">
  import { fly, fade } from 'svelte/transition';
  import { onMount, onDestroy, afterUpdate } from 'svelte';
  import { currentUser, pb } from './pocketbase';

  let messages: any[] = [];
  let newMessage: string;
  let chatPosition: HTMLElement;
  let unsubscribe: () => void;

  onMount(async () => {
    const resultList = await pb.collection('messages').getList(1, 50, {
      sort: 'created',
      expand: 'user',
    });

    messages = resultList.items;

    unsubscribe = await pb
      .collection('messages')
      .subscribe('*', async ({ action, record }) => {
        if (action === 'create') {
          const user = await pb.collection('users').getOne(record.user);
          record.expand = { user };
          messages = [...messages, record];
        }
        if (action === 'delete') {
          messages = messages.filter((m) => m.id !== record.id);
        }
      });
  });

  afterUpdate((): void => {
    if (messages) scrollDown(chatPosition);
  });

  $: if (messages && chatPosition) scrollDown(chatPosition);

  onDestroy((): void => {
    unsubscribe?.();
  });

  const scrollDown = async (node) => {
    node.scroll({ top: node.scrollHeight, behavior: 'smooth' });
  };

  async function sendMessage(): Promise<void> {
    let data = {
      text: newMessage,
      user: $currentUser.id,
    };
    if (!data.text) {
      data.text = '🙉';
    }

    if (data.text.length < 160) {
      await pb.collection('messages').create(data);
    } else {
      return;
    }
    newMessage = null;
  }

  function deleteMessage(message: { user: string; id: string }): any {
    $currentUser.id === message.user
      ? pb.collection('messages').delete(message.id)
      : null;
  }
</script>

<div class="wrapper">
  <div class="messages" bind:this={chatPosition}>
    {#if messages.length === 0}
      <p
        class="empty"
        in:fly={{ y: -100, delay: 400, duration: 400 }}
        out:fly={{ y: -100, delay: 0, duration: 10 }}
      >
        empty room! 😁
      </p>
    {:else}
      {#each messages as message (message.id)}
        {#if $currentUser && $currentUser.id === message.user}
          <div
            class="msg me"
            in:fly={{ x: 600, duration: 400 }}
            out:fly={{ x: 600, duration: 400 }}
          >
            <p class="msg-text">
              {message.text}
            </p>
            <button class="btn" on:click={deleteMessage(message)}>Delete</button
            >
            <small class="sentBy">
              Sent by @{message.expand?.user?.username}
              <img
                src={`https://avatars.dicebear.com/api/identicon/${message.expand?.user?.username}.svg`}
                alt="avatar"
                class="avatar"
                width="40px"
              />
            </small>
          </div>
        {:else}
          <div
            class="msg other"
            in:fly={{ x: -200, duration: 300 }}
            out:fly={{ duration: 100 }}
          >
            <p class="msg-text">
              {message.text}
            </p>
            <small class="sentBy">
              Sent by @{message.expand?.user?.username}
              <img
                src={`https://avatars.dicebear.com/api/identicon/${message.expand?.user?.username}.svg`}
                alt="avatar"
                class="avatar"
                width="40px"
              />
            </small>
          </div>
        {/if}
      {/each}
    {/if}
  </div>
</div>

{#if $currentUser}
  <form class="form" on:submit|preventDefault={sendMessage}>
    <input
      class="input"
      type="text"
      placeholder="if empty.............................you send => 🙉"
      bind:value={newMessage}
    />
    <button class="btn" type="submit">Send</button>
  </form>
{/if}

<style>
  .empty {
    display: flex;
    padding: 10px;
    justify-content: center;
  }
  .sentBy {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 10px;
    margin: 10px;
  }

  .form {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .messages {
    align-self: center;
    overflow-x: hidden;
    width: 500px;
    height: 550px;
    border: 1px solid rgba(255, 255, 255, 0.185);
    border-radius: 8px;
    box-shadow: 1px 10px 50px -30px black;
    margin: 0 auto;
  }

  .messages::-webkit-scrollbar {
    width: 8px;
    height: 10px;
    border: none;
  }
  .messages::-webkit-scrollbar-corner {
    background-color: transparent;
  }

  .messages::-webkit-scrollbar-thumb {
    background-color: rgb(173, 173, 173);
    border-radius: 8px;
  }
  .messages::-webkit-scrollbar-track {
    background-color: transparent;
  }

  .msg {
    border: 1px solid rgba(255, 255, 255, 0.185);
    margin: 3px 0;
    border-radius: 8px;

    background-color: rgba(0, 0, 0, 0.678);
  }
  .msg-text {
    word-break: break-all;
    padding: 20px;
  }

  .avatar {
    border-radius: 10px;
  }
</style>
