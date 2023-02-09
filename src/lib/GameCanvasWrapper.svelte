<script>
  import { onMount } from 'svelte';
  import GameCanvasRPG from './GameCanvasRPG.svelte';
  import GameCanvasSpaceInvader from './GameCanvasSpaceInvader.svelte';
  import { _ } from 'svelte-i18n';

  export let state;

  const options = [
    { gameType: 'rpg', component: GameCanvasRPG },
    { gameType: 'spaceInvader', component: GameCanvasSpaceInvader },
  ];

  let selected = options[0];
  let gameCanvasRef;

  export function spawnKaboom() {
    gameCanvasRef.spawnKaboom();
  }
</script>

<select bind:value={selected} disabled={state === 'progress'}>
  {#each options as option}
    <option value={option}>{option.gameType}</option>
  {/each}
</select>

{#if state === 'progress'}
  <div id="mygame" />
  <svelte:component this={selected.component} bind:this={gameCanvasRef} />
{/if}
