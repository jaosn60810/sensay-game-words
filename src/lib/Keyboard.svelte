<script>
  import { createEventDispatcher, onMount } from 'svelte'
  let keyboard = [
    {
      '0': 'Q',
      '1': 'W',
      '2': 'E',
      '3': 'R',
      '4': 'T',
      '5': 'Y',
      '6': 'U',
      '7': 'I',
      '8': 'O',
      '9': 'P',
    },
    {
      '0': 'A',
      '1': 'S',
      '2': 'D',
      '3': 'F',
      '4': 'G',
      '5': 'H',
      '6': 'J',
      '7': 'K',
      '8': 'L',
    },
    { '0': 'Z', '1': 'X', '2': 'C', '3': 'V', '4': 'B', '5': 'N', '6': 'M' },
  ]

  const dispatch = createEventDispatcher()

  function pressKey(key) {
    dispatch('pressKey', key)
  }
</script>

{#each keyboard as ln, i}
  <div style="display: flex;">
    {#each Array.from(Object.keys(ln)) as b, j}
      <div class="">
        <button
          class="btn btn-primary btn-key "
          on:click={() => pressKey(keyboard[i][j])}>{keyboard[i][j]}</button
        >
      </div>
    {/each}
  </div>
{/each}

<style>
  .btn-key {
    margin-left: 0.5em;
    margin-right: 0.5em;
    margin-top: 1em;
    margin-bottom: 1em;
  }

  @media only screen and (max-width: 800px) {
    .btn-key {
      margin-left: 0.2em;
      margin-right: 0.2em;
      margin-top: 0.5em;
      margin-bottom: 0.5em;
    }
  }
</style>
