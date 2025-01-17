<template>
  <div class="flex w-screen h-screen transition-all duration-500 bg-gradient-to-tl bg-size-200" :class="[ pokeColors[pokemon?.color ?? 'default'].gradient ]">
    <div v-if="pokemon && pokemon.bgImage" class="absolute intro-opac-30 inset-0 z-0 top-0 left-[-10rem] h-screen w-screen overflow-hidden opacity-30 blur-sm">
      <img :src="pokemon.bgImage" alt="pokemon" class="h-full transform scale-150"/>
    </div>
    <!-- We've used 3xl here, but feel free to try other max-widths based on your needs -->
    <div class="relative w-full max-w-7xl h-3/4 p-4 m-auto shadow-lg rounded-lg transition-colors bg-white/40 backdrop-blur-lg">
      <div v-if="isSearching" class="pokeball-wrapper transition-opacity duration-300" :class="[isSearching ? 'opacity-1' : 'opacity-0']">
        <div class="pokeball"></div>
      </div>
      <div class="absolute flex gap-3 left-0 -top-[3rem] ">
        <input v-model="pokemonInput" name="pokemon" class="w-[15rem] block bg-white/50 rounded-md border-0 py-1.5 px-3 pr-12 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-slate-800 sm:text-sm sm:leading-6" type="text" placeholder="Search Pokemon Name or ID" @keyup.enter="searchPokemon">
        <button type="button" class="rounded-md bg-white/50 px-2.5 py-1.5 text-sm font-semibold text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 hover:bg-white/60" @click.prevent="searchRandom">Random</button>
      </div>
      <div class="pointer-event-none">
        <h1 class="absolute top-4 font-bebas w-full overflow-hidden text-white/10 text-[18vw] leading-[0.8]" style="height: calc(100% - 32px);">
          {{ pokemon?.name }}
        </h1>
        <div class="relative z-50">
          <h1 class="mt-[7rem] font-bebas w-full text-[10vw] leading-[0.8] drop-shadow-light" :class="[ pokeColors[pokemon?.color ?? 'default'].dark ]">
            {{ pokemon?.name }}
          </h1>
          <p v-if="pokemon" class="font-bebas -mt-[1.5rem] text-[4vw] text-neutral-800">
            #{{ pokemon?.id }}
          </p>
        </div>
        <div class="z-10 fixed pointer-events-none h-full max-w-4xl bottom-0 right-0 pt-6 scale-125">
          <img v-if="pokemon" :src="pokemon.image" alt="pokemon" class="intro-y h-full w-full object-contain object-bottom drop-shadow-xl" :class="[{'brightness-0': !pokemon.hasImage}]"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const pokemonInput = ref('');
const pokemon = ref(null);
const isSearching = ref(false);
const notFound = ref(false);

const pokeColors = {
  black: { default: "bg-pokeblack", light: "bg-pokeblack-light/50", gradient: "from-pokeblack-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokeblack-dark' },
  blue: { default: "bg-pokeblue", light: "bg-pokeblue-light/50", gradient: "from-pokeblue-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokeblue-dark' },
  brown: { default: "bg-pokebrown", light: "bg-pokebrown-light/50", gradient: "from-pokebrown-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokebrown-dark' },
  gray: { default: "bg-pokegray", light: "bg-pokegray-light/50", gradient: "from-pokegray-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokegray-dark' },
  green: { default: "bg-pokegreen", light: "bg-pokegreen-light/50", gradient: "from-pokegreen-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokegreen-dark' },
  pink: { default: "bg-pokepink", light: "bg-pokepink-light/50", gradient: "from-pokepink-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokepink-dark' },
  purple: { default: "bg-pokepurple", light: "bg-pokepurple-light/50", gradient: "from-pokepurple-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokepurple-dark' },
  red: { default: "bg-pokered", light: "bg-pokered-light/50", gradient: "from-pokered-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokered-dark' },
  white: { default: "bg-pokewhite", light: "bg-pokewhite-light/50", gradient: "from-pokewhite-light via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokewhite-dark' },
  yellow: { default: "bg-pokeyellow", light: "bg-pokeyellow-light/50", gradient: "from-pokeyellow-dark via-neutral-700 to-neutral-900 bg-pos-100", dark: 'text-pokeyellow-dark' },
  default: { default: "bg-slate-500", light: "bg-slate-300", gradient: "from-neutral-600 via-neutral-700 to-neutral-900 bg-pos-0", dark: 'text-slate-700' }
}

onMounted(async () => {
  await nextTick();
  searchRandom();
});

const getRandomNumber = () => {
  const range1 = Math.floor(Math.random() * (1017 - 1 + 1)) + 1; // Random number between 1 and 1000
  const range2 = Math.floor(Math.random() * (10292 - 10001 + 1)) + 10000; // Random number between 10000 and 10227

  // Randomly choose between the two ranges
  const isRange1 = Math.random() < 0.9;
  
  return isRange1 ? range1 : range2;
}

const searchRandom = async () => {
  const randomPokemonNumber = getRandomNumber();
  pokemonInput.value = randomPokemonNumber;
  await searchPokemon();
}

const searchPokemon = async () => {
  pokemon.value = null;
  isSearching.value = true;
  notFound.value = false;

  const transformedInput = transformInput(`${pokemonInput.value}`);
  const { data, error } = await useFetch(`/api/${transformedInput}`)

  await setTimeout(() => {
    pokemon.value = data.value;

    if (error.value) {
      notFound.value = true;
    }

    isSearching.value = false;
  }, 500);
}

const transformInput = (input) => {
  const lowercased = input.toLowerCase();
  const kebabCased = lowercased.replace(/[^a-z0-9]+/g, '-');
  return kebabCased.replace(/^-+|-+$/g, '');
}
</script>