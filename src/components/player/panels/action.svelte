<script>
  import { clickedTune } from '../../../stores/tune.store.ts'
  import { playerConfiguration } from '../../../stores/player.store.ts'

  const stop = () => {
    clickedTune.set('')
    $playerConfiguration.loop = false
  }

  const timeoutPromise = timeout => new Promise(resolve => setTimeout(resolve, timeout))

  const resetHiglight = () => {
    playerConfiguration.update(config => {
      config.tunes.map(tune => (tune.enabled = false))
      return config
    })
  }

  const play = async () => {
    resetHiglight()

    for (let i = 0; i < $playerConfiguration.count; i++) {
      // drop highlight from previous tune
      if (i !== 0) {
        $playerConfiguration.tunes[i - 1].enabled = false
      }
      // drop highlight at loop
      else if ($playerConfiguration.loop) {
        $playerConfiguration.tunes[$playerConfiguration.count - 1].enabled = false
      }

      // play tune
      clickedTune.set('')
      clickedTune.set($playerConfiguration.tunes[i].value)

      // add highlight to current tune
      $playerConfiguration.tunes[i].enabled = true

      await timeoutPromise($playerConfiguration.timeout)
    }
  }

  const add = () => {
    $playerConfiguration.count++
    fillEmpty()
  }

  const fillEmpty = () => {
    $playerConfiguration.tunes.push({
      enabled: false,
      value: '',
    })
  }

  const remove = () => {
    $playerConfiguration.count--
    $playerConfiguration.tunes.pop()
  }

  const loop = async () => {
    $playerConfiguration.loop = true
    while ($playerConfiguration.loop) {
      await play()
    }
  }

  const addClickedTune = () => {
    $playerConfiguration.count++
    $playerConfiguration.tunes.push({
      enabled: false,
      value: $clickedTune,
    })
  }

  const reset = () => {
    $playerConfiguration.count = 0
    $playerConfiguration.tunes = []
  }

  const fillEmptyByAction = event => {
    $playerConfiguration.count = parseInt(event.target.value)
    fillEmptyByCount()
  }

  const fillEmptyByCount = () => {
    while ($playerConfiguration.count > $playerConfiguration.tunes.length) {
      fillEmpty()
    }
  }

  const duplicate = () => {
    $playerConfiguration.count += $playerConfiguration.count
    $playerConfiguration.tunes.map(tune => {
      $playerConfiguration.tunes.push({ enabled: false, value: tune.value })
    })
  }
</script>

<style>
  .player__actions > * {
    border: none;
    width: 60px;
    font-family: cursive;
    padding: 5px;
    font-size: 25px;
    text-align: center;
  }
  .player__actions {
    display: flex;
    align-items: center;
    width: 50%;
    height: 50px;
    justify-content: center;
    flex-wrap: wrap;
  }

  .player__actions > :not(:first-child) {
    margin-left: 5px;
  }

  .player__actions span {
    cursor: pointer;
  }

  .current-tune {
    font-size: 42px;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: cursive;
    height: 500px;
    cursor: pointer;
  }
</style>

<div class="player__actions">
  <input on:change={fillEmptyByAction} value={$playerConfiguration.count} />
  <span class="icon-plus" on:click={add} />
  <span class="icon-minus" on:click={remove} />
  <span class="icon-cw" on:click={duplicate} />
  <span class="icon-cancel" on:click={reset} />
  <input bind:value={$playerConfiguration.timeout} />
  <span class="icon-play" on:click={play} />
  <span class="icon-stop" on:click={stop} />
  <span class="icon-loop" on:click={loop} />

  {#if $clickedTune !== ''}
    <span on:click={addClickedTune} class="current-tune">{$clickedTune}</span>
  {/if}
</div>
