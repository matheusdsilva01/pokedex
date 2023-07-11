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
  <ModalPokeInfo
    :modal-is-open="modalState"
    :open-modal-detail="openModal"
    :close-modal-detail="closeModal"
    :info-poke="pokemon"
    :evolutions="pokemons"
  />
</template>

<script setup lang="ts">
import type { pokemonsType } from "@/types/pokemon";
import { ref } from "vue";
import Loading from "../assets/loading.svg";
import Header from "./Header.vue";
import ModalPokeInfo from "./ModalPokeInfo.vue";

const pokemons = ref<pokemonsType[] | undefined>();
const valueInput = ref<String>("");
const modalState = ref<boolean>(false);
const isLoading = ref<boolean>(false);
const pokemon = ref();

const openModal = () => {
  modalState.value = true;
};

const closeModal = () => {
  modalState.value = false;
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

const getEvolutionFromPoke = async (url: string) => {
  const response = await (await fetch(url)).json();

  const evolutions = await (await fetch(response.evolution_chain.url)).json();

  getEvolutions(evolutions);
};

const getPokeFromName = async (event: Event) => {
  event.preventDefault();
  isLoading.value = true;
  try {
    const response = await (
      await fetch(
        `https://pokeapi.co/api/v2/pokemon/${valueInput.value.toLocaleLowerCase()}`
      )
    ).json();
    pokemon.value = response;
    getEvolutionFromPoke(response.species.url);
  } catch (err) {
    () => alert("Preencha o campo corretamente!");
  } finally {
    isLoading.value = false;
  }
};
</script>

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
