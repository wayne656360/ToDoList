<script setup>
import { provide } from 'vue'
import ToDoListVue from './components/ToDoList.vue'
import PokerPairVue from './components/PokerPair.vue'
import emitter from './methods/emitter.js';

//provide(()=> emitter)
provide('emitter', emitter)

</script>

<template>
  <ToDoListVue />
  <PokerPairVue />
</template>
