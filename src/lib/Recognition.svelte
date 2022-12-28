<script>
  import { writable } from 'svelte/store'
  import { createEventDispatcher, onMount } from 'svelte'
  import { to_number } from 'svelte/internal'

  const dispatch = createEventDispatcher()

  let ttsToken
  export let ttsButtonState
  let recognizer

  onMount(async () => {
    if (!!window.SpeechSDK) {
      SpeechSDK = window.SpeechSDK
      ttsButtonState = 'starting'
      initPlayPage()
    } else {
      ttsButtonState = 'error'
      console.log('error with SpeechSDK')
    }
  })

  export function startRecognition() {
    fromMic()
  }

  export function stopRecognition(fromTimer = false) {
    ttsButtonState = 'stop'
    if (recognizer) {
      recognizer.stopContinuousRecognitionAsync()
    }
  }

  async function getToken(url = '', data = {}) {
    // Default options are marked with *
    const response = await fetch(url, {
      method: 'GET', // *GET, POST, PUT, DELETE, etc.
      mode: 'cors', // no-cors, *cors, same-origin
      cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
      credentials: 'same-origin', // include, *same-origin, omit
      headers: {
        //'Content-Type': 'application/json'
        'Content-Type': 'application/x-www-form-urlencoded',
        ...data,
      },
      redirect: 'follow', // manual, *follow, error
      referrerPolicy: 'no-referrer', // no-referrer, *no-referrer-when-downgrade, origin, origin-when-cross-origin, same-origin, strict-origin, strict-origin-when-cross-origin, unsafe-url
    })
    console.log('getToken', response)
    //uploadStatus.innerHTML = response.status;
    return response.text() // parses JSON response into native JavaScript objects
  }

  function validateToken(token) {
    var base64Url = token.split('.')[1]
    var base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/')
    var jsonPayload = decodeURIComponent(
      window
        .atob(base64)
        .split('')
        .map(function (c) {
          return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2)
        })
        .join('')
    )

    let part1 = JSON.parse(jsonPayload)

    let timestamp = part1?.exp
    let isValid = timestamp > Math.round(Date.now() / 1000) + 60
    console.log(555, timestamp, isValid)

    return isValid
  }

  function initPlayPage() {
    // getNewToken();
    getToken('https://api.oksensay.com/public/api/azureToken2/1')
      .then((data) => {
        console.log('token', data)
        ttsToken = data
        ttsButtonState = 'ready'
      })
      .catch((error) => {
        console.log('token', error)
        ttsButtonState = 'error'
      })
  }

  async function fromMic() {
    let speechConfig

    if (ttsToken) {
      speechConfig = SpeechSDK.SpeechConfig.fromAuthorizationToken(
        ttsToken,
        'eastasia'
      )
    } else {
      return
    }

    if (validateToken(ttsToken) === false) {
      await getToken()
    }

    speechConfig.speechRecognitionLanguage = 'en-US'
    speechConfig.setProfanity(2)

    var pronunciationAssessmentConfig =
      new SpeechSDK.PronunciationAssessmentConfig(
        '',
        SpeechSDK.PronunciationAssessmentGradingSystem.HundredMark,
        SpeechSDK.PronunciationAssessmentGranularity.Phoneme,
        true
      )

    let audioConfig = SpeechSDK.AudioConfig.fromDefaultMicrophoneInput()
    if (recognizer) {
      recognizer.stopContinuousRecognitionAsync()
    }
    recognizer = new SpeechSDK.SpeechRecognizer(speechConfig, audioConfig)

    pronunciationAssessmentConfig.applyTo(recognizer)

    console.log('Speak into your microphone.')

    recognizer.recognizing = (s, e) => {
      console.log(`RECOGNIZING: Text=${e.result.text}`)
    }

    recognizer.recognized = (s, e) => {
      if (e.result.reason == SpeechSDK.ResultReason.RecognizedSpeech) {
        console.log(`RECOGNIZED: Text=${e.result.text}`)

        let pronunciationAssessmentResult = JSON.parse(e.result.json)

        let part = pronunciationAssessmentResult['NBest'][0]
        let words = part['Words']

        if (words[0]?.PronunciationAssessment?.AccuracyScore > 50) {
          let dispatchWord = words[0]?.Word
          dispatch('word', dispatchWord)
        } else {
          dispatch('mispronunciation', {
            word: words[0]?.Word,
            score: words[0]?.PronunciationAssessment?.AccuracyScore,
          })
        }
      } else if (e.result.reason == SpeechSDK.ResultReason.NoMatch) {
        console.log('NOMATCH: Speech could not be recognized.')
      }
    }

    recognizer.canceled = (s, e) => {
      console.log(`CANCELED: Reason=${e.reason}`)

      if (e.reason == CancellationReason.Error) {
        console.log(`"CANCELED: ErrorCode=${e.errorCode}`)
        console.log(`"CANCELED: ErrorDetails=${e.errorDetails}`)
        console.log('CANCELED: Did you update the subscription info?')
      }

      recognizer.stopContinuousRecognitionAsync()
      getToken()
    }

    ttsButtonState = 'recording'
    recognizer.startContinuousRecognitionAsync()
  }
</script>

<div class="row justify-content-md-center" style="display: none">
  <div style="display:none;" class="col">
    {ttsButtonState} - {ttsToken}
  </div>
  <div class="col col-md-auto">
    {#if ttsButtonState === 'starting'}...
    {:else if ttsButtonState === 'error'}STT Error
    {:else if ttsButtonState === 'ready' || ttsButtonState === 'stop'}
      <button
        class="btn btn-primary btn-lg"
        style="font-size: 32px;"
        on:click={startRecognition}
        ><i
          class="fas fa-microphone-alt pr-3"
          style="font-size: 32px;"
        />Start</button
      >
    {:else if ttsButtonState === 'recording'}
      <button
        class="btn btn-primary btn-lg"
        style="font-size: 32px;"
        on:click={stopRecognition}
        ><i
          class="fas fa-microphone-alt pr-3"
          style="font-size: 32px;"
        />Stop</button
      >
    {/if}
    {#if ttsButtonState === 'stop'}
      Submit
    {/if}
  </div>
</div>
