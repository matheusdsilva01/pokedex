<script setup lang="ts">
import type { Pokemon } from "@/types/pokemon";
import { ref, watchEffect, type PropType } from "vue";
import CloseIcon from "../assets/close-icon.svg";
import Pikachu from "../assets/pikachu.png";

type evolutionsType = {
  evolution_name: string;
};

const props = defineProps({
  modalIsOpen: {
    default: false,
    required: true,
    type: Boolean
  },
  openModal: {
    required: true,
    type: Function
  },
  closeModal: {
    required: true,
    type: Function
  },
  infoPoke: {
    required: true,
    type: Object as PropType<Pokemon | undefined>
  }
});

const evolutions = ref<evolutionsType[] | []>([]);
const typesPokemon = ref<string>();
const getTypesPoke = () => {
  typesPokemon.value = props.infoPoke?.types
    .map(type => type.type.name)
    .join(", ");
};

const getTotalValuesStats = () => {
  const valuesStats = props.infoPoke?.stats.map(stat => stat.base_stat);
  return valuesStats?.reduce((prev, current) => prev + current, 0);
};

const getEvolution = (data: any) => {
  let evoChain = [];
  let evoData = data.chain;

  do {
    let numberOfEvolutions = evoData["evolves_to"].length;

    evoChain.push({
      evolution_name: !evoData ? null : evoData.species.name
    });

    if (numberOfEvolutions > 1) {
      for (let i = 1; i < numberOfEvolutions; i++) {
        evoChain.push({
          evolution_name: !evoData.evolves_to[i]
            ? null
            : evoData.evolves_to[i].species.name
        });
      }
    }

    evoData = evoData["evolves_to"][0];
  } while (!!evoData && evoData.hasOwnProperty!("evolves_to"));
  evolutions.value = evoChain;
};

window.addEventListener(
  "keydown",
  e => e.key === "Escape" && props.closeModal()
);

const getEvolutionPoke = async (url: string) => {
  await fetch(url)
    .then(response => response.json())
    .then(data => getEvolution(data));
};

watchEffect(() => {
  getTypesPoke();
  props.infoPoke?.id &&
    fetch(`https://pokeapi.co/api/v2/pokemon-species/${props.infoPoke.id}/`)
      .then(response => response.json())
      .then(data => getEvolutionPoke(data.evolution_chain.url));
});
</script>

<template>
  <div v-if="modalIsOpen" class="container-modal" @click="closeModal()">
    <button class="icon-close-modal" @click="closeModal()">
      <img :src="CloseIcon" />
    </button>
    <div class="container-content" @click="e => e.stopPropagation()">
      <div class="heading-modal">
        <img :src="Pikachu" alt="icon-heading-modal" />
      </div>
      <div class="content-info-poke">
        <div class="background-cover-image">
          <img
            :src="infoPoke?.sprites.other?.dream_world.front_default"
            :alt="infoPoke?.name"
          />
        </div>
        <div class="content-principal-poke">
          <p><strong>Name: </strong>{{ infoPoke?.name }}</p>
          <p><strong>Weight: </strong>{{ infoPoke?.weight }}'</p>
          <p><strong>Type(s): </strong>{{ typesPokemon }}</p>
          <p v-if="evolutions.length > 1">
            <strong>Evolutions(s): </strong
            >{{ evolutions.map(e => e.evolution_name).join(", ") }}
          </p>
          <p v-else><strong>Evolutions(s): </strong>no contain evolution</p>
        </div>
      </div>
      <div class="stats-poke">
        <ul>
          <li v-for="stats in infoPoke?.stats" v-bind:key="stats.stat.name">
            <strong>{{ stats.stat.name.toUpperCase() }}: </strong
            >{{ stats.base_stat }}
          </li>
          <li><strong>TOTAL: </strong>{{ getTotalValuesStats() }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.container-modal {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 100;
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
    .content-info-poke {
      width: 100%;
      display: flex;
      .background-cover-image {
        display: flex;
        width: 30%;
        height: 300px;
        border-bottom-right-radius: 60%;
        background-color: #ed5564;
        img {
          max-width: 175px;
          margin: auto;
        }
      }
      .content-principal-poke {
        width: 50%;
        padding: 10px 20px;
        font-size: 26px;
      }
    }
  }
  .stats-poke {
    margin: 5px 10px;
    border: 3px solid #000;
    box-shadow: 0 0 0 2px #fff, 0 0 0 5px #000;
    margin-top: 20px;
    display: block;
    height: 100%;
    margin-bottom: 10px;
    ul {
      flex-direction: column;
      display: flex;
      row-gap: 20px;
    }
    li {
      list-style: none;
      padding: 2px 5px;
      color: #fff;
      &:nth-child(odd) {
        background-color: #111;
      }
      &:nth-child(even) {
        background-color: #c14450;
      }
    }
  }
}

@media (max-width: 769px) {
  .container-modal {
    .container-content {
      .content-info-poke {
        flex-direction: column;
        .content-principal-poke {
          font-size: 18px;
        }
        .background-cover-image {
          width: 100%;
          height: 230px;
          border-bottom-right-radius: 25px;
          border-bottom-left-radius: 25px;
        }
      }
    }
    .stats-poke {
      font-size: 14px;
    }
  }
}
</style>
