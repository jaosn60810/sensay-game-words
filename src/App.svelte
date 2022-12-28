<script>
  import Game from './lib/Game.svelte'
  import {
    addMessages,
    register,
    init,
    getLocaleFromNavigator,
  } from 'svelte-i18n'
  import en from './i18n/en.json'

  addMessages('en', en)

  init({
    fallbackLocale: 'en',
    initialLocale: getLocaleFromNavigator(),
  })
</script>

<main>
  <h1>Word Game</h1>

  <div class="card">
    <Game />
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
