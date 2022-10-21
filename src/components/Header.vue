<script setup lang="ts">
import Pokebola from "../assets/pokebola.png";
</script>
<template>
  <header>
    <img :src="Pokebola" alt="icon header" class="icon-header" />
    <h1>Pokédex</h1>
  </header>
</template>

<style lang="scss">
header {
  color: #f5f5f5;
  text-shadow: 1px 2px 4px #111;
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  background-color: #ed5564;
  padding: 10px 20px;
  .icon-header {
    width: 80px;
  }
}
</style>
