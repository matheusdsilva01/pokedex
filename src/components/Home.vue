<script setup lang="ts">
import type { pokemonsType } from "@/types/pokemon";
import { ref } from "vue";
import Loading from "../assets/loading.svg";
import Header from "./Header.vue";
import ModalResultSearchPokes from "./ModalResultSearchPokes.vue";

const pokemons = ref<pokemonsType[] | undefined>();
const valueInput = ref<String>("");
const modalOpen = ref<boolean>(false);
const isLoading = ref<boolean>(false);

const openModal = () => {
  modalOpen.value = true;
};

const closeModal = () => {
  modalOpen.value = false;
};

const getEvolutionFromPoke = async (url: string) => {
  await fetch(url)
    .then(response => response.json())
    .then(async data => {
      await fetch(data.evolution_chain.url)
        .then(response => response.json())
        .then(data => getEvolutions(data));
    });
};

const getEvolutions = async (data: any) => {
  let evoChain = [];
  let evoData = data.chain;

  do {
    let numberOfEvolutions = evoData["evolves_to"].length;

    evoChain.push({
      evolution_name: !evoData ? null : evoData.species.name,
      url_poke: !evoData ? null : evoData.species.url
    });

    if (numberOfEvolutions > 1) {
      for (let i = 1; i < numberOfEvolutions; i++) {
        evoChain.push({
          evolution_name: !evoData.evolves_to[i]
            ? null
            : evoData.evolves_to[i].species.name,
          url_poke: !evoData.evolves_to[i]
            ? null
            : evoData.evolves_to[i].species.url
        });
      }
    }

    evoData = evoData["evolves_to"][0];
  } while (!!evoData && evoData.hasOwnProperty!("evolves_to"));
  pokemons.value = evoChain;
  openModal();
};

const getPokeFromName = async (event: Event) => {
  event.preventDefault();
  isLoading.value = true;

  await fetch(
    `https://pokeapi.co/api/v2/pokemon/${valueInput.value.toLocaleLowerCase()}`
  )
    .then(response => response.json())
    .then(data => {
      getEvolutionFromPoke(data.species.url);
    })
    .catch(() => alert("Preencha o campo corretamente"));
  isLoading.value = false;
};
</script>

<template>
  <Header />
  <div class="container">
    <form class="form" v-on:submit="getPokeFromName($event)">
      <div class="input-receptor">
        <label for="input-search" class="label-input-search">
          Search Pokemon for name:
        </label>
        <input
          id="input-search"
          required
          type="text"
          v-model="valueInput"
          class="input-search"
        />
        <img v-if="isLoading" id="icon-loading" :src="Loading" />
      </div>
    </form>
  </div>
  <ModalResultSearchPokes
    :modal-is-open="modalOpen"
    :open-modal="openModal"
    :close-modal="closeModal"
    :pokemons-evolutions="pokemons"
  />
</template>

<style lang="scss">
.container {
  margin: auto;
  background-color: #f84b4b;
  border-radius: 4px;
  border: 4px solid #111;
  box-shadow: 5px 5px 5px #000;
  width: 100%;
  max-width: 800px;
  height: 500px;
  form {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
    .input-receptor {
      color: #f5f5f5;
      display: flex;
      flex-direction: column;
      label {
        font-size: 24px;
      }
      input {
        padding: 10px 5px;
        width: 100%;
        border-radius: 4px;
        border: 2px solid #111;
        outline: none;
        &:hover {
          box-shadow: 0 0 5px #f5f5f5;
        }
        &:active,
        &:focus {
          box-shadow: 0 0 5px 2px #f5f5f5;
        }
      }
      #icon-loading {
        width: 50px;
        margin: auto;
        animation: infinite rotate-infinite 0.8s linear;
      }
    }
  }
}
@keyframes rotate-infinite {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
