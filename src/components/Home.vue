<script setup lang="ts">
import type { Pokemon } from "@/types/pokemon";
import { ref } from "vue";
import Header from "./Header.vue";
import ModalPokeInfo from "./ModalPokeInfo.vue";
import Loading from "../assets/loading.svg";

const pokemon = ref<Pokemon | undefined>();
const valueInput = ref<String>("");
const modalOpen = ref<boolean>(false);
const isLoading = ref<boolean>(false);

const openModal = () => {
  modalOpen.value = true;
};

const closeModal = () => {
  modalOpen.value = false;
};

const getPokeFromName = async (event: Event) => {
  event.preventDefault();
  isLoading.value = true;

  await fetch(
    `https://pokeapi.co/api/v2/pokemon/${valueInput.value.toLocaleLowerCase()}`
  )
    .then(response => response.json())
    .then(data => {
      pokemon.value = data;
      openModal();
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
  <ModalPokeInfo
    :modal-is-open="modalOpen"
    :open-modal="openModal"
    :close-modal="closeModal"
    :info-poke="pokemon"
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
