<script>
  import {
    writeText,
    readText,
    readImage,
    writeImage,
    listenText,
    TEXT_CHANGED,
    IMAGE_CHANGED,
    listenImage,
  } from "tauri-plugin-clipboard-api";
  import { writeBinaryFile, BaseDirectory } from "@tauri-apps/api/fs";
  import { emit, listen } from "@tauri-apps/api/event";
  import { onDestroy, onMount } from "svelte";

  let open = false;

  let listenTextContent = [];
  let listenImageContent = "";
  let tauriTextUnlisten;
  let tauriImageUnlisten;
  let textUnlisten;
  let imageUnlisten;
  export async function startListening() {
    tauriTextUnlisten = await listen(TEXT_CHANGED, (event) => {
      listenTextContent = [{time: new Date().toLocaleString(), content:event.payload.value}, ...listenTextContent];
    });
    tauriImageUnlisten = await listen(IMAGE_CHANGED, (event) => {
      listenImageContent = event.payload.value;
      listenTextContent = [
        {time: new Date().toLocaleString(), content:`data:image/png;base64, ${listenImageContent}`},
        ...listenTextContent,
      ];
    });
    imageUnlisten = listenImage();
    textUnlisten = listenText();
  }
  function stopListening() {
    imageUnlisten();
    textUnlisten();
    tauriTextUnlisten();
    tauriImageUnlisten();
  }
  onMount(() => {
    startListening();
  });
  onDestroy(() => {
    stopListening();
  });
</script>

<div>
  <div
    style="display: {open
      ? 'block'
      : 'none'};"
    class="modal-dialog"
  >
    复制成功!
  </div>
  <div>
    {#each listenTextContent as {time,content}}
    
      <div class="list-item-ctn">
        <div class="list-item-time">{time}</div>
        {#if content.startsWith("data:image/png;base64,")}
          <img src={content} alt="img" class="list-item-img" />
        {:else}
          <span class="list-item">{content}</span>
        {/if}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <div
          style="cursor:pointer;color:#1777ff"
          on:click={() =>
            writeText(content).then(() => {
              open = true;
              setTimeout(() => {
                open = false;
              }, 3000);
            })}
        >
          copy
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  .list-item-ctn {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    padding: 20px;
    border-bottom: 1px solid #ccc;
  }
  .list-item {
    color: brown;
    text-decoration: none;
    font-size: 15px;
    font-weight: bold;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
    width: 60vw;
  }
  .list-item-time {
    color: #666;
    font-size: 12px;
    font-weight: bold;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
    width: 16vw;
    min-width: 140px;
  }
  .list-item-img{
    max-width: 60vw;
    max-height:200px
  }
  .modal-dialog {
    position:fixed;
    top:100px;
    left:40%;
    background:#f7df37ff;
    padding:10px;
    border-radius:10px
  }
</style>
