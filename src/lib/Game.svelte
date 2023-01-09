<script>
  import AudioPlayer from './AudioPlayer.svelte'
  import Recognition from './Recognition.svelte'
  import { _ } from 'svelte-i18n'

  import { onMount } from 'svelte'

  export let gameLevel

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
      { word: 'bill' },
      { word: 'spoon' },
    ],
    Level1: [
      { word: 'into' },
      { word: 'suit' },
      { word: 'sweat' },
      { word: 'take' },
      { word: 'target' },
      { word: 'just' },
      { word: 'know' },
      { word: 'like' },
      { word: 'because' },
      { word: 'look' },
      { word: 'make' },
      { word: 'can' },
      { word: 'come' },
      { word: 'could' },
      { word: 'day' },
      { word: 'man' },
      { word: 'even' },
      { word: 'find' },
      { word: 'first' },
      { word: 'many' },
    ],
    Level2: [
      { word: 'from' },
      { word: 'give' },
      { word: 'more' },
      { word: 'have' },
      { word: 'new' },
      { word: 'here' },
      { word: 'him' },
      { word: 'get' },
      { word: 'his' },
      { word: 'not' },
      { word: 'how' },
      { word: 'now' },
      { word: 'target' },
      { word: 'tell' },
      { word: 'than' },
      { word: 'that' },
      { word: 'out' },
      { word: 'pear' },
      { word: 'people' },
      { word: 'pour' },
    ],
    Level3: [
      { word: 'about' },
      { word: 'all' },
      { word: 'also' },
      { word: 'and' },
      { word: 'one' },
      { word: 'only' },
      { word: 'other' },
      { word: 'our' },
      { word: 'say' },
      { word: 'see' },
      { word: 'since' },
      { word: 'some' },
      { word: 'with' },
      { word: 'would' },
      { word: 'year' },
      { word: 'you' },
      { word: 'tell' },
      { word: 'than' },
      { word: 'that' },
      { word: 'their' },
    ],
    Level4: [
      { word: 'them' },
      { word: 'then' },
      { word: 'there' },
      { word: 'these' },
      { word: 'they' },
      { word: 'thing' },
      { word: 'think' },
      { word: 'this' },
      { word: 'those' },
      { word: 'who' },
      { word: 'two' },
      { word: 'use' },
      { word: 'very' },
      { word: 'want' },
      { word: 'way' },
      { word: 'well' },
      { word: 'wet' },
      { word: 'what' },
      { word: 'when' },
      { word: 'which' },
    ],
    Level5: [
      { word: 'ability' },
      { word: 'alien' },
      { word: 'angel' },
      { word: 'ballet' },
      { word: 'caffeine' },
      { word: 'calm' },
      { word: 'rind' },
      { word: 'orange' },
      { word: 'chic' },
      { word: 'climb' },
      { word: 'island' },
      { word: 'lettuce' },
      { word: 'onion' },
      { word: 'owl' },
      { word: 'photograph' },
      { word: 'recipe' },
      { word: 'salmon' },
      { word: 'sandwich' },
      { word: 'vowel' },
      { word: 'wolf' },
    ],
    Level6: [
      { word: 'column' },
      { word: 'comb' },
      { word: 'comfortable' },
      { word: 'dessert' },
      { word: 'exercise' },
      { word: 'fruit' },
      { word: 'garage' },
      { word: 'height' },
      { word: 'hotel' },
      { word: 'hour' },
      { word: 'infamous' },
      { word: 'chaos' },
      { word: 'change' },
      { word: 'chorus' },
      { word: 'concert' },
      { word: 'lilac' },
      { word: 'jewelry' },
      { word: 'dinosaur' },
      { word: 'saucy' },
      { word: 'delicious' },
    ],
    Level7: [
      { word: 'Arkansas' },
      { word: 'albeit' },
      { word: 'colonel' },
      { word: 'debris' },
      { word: 'despicable' },
      { word: 'epitome' },
      { word: 'fuchsia' },
      { word: 'heir' },
      { word: 'hyperbole' },
      { word: 'lieutenant' },
      { word: 'lozenge' },
      { word: 'miscellaneous' },
      { word: 'niche' },
      { word: 'pyramid' },
      { word: 'queue' },
      { word: 'subtle' },
      { word: 'zealot' },
      { word: 'Yosemite' },
      { word: 'asterisk' },
      { word: 'brewery' },
    ],
    Level8: [
      { word: 'cavalry' },
      { word: 'deteriorate' },
      { word: 'explicit' },
      { word: 'February' },
      { word: 'library' },
      { word: 'rural' },
      { word: 'sixth' },
      { word: 'specific' },
      { word: 'temperature' },
      { word: 'charcuterie' },
      { word: 'mischievous' },
      { word: 'quinoa' },
      { word: 'accessory' },
      { word: 'turmeric' },
      { word: 'squirrel' },
      { word: 'cacaphony' },
      { word: 'ibuprofen' },
      { word: 'drawers' },
      { word: 'entrepreneur' },
      { word: 'finale' },
    ],
  }

  //let wordList = wordBank['restaurant']
  let wordList

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

  onMount(async () => {})

  function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1))
      ;[array[i], array[j]] = [array[j], array[i]]
    }
  }

  function start() {
    audioPlayer.playJingle3()

    console.log('gameLevel', gameLevel)
    wordList = wordBank[gameLevel]
    gameCounter++
    pos = 0
    state = 'progress'
    shuffleArray(wordList)
    wordList = wordList.slice(0, 5)

    selectedWord = wordList[pos].word

    console.log(111, selectedWord)

    selectedWordDuration = 10000

    gameTimer = 45000

    lastMessage = ''

    clearInterval(gameTimerInterval)
    gameTimerInterval = setInterval(() => {
      gameTimer -= 100
      gameTimerTotal += 100

      if (gameTimer <= 0) {
        end('over-loss')
      }
    }, 100)

    recognition.startRecognition()
  }

  function end(resultState) {
    clearInterval(gameTimerInterval)
    recognition.stopRecognition()
    state = resultState
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
        lastMessage = ''
        end('over-win')
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

  function keyboardPress(zEvent) {
    console.log(zEvent)
    if (zEvent.ctrlKey && zEvent.altKey && zEvent.key === 'E') {
      // case sensitive
      debug = !debug
    }
  }
</script>

<svelte:window on:keydown={keyboardPress} />

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
  <button class="btn btn-primary" on:click={() => end('over-loss')}
    >{$_('END_GAME')}</button
  >
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
