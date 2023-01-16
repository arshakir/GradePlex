<script>
  export let selected;
  export let classes;
  let show;
  let w;
  let iw;
  import { fly } from 'svelte/transition';
  import Icon from '@iconify/svelte';
</script>
<svelte:window bind:innerWidth={w}/>

{#if w < iw}
  {#if show}
    <nav transition:fly={{ x: -500, opacity : 1}}>
      <button on:click={() => show = false}> <Icon icon="material-symbols:close" inline=true /></button> <br> <br>
      <button on:click={() => {selected=-1; show = false}}> All Grades </button> <br> <br>
      {#each classes as c,i}
        <button on:click={() => {selected = i; show = false}}> {classes[i].title} </button> <br> <br>
      {/each}
    </nav>
  {/if}

  <button on:click={() => show = true}> <Icon icon="uil:bars" inline=true/></button>
{:else}
  <div class="bar" bind:offsetWidth={iw}>
    <button on:click={() => {selected=-1}}> All Grades </button>
    {#each classes as c,i}
      <button on:click={() => {selected = i}}> {classes[i].title} </button>
    {/each}
  </div>
{/if}
<style>
nav {
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  padding: 2rem 1rem 0.6rem;
  border-left: 1px solid #aaa;
  background: #ffffff;
  overflow-y: auto;
}
.bar {
  padding: 10px;
  display: inline;
  white-space: nowrap;
}
button {
  border: 1px solid #000;
}
</style>
