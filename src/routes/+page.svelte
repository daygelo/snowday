<script lang='ts'>
  import Seal from 'phosphor-svelte/lib/Seal';
  import Minus from 'phosphor-svelte/lib/Minus';
  import Plus from 'phosphor-svelte/lib/Plus';
  import Check from 'phosphor-svelte/lib/Check';
  import X from 'phosphor-svelte/lib/X';
  import SnowmanAlive from '$lib/images/Snowman_Alive.png';
  import SnowmanDead from '$lib/images/Snowman_Dead.png';

  interface TeamData {
    backAtStart: boolean;
    snowmen: [boolean, boolean];
    snowballs: number;
    snowforts: {
      height: number;
      blocks: number;
    }[];
  }

  interface TeamScore {
    total: number;
    backAtStart: number;
    snowmen: number;
    snowballs: number;
    snowforts: number[];
  }

  const defaultTeamData: TeamData = {
    backAtStart: false,
    snowmen: [true, true],
    snowballs: 0,
    snowforts: [
      {
        height: 0,
        blocks: 0,
      },
      {
        height: 0,
        blocks: 0,
      }
    ]
  }

  let teams = $state<TeamData[]>([ defaultTeamData, defaultTeamData ]);
  let scores = $derived<TeamScore[]>(
    teams.map(({ backAtStart, snowmen, snowballs, snowforts }) => {
      const score = {
        backAtStart: backAtStart ? 2 : 0,
        snowmen: snowmen.reduce((a, x) => a + (x ? 10 : 0), 0),
        snowballs,
      };

      const snowfortsScore = snowforts.map(({ height, blocks }) =>
        Math.min(blocks, 6) * (Math.min(Math.floor(height / 6), 3) + 1)
      );

      const total = [ ...Object.values(score), ...snowfortsScore ].reduce((a, x) => a + x, 0);

      return { ...score, snowforts: snowfortsScore, total };
    })
  );

  let currentTeamIndex = $state(0);
  let currentTeam = $derived(teams[currentTeamIndex]);
  let currentScore = $derived(scores[currentTeamIndex]);
</script>

<svelte:head>
  <title>Snowday!</title>
</svelte:head>

<main class={`w-full min-h-screen pb-48 overflow-x-clip ${currentTeamIndex ? 'primary-orange' : 'primary-sky'}`}>
  <h1 class='max-sm:pb-4 py-8 text-3xl sm:text-5xl text-center text-badge-color font-black uppercase'>Snowday!</h1>
  <div class='max-w-100 mx-auto space-y-4 pl-1 pr-2'>
    <section class='flex items-center'>
      <div class='relative flex justify-center items-center size-12 -mr-5'>
        <Seal class='absolute size-12 text-badge-color' weight='fill'/>
        <span class='text-2xl text-white font-black z-10'>{currentScore.backAtStart}</span>
      </div>
      <div class='w-full p-4 bg-white border-2 border-white shadow-ice rounded-2xl'>
        <p class='mb-2 text-center text-label-color font-bold'>Both robots back at starting zone?</p>
        <button
          class={`flex justify-center items-center gap-2 w-full p-1 mx-auto text-lg font-bold rounded-xl shadow-inner ${currentTeam.backAtStart ? 'bg-green-200 shadow-green-300' : 'bg-red-200 shadow-red-300'}`}
          onclick={() => currentTeam.backAtStart = !currentTeam.backAtStart}
        >
          {#if currentTeam.backAtStart}
            <Check weight='bold'/> Yeah!
          {:else}
            <X weight='bold'/> No...
          {/if}
        </button>
      </div>
    </section>

    <section class='flex items-center'>
      <div class='relative flex justify-center items-center size-12 -mr-5'>
        <Seal class='absolute size-12 text-badge-color' weight='fill'/>
        <span class='text-2xl text-white font-black z-10'>{currentScore.snowmen}</span>
      </div>
      <div class='w-full p-4 pb-0 bg-white border-2  border-white shadow-ice rounded-2xl'>
        <p class='mb-2 text-center text-label-color font-bold'>Snowmen standing?</p>
        <div class='flex justify-evenly'>
          {#each currentTeam.snowmen as _, i}
            <button onclick={() => currentTeam.snowmen[i] = !currentTeam.snowmen[i]}>
              <img
                class={`w-20 transition ease-bounce origin-bottom ${currentTeam.snowmen[i] ? '' : 'rotate-60'}`}
                src={currentTeam.snowmen[i] ? SnowmanAlive : SnowmanDead}
                alt='snowman'
              />
            </button>
          {/each}
        </div>
      </div>
    </section>

    <section class='flex items-center'>
      <div class='relative flex justify-center items-center size-12 -mr-5'>
        <Seal class='absolute size-12 text-badge-color' weight='fill'/>
        <span class='text-2xl text-white font-black z-10'>{currentScore.snowballs}</span>
      </div>
      <div class='w-full p-4 bg-white border-2 border-white shadow-ice rounded-2xl'>
        <p class='mb-2 text-center text-label-color font-bold'>Snowballs on the <strong>OPPOSING</strong> side</p>
        <div class='flex justify-center items-center'>
          <button class='p-1 rounded-full shadow-inner shadow-sky-100 hover:bg-sky-100' onclick={() => currentTeam.snowballs--} disabled={currentTeam.snowballs === 0}>
            <Minus class='text-label-color' weight='bold'/>
          </button>
          <p class='w-18 text-3xl text-center font-bold'>{currentTeam.snowballs}</p>
          <button class='p-1 rounded-full shadow-inner shadow-sky-100 hover:bg-sky-100' onclick={() => currentTeam.snowballs++}>
            <Plus class='text-label-color' weight='bold'/>
          </button>
        </div>
      </div>
    </section>

    {#each currentTeam.snowforts as { height, blocks }, i}
      <section class='flex items-center'>
        <div class='relative flex justify-center items-center size-12 -mr-5'>
          <Seal class='absolute size-12 text-badge-color' weight='fill'/>
          <span class='text-2xl text-white font-black z-10'>{currentScore.snowforts[i]}</span>
        </div>
        <div class='w-full p-4 bg-white border-2 border-white shadow-ice rounded-2xl'>
          <p class='mb-2 text-center text-label-color font-bold'>Snowfort {i + 1}</p>
          <div class='grid grid-cols-2'>
            <div>
              <p class='text-center text-label-color font-medium'>Height</p>
              <div class='flex justify-center items-center'>
                <button class='p-1 rounded-full shadow-inner shadow-sky-100 hover:bg-sky-100' onclick={() => currentTeam.snowforts[i].height -= 3} disabled={currentTeam.snowforts[i].height === 0}>
                  <Minus class='text-label-color' weight='bold'/>
                </button>
                <p class='w-18 text-3xl text-center font-bold'>{height}"</p>
                <button class='p-1 rounded-full shadow-inner shadow-sky-100 hover:bg-sky-100' onclick={() => currentTeam.snowforts[i].height += 3}>
                  <Plus class='text-label-color' weight='bold'/>
                </button>
              </div>
            </div>
            <div>
              <p class='text-center text-label-color font-medium'>Blocks</p>
              <div class='flex justify-center items-center'>
                <button class='p-1 rounded-full shadow-inner shadow-sky-100 hover:bg-sky-100' onclick={() => currentTeam.snowforts[i].blocks--} disabled={currentTeam.snowforts[i].blocks === 0}>
                  <Minus class='text-label-color' weight='bold'/>
                </button>
                <p class='w-14 text-3xl text-center font-bold'>{blocks}</p>
                <button class='p-1 rounded-full shadow-inner shadow-sky-100 hover:bg-sky-100' onclick={() => currentTeam.snowforts[i].blocks++}>
                  <Plus class='text-label-color' weight='bold'/>
                </button>
              </div>
            </div>
          </div>
        </div>
      </section>
    {/each}
  </div>
</main>

<footer class='fixed bottom-0 grid grid-cols-2 w-full py-2 bg-black divide-x divide-gray-800 z-10'>
  <button class={`px-8 py-4 text-right ${currentTeamIndex === 0 ? '' : 'opacity-75 translate-y-1'}`} onclick={() => currentTeamIndex = 0}>
    <p class='text-6xl text-white font-black'>{scores[0].total}</p>
    <p class='text-xl text-sky-400 font-bold'>Team 1</p>
  </button>
  <button class={`px-8 py-4 text-left ${currentTeamIndex === 1 ? '' : 'opacity-75 translate-y-1'}`} onclick={() => currentTeamIndex = 1}>
    <p class='text-6xl text-white font-black'>{scores[1].total}</p>
    <p class='text-xl text-orange-400 font-bold'>Team 2</p>
  </button>
</footer>