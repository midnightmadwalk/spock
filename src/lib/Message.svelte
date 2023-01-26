<script>
  import { onMount, onDestroy } from "svelte";
  import { currentUser, pb } from "./pocketbase";
  let messages = [];
  let newMessage = "";
  let files;
  let unsubscribe;
  const formData = new FormData();

  onMount(async () => {
    const resultList = await pb.collection("messages").getList(1, 50, {
      sort: "created",
      expand: "user",
    });
    messages = resultList.items;

    //subscribe

    unsubscribe = pb
      .collection("messages")
      .subscribe("*", async ({ action, record }) => {
        if (action === "create") {
          //get user
          const user = await pb.collection("users").getOne(record.user);
          record.expand = { user };
          messages = [...messages, record];
        }
        if (action === "delete") {
          messages = messages.filter((m) => m.id !== record.id);
        }
        if (action === "update") {
          messages = messages.map((m) => {
            if (m.id === record.id) {
              m.reaction = record.reaction;
            }
            return m;
          });
        }
      });
  });

  onDestroy(() => {
    unsubscribe?.();
  });

  async function sendMessage(e) {
    formData.append("text", newMessage);
    formData.append("user", $currentUser.id);
    if (files) formData.append("photo", files[0]);
    const createdMessage = await pb.collection("messages").create(formData);
    newMessage = "";
  }

  async function ReactToMessage({ message, react }) {
    const data = {
      reaction: [...message.reaction, react],
    };
    const record = await pb.collection("messages").update(message.id, data);
  }

  function init(el) {
    el.focus();
  }
</script>

<div class="message_container">
  <div class="messages">
    {#each messages as message (message.id)}
      <div class="msg" class:me={$currentUser.id === message.expand?.user?.id}>
        <div class="msg-details">
          <img
            src={`https://api.dicebear.com/5.x/identicon/svg?seed=${message.expand?.user?.username}`}
            alt="avatar"
            width="40px"
          />
          <small>{message.expand?.user?.username}</small>
        </div>
        <p class="msg-text">{message.text}</p>
        {#if message.photo}
          <img
            class="attachment"
            src={`http://127.0.0.1:8090/api/files/messages/${message.id}/${message.photo}`}
            alt=""
          />
        {/if}
        <span>{message.reaction}</span>
      </div>
      <div class="reactions">
        <button on:click={() => ReactToMessage({ message, react: "😂" })}
          >😂</button
        >
        <button on:click={() => ReactToMessage({ message, react: "🔥" })}
          >🔥</button
        >
        <button on:click={() => ReactToMessage({ message, react: "👍" })}
          >👍</button
        >
        <button on:click={() => ReactToMessage({ message, react: "👎" })}
          >👎</button
        >
        <button on:click={() => ReactToMessage({ message, react: "💩" })}
          >💩</button
        >
      </div>
    {/each}
  </div>
  <form class="form" on:submit|preventDefault={sendMessage}>
    <input
      type="text"
      bind:value={newMessage}
      use:init
      placeholder="Type Your Message"
    />
    <input
      accept="image/png, image/jpeg"
      bind:files
      id="avatar"
      name="avatar"
      type="file"
    />
    <button>Send</button>
  </form>
</div>

<style>
  .messages {
    display: flex;
    flex-flow: column nowrap;
    justify-content: flex-end;
    margin: 0 auto;
    width: 560px;
    flex-direction: column;
  }
  .msg {
    width: fit-content;
    background-color: honeydew;
    border-radius: 12px;
    padding: 0.5rem 1rem;
    margin-block: 0.5rem;
  }
  .msg.me {
    align-self: flex-end;
    background-color: lavender;
  }
  .reactions {
    align-self: flex-start;
  }
  .msg.me + .reactions {
    align-self: flex-end;
  }
  .reactions button {
    padding: 0;
  }
  .msg-details {
    display: flex;
    justify-content: start;
    align-items: center;
    gap: 0.5rem;
  }
  .msg-details > small {
    font-weight: 700;
  }
  .msg-details img {
    max-width: 32px;
    border-radius: 100vmax;
    display: block;
    user-select: none;
  }
  .msg-text {
    margin: 0.5rem 0;
    text-align: start;
  }
  .attachment {
    width: 500px;
    height: auto;
  }
  .form {
    position: sticky;
    bottom: 0%;
    width: 100%;
    display: flex;
    gap: 1rem;
    margin: 1rem 0;
  }
  .form input {
    width: 100%;
    padding: 0.5rem;
    border-radius: 5px;
    font-family: "Roboto", sans-serif;
  }
  .form button {
    font-family: "Roboto", sans-serif;
    background-color: aliceblue;
    color: black;
  }
</style>
