<script context="module" lang="ts">
  import mainLoop from 'mainloop.js'

  const functions = {
    begin: [],
    draw: [],
    update: [],
    end: []
  }

  function draw(interpolationPercentage) {
    functions.draw.forEach(drawFn => {
			drawFn(interpolationPercentage)
		})
  }

  function run(stage, param) {
    functions[stage].forEach(fn => {
      fn(param)
    })
  }

  function addFunction(stage, fn) {
    if (functions[stage].indexOf(fn) === -1) functions[stage].push(fn);
    return functions[stage].length;
  }

  function removeFunction(stage, fn) {
    functions[stage].splice(functions[stage].indexOf(fn), 1)
    return functions[stage].length;
  }

  mainLoop.setBegin((timestamp) => run('begin', timestamp))
  mainLoop.setDraw((interp) => run('draw', interp))
  mainLoop.setUpdate((delta) => run('update', delta))
  mainLoop.setEnd((fps, panic) => run('end', [fps, panic]))

  function getLength(stage) {
    return functions[stage].length;
  }
</script>

<script>
  import { setContext } from 'svelte'

  let isVisible = true;
  let timestamp = 0;
  let frame = 0
  let tick = 0
  let isRunning = false;
  let fps = 0;
  let lastInterp = 0;
  let lastDelta = 0;
  let panic = "don't";
  let lastTimestamp = 0;
  let lastAbsence = 0;
  let checkAway = false;

  const lengths = {
    begin: 0,
    draw: 0,
    update: 0,
    end: 0
  }

  function begin(ptimestamp) {
    timestamp = ptimestamp;
    if (checkAway) {
      lastAbsence = timestamp - lastTimestamp;
      checkAway = false;
    }
  }

  function draw(interp) {
    lastInterp = interp;
    frame++
  }

  function update(delta) {
    tick++
    lastDelta = delta;
  }

  function end(params) {
    const [pfps, ppanic] = params
    fps = pfps;
    panic = ppanic;
  }

  register('begin', begin)
  register('draw', draw)
  register('update', update)
  register('end', end)

  function start() {
    mainLoop.start()
    requestAnimationFrame(() => isRunning = true);
  }

  function stop() {
    isRunning = false;
    mainLoop.stop();
    lastTimestamp = timestamp;
    checkAway = true;
  }

  function reset() {
    stop();
    frame = 1;
    tick = 1;
  }

  const getFrame = () => frame;
  const getTimestamp = () => timestamp;

  function register(stage, fn) {
    lengths[stage] = addFunction(stage, fn);
  }

  function unregister(stage, fn) {
    lengths[stage] = removeFunction(stage, fn);
  }

  setContext('loop', { register, unregister, getFrame, getTimestamp })
</script>

<section>
<button on:click={() => isVisible = !isVisible}>{ isVisible ? "Hide Loop Controls" : "Show Loop Controls"}</button><br />
  {#if isVisible}
  { isRunning ? "Running" : "Paused"} |
  FPS: {fps.toFixed(0)} |
  Frame: {frame}|
  Tick: {tick}<br />
  Timestamp: {timestamp} |
  Last Timestamp: {lastTimestamp} |
  Last Absence: {lastAbsence}<br />
  Last Delta: {lastDelta.toFixed(2)}ms | 
  Interpolation: {lastInterp.toFixed(2)}% |
  Panic: {panic} <br />
  {#key isRunning}
  {#if isRunning }
  <button on:click={stop}>Stop</button>
  {:else}
  <button on:click={start}>Start</button>
  {/if}
  <button on:click={reset}>Reset</button>
  {/key}
  <button on:click={() => numDrawFns = register('draw', draw)}>Register</button>
  <button on:click={() => numDrawFns = unregister('draw', draw)}>Unregister</button>
  <ul>
  {#each Object.entries(lengths) as [stage, length]}
    <li>
      # {stage}: {length}
    </li>
  {/each}
  </ul>
  {/if}
</section>

<slot />

<style>
  ul {
    list-style-type: none;
    text-indent: none;
  }
</style>