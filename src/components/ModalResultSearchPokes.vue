<template>
  <div class="container-modal" @click="closeModal()">
    <button class="icon-close-modal" @click="closeModal()">
      <img :src="CloseIcon" />
    </button>
    <div class="container-content" @click="(e) => e.stopPropagation()">
      <div class="heading-modal">
        <img :src="Pikachu" alt="icon-heading-modal" />
      </div>
      <section class="container-pokemons">
        <div
          v-for="(poke, index) in props.pokemonsEvolutions"
          @click="openModalDetailPokemon(poke.evolution_name)"
          class="card-pokemon"
          v-bind:key="index"
        >
          <h2>{{ poke.evolution_name }}</h2>
        </div>
      </section>
    </div>
    <ModalPokeInfo
      :modal-is-open="modalOpenDetail"
      :open-modal-detail="openModalDetail"
      :close-modal-detail="closeModalDetail"
      :info-poke="pokemon"
      :evolutions="props.pokemonsEvolutions"
    />
  </div>
</template>

<script setup lang="ts">
import type { Pokemon, pokemonsType } from "@/types/pokemon";
import { ref, type PropType } from "vue";
import CloseIcon from "../assets/close-icon.svg";
import Pikachu from "../assets/pikachu.png";
import ModalPokeInfo from "./ModalPokeInfo.vue";

const props = defineProps({
  modalIsOpen: {
    default: false,
    required: true,
    type: Boolean,
  },
  openModal: {
    required: true,
    type: Function,
  },
  closeModal: {
    required: true,
    type: Function,
  },
  pokemonsEvolutions: {
    required: true,
    type: Object as PropType<pokemonsType[] | undefined>,
  },
});
const modalOpenDetail = ref<boolean>(false);
const pokemon = ref<Pokemon>();
const openModalDetail = () => {
  modalOpenDetail.value = true;
};

const closeModalDetail = () => {
  modalOpenDetail.value = false;
};
const namePokemon = ref<string>("");

const openModalDetailPokemon = (name: string) => {
  namePokemon.value = name;
};

window.addEventListener(
  "keydown",
  (e) => e.key === "Escape" && props.closeModal()
);
</script>

<style lang="scss">
.container-modal {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 50;
  padding: 40px 5px;
  display: flex;
  background-color: #00000025;
  .icon-close-modal {
    background: none;
    border: none;
    cursor: pointer;
    position: absolute;
    right: 5px;
    top: 5px;
  }
  .container-content {
    display: flex;
    flex-direction: column;
    margin: auto;
    background-color: #f5f5f5;
    border: 4px solid #111;
    border-radius: 5px;
    width: 1200px;
    min-height: 800px;
    .heading-modal {
      padding: 5px 10px;
      height: 60px;
      display: flex;
      background-color: #c14450;
      img {
        margin: auto;
        height: 100%;
      }
    }
    .container-pokemons {
      display: flex;
      flex-direction: column;
      row-gap: 20px;
      .card-pokemon {
        width: 100%;
        border: 1px solid #000;
        box-shadow: 0 0 0 2px #fff, 0 0 0 3px #000;
      }
    }
  }
}
</style>
