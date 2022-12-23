<script lang="ts">
  import { fly } from 'svelte/transition';
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

  const scrollDown = async (node: HTMLElement) => {
    node.scroll({ top: node.scrollHeight, behavior: 'smooth' });
  };

  async function sendMessage(): Promise<void> {
    let data = {
      text: newMessage,
      user: $currentUser.id,
    };
    //

    if (data.text && data.text.length < 160) {
      await pb.collection('messages').create(data);
    } else {
      return;
    }

    newMessage = null;
  }

  async function sendMonkey(): Promise<void> {
    try {
      let data = {
        text: '🙉',
        user: $currentUser.id,
      };
      if (data.text !== '🙉') return;

      await pb.collection('messages').create(data);
    } catch (err) {
      console.log(err.message);
    }
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
            class="msg-me"
            in:fly={{ x: 600, duration: 400 }}
            out:fly={{ x: 600, duration: 400 }}
          >
            <div class="me">
              <p class="msg-text">
                {message.text}
              </p>
            </div>
            <div class="avatar">
              <button class="me-btn" on:click={deleteMessage(message)}
                >💀</button
              >
            </div>
          </div>
        {:else}
          <div
            class="msg-not-me"
            in:fly={{ x: -200, duration: 300 }}
            out:fly={{ duration: 100 }}
          >
            <div class="not-me">
              <p class="msg-text">
                {message.text}
              </p>
            </div>
            <div class="avatar">
              <img
                src={`https://avatars.dicebear.com/api/identicon/${message.expand?.user?.username}.svg`}
                alt="avatar"
                class="img"
                width="40px"
              />
            </div>
          </div>
        {/if}
      {/each}
    {/if}
  </div>
</div>

{#if $currentUser}
  <div class="form-wrapper">
    <button on:click={sendMonkey} class="btn-monkey">🙉</button>
    <form class="form" on:submit|preventDefault={sendMessage}>
      <input
        class="input"
        type="text"
        placeholder="Message"
        bind:value={newMessage}
      />
      <button class="btn" type="submit">Send</button>
    </form>
  </div>
{/if}

<style>
  .empty {
    display: flex;
    padding: 10px;
    justify-content: center;
  }

  .form-wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 20px;
    gap: 10px;
  }

  .btn-monkey {
    border-radius: 50%;
    padding: 10px;
    font-size: 20px;
  }
  .form {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  ::-webkit-scrollbar {
    width: 12px;
  }
  ::-webkit-scrollbar-thumb {
    background: #a5a5a5;
    border-radius: 10px;
    border: 3px solid transparent;
    background-clip: padding-box;
    transition-duration: 200ms;
  }
  ::-webkit-scrollbar-thumb:hover {
    border: 0;
    transition-duration: 200ms;
  }
  ::-webkit-scrollbar-track {
    background: transparent;
  }

  .messages {
    align-self: center;
    overflow-x: hidden;
    height: 550px;
    width: 477px;
    border: 1px solid rgba(255, 255, 255, 0.185);
    border-radius: 8px;
    box-shadow: 1px 10px 50px -30px black;
    margin: 0 auto;
    padding: 20px;
    background-color: rgba(0, 0, 0, 0.678);
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .msg-me,
  .msg-not-me {
    display: flex;
    align-items: center;
    gap: 5px;
  }
  .me,
  .not-me {
    display: flex;
    padding: 5px;
    width: max-content;

    max-width: 250px;
    border-radius: 30px;
  }

  .msg-me {
    justify-content: flex-end;
    flex-direction: row;
  }
  .me {
    background-color: rgb(0, 131, 102);
  }
  .me-btn {
    padding: 5px;
    border-radius: 50%;
  }

  .msg-not-me {
    flex-direction: row-reverse;
    justify-content: flex-end;
  }
  .not-me {
    background-color: rgb(73, 73, 73);
  }

  .msg-text {
    word-break: break-all;
    padding: 10px 10px;
    font-size: 18px;
  }

  .avatar {
    border-radius: 50%;
    display: inherit;
    align-self: center;
  }

  .img {
    border-radius: 50%;
    scale: 0.8;
  }
</style>
