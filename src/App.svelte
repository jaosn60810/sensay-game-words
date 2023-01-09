<script>
  import Game from './lib/Game.svelte'
  import {
    addMessages,
    register,
    init,
    getLocaleFromNavigator,
  } from 'svelte-i18n'
  import { onMount } from 'svelte'
  import en from './i18n/en.json'

  addMessages('en', en)

  init({
    fallbackLocale: 'en',
    initialLocale: getLocaleFromNavigator(),
  })

  let gameLevel = 'Level5'

  onMount(async () => {
    const url_string = window.location.href
    const queryString = window.location.search
    console.log(queryString)
    const urlParams = new URLSearchParams(queryString)
    gameLevel = urlParams.get('gameLevel')
    console.log('gameLevel', gameLevel)
  })
</script>

<main>
  <h1>Word Game</h1>

  <div class="card">
    <Game {gameLevel} />
  </div>
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
