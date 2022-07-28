<script setup>
import { gsap } from "gsap/all";
import { watch, ref } from "vue";

const props = defineProps({
  count: {
    type: Number,
    required: true,
  },
  active: {
    type: Number,
    required: true,
  },
});

const $pagination = ref(null);

watch(
  () => props.active,
  (index) => {
    gsap.killTweensOf($pagination.value);

    gsap.to($pagination.value, {
      y: `-${24 * index}px`,
    });
  }
);
</script>

<template>
  <div
    class="fixed bottom-[4vh] left-1/2 -translate-x-1/2 text-white flex space-x-2 items-center"
  >
    <div class="h-[24px] overflow-hidden">
      <div ref="$pagination" class="flex flex-col">
        <span :key="`pagination-${i}`" v-for="i in count">{{ i }}</span>
      </div>
    </div>
    <div class="w-3 h-px bg-white"></div>
    <div>{{ count }}</div>
  </div>
</template>
