<script setup>
import { nextTick, onMounted, ref, watch } from "vue-demi";
import { interpolate } from "gsap/all";

const props = defineProps({
  item: {
    type: Object,
    required: true,
  },
  x: {
    type: Number,
    required: true,
  },
});

const $el = ref(null);
const progress = ref(0);
const state = ref(null);

const calcState = () => {
  const { left, width } = $el.value.getBoundingClientRect();

  state.value = {
    left,
    width,
  };

  calculateProgress();
};

const calculateProgress = () => {
  const x = state.value.left + props.x;

  progress.value = interpolate(
    -1,
    1,
    (1 / window.innerWidth) * (x + state.value.width * 0.5)
  );
};

onMounted(async () => {
  await nextTick();

  calcState();
});

watch(() => props.x, calculateProgress);

defineExpose({
  calcState,
  state,
  progress,
});
</script>

<template>
  <div ref="$el" :style="{ transform: `translateX(${x}px)` }" class="item">
    <div class="flex w-[18vw] h-[25.5vw] overflow-hidden">
      <div class="w-[56vw] h-[56vh] overflow-hidden">
        <img
          v-bind:progress="progress"
          class="w-full h-full object-cover"
          :style="{
            objectPosition: `${50 + 30 * progress}% 50%`,
          }"
          :src="item.image"
        />
      </div>
    </div>
  </div>
</template>
