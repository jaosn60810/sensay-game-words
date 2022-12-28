<script>
  import AudioPlayer from './AudioPlayer.svelte'
  import { flip } from 'svelte/animate'
  import Keyboard from './Keyboard.svelte'
  import Recognition from './Recognition.svelte'
  import { _ } from 'svelte-i18n'

  import { onMount } from 'svelte'

  let audioPlayer
  let recognition
  let ttsButtonState

  let wordBank = {
    animals: [
      {
        word: 'mouse',
      },
      {
        word: 'cat',
      },
      {
        word: 'horse',
      },
      {
        word: 'chicken',
      },
      {
        word: 'rabbit',
      },
      {
        word: 'pig',
      },
      {
        word: 'dog',
      },
      {
        word: 'insect',
      },
      {
        word: 'cow',
      },
      {
        word: 'bird',
      },
      {
        word: 'goat',
      },
      {
        word: 'sheep',
      },
    ],

    restaurant: [
      {
        word: 'waitress',
      },
      {
        word: 'glass',
      },
      {
        word: 'menu',
      },
      {
        word: 'cup',
      },
      {
        word: 'knife',
      },
      {
        word: 'plate',
      },
      {
        word: 'chef',
      },
      {
        word: 'meal',
      },
      {
        word: 'waiter',
      },
      {
        word: 'fork',
      },
      {
        word: 'bill',
      },
      {
        word: 'spoon',
      },
    ],
  }

  let wordList = wordBank['restaurant']

  let gameCounter = 0
  let winCounter = 0
  let state = 'new'

  let selectedWord = ''
  let selectedWordDuration
  let pos = 0

  let gameTimer = 0
  let gameTimerInterval
  let gameTimerTotal = 0

  let lastMessage

  let debug = false

  let wordsSaid = []

  function start() {
    audioPlayer.playJingle3()

    gameCounter++
    pos = 0
    state = 'progress'
    selectedWord = wordList[pos].word

    console.log(111, selectedWord)

    selectedWordDuration = 10000

    gameTimer = 0

    lastMessage = ''

    clearInterval(gameTimerInterval)
    gameTimerInterval = setInterval(() => {
      gameTimer += 100
      gameTimerTotal += 100

      if (gameTimer >= 45000) {
        end()
      }
    }, 100)

    recognition.startRecognition()
  }

  function end() {
    clearInterval(gameTimerInterval)
    recognition.stopRecognition()
    state = 'over-loss'
  }

  function msToTime(s) {
    // Pad to 2 or 3 digits, default is 2
    function pad(n, z) {
      z = z || 2
      return ('00' + n).slice(-z)
    }

    var ms = s % 1000
    s = (s - ms) / 1000
    var secs = s % 60
    s = (s - secs) / 60
    var mins = s % 60
    var hrs = (s - mins) / 60

    return pad(mins) + ':' + pad(secs)
    // return pad(hrs) + ':' + pad(mins) + ':' + pad(secs) + '.' + pad(ms, 3)
  }

  function processWord(event) {
    let word = event.detail

    wordsSaid.push({ word: word, timestamp: gameTimer })
    console.log(666, wordsSaid)

    if (word === selectedWord) {
      winCounter++
      pos++
      if (pos === wordList.length) {
        audioPlayer.playJingle3()
        state = 'over-win'
        lastMessage = ''
        end()
      } else {
        selectedWord = wordList[pos].word
        audioPlayer.playJingle1()
        lastMessage = ''
      }
    } else {
      audioPlayer.playJingle2()
      lastMessage = 'WRONG_WORD: ' + JSON.stringify(word)
    }
  }

  function mispronunciation(event) {
    let word = event.detail
    audioPlayer.playJingle2()
    lastMessage = 'MISSPRONUNCIATION: ' + JSON.stringify(word)
  }
</script>

<AudioPlayer bind:this={audioPlayer} />
<Recognition
  bind:this={recognition}
  bind:ttsButtonState
  on:word={processWord}
  on:mispronunciation={mispronunciation}
/>
<div class:hidden={!debug}>
  {ttsButtonState}
</div>

{#if state === 'new'}
  {#if ttsButtonState === 'starting'}{$_(
      'LOADING'
    )}{:else if ttsButtonState === 'ready'}
    <button class="btn btn-primary" on:click={start}>{$_('START')}</button>
  {:else if ttsButtonState === 'error'}
    {$_('ERROR')}
  {/if}
{:else if state === 'progress'}
  <!-- <button class="btn btn-primary" on:click={start}>{$_('RESTART')}</button> -->
  <button class="btn btn-primary" on:click={end}>{$_('END_GAME')}</button>
{:else if state === 'over-win' || state === 'over-loss'}
  <button class="btn btn-primary" on:click={start}>{$_('NEW_GAME')}</button>
{/if}

<div class="container">
  <h3>
    {$_('SCORE')}: {winCounter} | {$_('GAMES')}: {gameCounter} | {$_('TIME')}: {msToTime(
      gameTimer
    )}
  </h3>

  {#if state === 'progress'}
    <progress value={Math.round((pos / wordList.length) * 100)} max="100">
      {Math.round((pos / wordList.length) * 100)}%
    </progress>
    <div class="word-area">
      <div class="word-area-word" style="left: {-20}px; top:{20}px">
        {selectedWord}
      </div>
    </div>

    <div class="">
      {#if lastMessage}
        {lastMessage}
      {/if}
    </div>
    <br />
    <p>{$_('SAY_WORD_PROMPT')}</p>
  {:else if state === 'over-loss'}
    <h2>{$_('GAME_OVER')}</h2>
  {:else if state === 'over-win'}
    <h2>{$_('CONGRATULATIONS')}</h2>
  {/if}
</div>
<br />
<h3
  on:click={() => {
    debug = !debug
  }}
>
  DEBUG
</h3>
{#if debug}
  <div>state:{state}</div>
  <div>time:{msToTime(gameTimer)}</div>
  <div>totalTime:{msToTime(gameTimerTotal)}</div>
  <div>pos:{pos}</div>
  <div>selectedWord:{selectedWord}</div>
  <div>
    wordsSaid:{#key wordsSaid}{JSON.stringify(wordsSaid)}{/key}
  </div>
{/if}

<style>
  .hidden {
    display: none;
  }
  .word-area {
    font-size: xx-large;
    font-weight: 600;
    min-width: 20em;
    min-height: 4em;
    border-radius: 3px;
    border-color: #4679bd;
    border-width: 1px;
    border-style: solid;
  }
  .word-area-word {
    position: relative;
  }

  .badge {
    font-size: large;
  }

  .round-button-circle {
    width: 150px;
    height: 50px;
    border-radius: 10px;
    border: 2px solid #cfdcec;
    overflow: hidden;
    font-weight: 15px;
    background: #4679bd;
    box-shadow: 0 0 3px gold;
  }
  .round-button-circle:hover {
    background: #30588e;
  }
</style>
