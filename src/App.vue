<script setup>
// import items from "@/data/items.json";
import items from "@/data/camillemormal.json";
import Slider from "./components/Slider/index.vue";
</script>

<template>
  <Slider :items="items" />
</template>
