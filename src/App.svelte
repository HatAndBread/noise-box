<script lang="ts">
  import { Noise, AutoFilter, Volume, start } from "tone";
  type Noises = "pink" | "brown" | "white";
  const NOISE_TYPES: Noises[] = ["brown", "pink", "white"];
  let started = false;
  let currentlyPlaying: Noises[] = [];

  const instruments = {};
  const autoFilterDefaults = {
    frequency: 0,
    baseFrequency: 10,
    octaves: 8,
  };
  NOISE_TYPES.forEach((noiseType) => {
    const n = new Noise(noiseType);
    n.volume.value = 0;
    const f = new AutoFilter(autoFilterDefaults).toDestination().start();
    n.connect(f);
    instruments[noiseType] = n;
    instruments[`${noiseType}Filter`] = f;
  });
  const getFilter = (noise: Noises) =>
    instruments[`${noise}Filter`] as AutoFilter;
  const getNoise = (noise: Noises) => instruments[noise] as Noise;
  const adjustFrequency = (noise: Noises, amount: string) => {
    getFilter(noise).frequency.rampTo(amount, 0.1);
  };
  const adjustBaseFrequency = (noise: Noises, amount: string) => {
    getFilter(noise).baseFrequency = amount;
  };
  const adjustOctaves = (noise: Noises, amount: string) => {
    getFilter(noise).octaves = parseFloat(amount);
  };
  const adjustPlaybackRate = (noise: Noises, amount: string) => {
    getNoise(noise).playbackRate = parseFloat(amount);
  };
  const adjustVolume = (noise: Noises, amount: string) => {
    getNoise(noise).volume.rampTo(parseFloat(amount));
  };
  const handleClick = async (noise: Noises) => {
    if (!started) {
      started = true;
      await start();
    }
    if (currentlyPlaying.includes(noise)) {
      handleStop(noise);
    } else {
      currentlyPlaying.push(noise);
      currentlyPlaying = currentlyPlaying;
      getNoise(noise).start();
      getFilter(noise).start();
    }
  };
  const handleStop = (noise: Noises) => {
    currentlyPlaying = currentlyPlaying.filter((n) => n !== noise);
    getNoise(noise).stop();
  };
</script>

<main class="flex flex-col h-screen">
  {#each NOISE_TYPES as n}
    <div
      class="p-2 border-[10px] bg-opacity-40 grow flex flex-col justify-center"
      class:bg-yellow-800={n === "brown"}
      class:border-yellow-800={n === "brown"}
      class:bg-pink-300={n === "pink"}
      class:border-pink-300={n === "pink"}
      class:bg-neutral-200={n === "white"}
      class:border-neutral-200={n === "white"}
    >
      <button
        class="btn btn-neutral w-full mb-2 btn-sm"
        on:click={() => handleClick(n)}
      >
        <span class="relative flex h-3 w-3">
          {#if currentlyPlaying.includes(n)}
            <span
              class="animate-ping absolute inline-flex h-full w-full rounded-full bg-green-400 opacity-75"
            />
          {/if}
          <span
            class="relative inline-flex rounded-full h-3 w-3 bg-green-500"
            class:bg-red-700={!currentlyPlaying.includes(n)}
          />
        </span>
        {currentlyPlaying.includes(n) ? "stop" : "start"}
      </button>
      <div class="flex items-center w-full my-2">
        <span class="w-14 text-xs"> Freq </span>
        <input
          type="range"
          min="0"
          max="100"
          value={autoFilterDefaults.frequency}
          step="0.1"
          class="range range-xs ml-2"
          on:input={(e) => adjustFrequency(n, e.currentTarget.value)}
        />
      </div>
      <div class="flex items-center w-full my-2">
        <span class="w-14 text-xs"> Base </span>
        <input
          type="range"
          min="0"
          max="100"
          value={autoFilterDefaults.baseFrequency}
          step="0.1"
          class="range range-xs"
          on:input={(e) => adjustBaseFrequency(n, e.currentTarget.value)}
        />
      </div>
      <div class="flex items-center w-full my-2">
        <span class="w-14 text-xs"> Octave </span>
        <input
          type="range"
          min="0"
          max="10"
          value={autoFilterDefaults.octaves}
          step="0.05"
          class="range range-xs"
          on:input={(e) => adjustOctaves(n, e.currentTarget.value)}
        />
      </div>
      <div class="flex items-center w-full my-2">
        <span class="w-14 text-xs"> Pitch </span>
        <input
          type="range"
          min="1"
          max="600"
          value="1"
          step="0.05"
          class="range range-xs"
          on:input={(e) => adjustPlaybackRate(n, e.currentTarget.value)}
        />
      </div>
      <div class="flex items-center w-full my-2">
        <span class="w-14 text-xs"> Volume </span>
        <input
          type="range"
          min="-20"
          max="20"
          value="0"
          step="1"
          class="range range-xs"
          on:input={(e) => adjustVolume(n, e.currentTarget.value)}
        />
      </div>
    </div>
  {/each}
</main>
