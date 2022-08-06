<script setup>
import normalizeWheel from "normalize-wheel";
import { onMounted, ref } from "vue";
import { useEventListener, useRafFn } from "@vueuse/core";
import { clamp, interpolate } from "gsap/all";

import Pagination from "./Pagination.vue";
import Item from "./Item.vue";
import Cross from "./Cross.vue";

defineProps({
  items: {
    type: Array,
    required: true,
  },
});

// References
const x = ref(0);
const positionX = ref(0);
const maxScroll = ref(0);
const activeIndex = ref(0);

// VueComp references
const sliderItems = ref([]);

// Mount and update
onMounted(() => setPosition);

useEventListener(
  window,
  "resize",
  () => {
    x.value = 0;
    resume();

    calcState();
  },
  { passive: true }
);

// Utility functions
const setPosition = (position) => {
  if (!maxScroll.value) {
    calcState();
  }

  x.value = clamp(maxScroll.value * -1, 0, x.value + position * -1);
  resume();
};

const calcState = () => {
  sliderItems.value.forEach(({ calcState }) => calcState());

  const { state } = sliderItems.value.reduce((a, b) =>
    a.state.left > b.state.left ? a : b
  );

  const { left, width } = state;

  maxScroll.value =
    x.value * -1 + (x.value * -1 + left - window.innerWidth / 2 + width / 2);
};

// Dragging logic
const dragging = ref(false);
let dragStart = 0;
let dragXStart = 0;
let triggerIndex = null;

useEventListener(document, "mousedown", (e) => {
  triggerIndex = e.target.closest(".item")?.getAttribute("index");

  dragging.value = true;
  dragStart = e.clientX;
  dragXStart = x.value;
});

useEventListener(document, "mouseup", () => {
  dragging.value = false;
  const distance = Math.abs(x.value - dragXStart);
  const item = sliderItems.value?.[triggerIndex];

  if (item && distance <= 6) {
    handleClick(item, triggerIndex);
  }

  triggerIndex = null;
  dragStart = 0;
});

useEventListener(document, "mousemove", (e) => {
  if (!dragging.value) {
    return;
  }

  const distance = dragStart - e.clientX - (dragXStart - x.value);

  setPosition(distance * 2);
});

const handleClick = (item) => {
  console.log(item);
};

// Scroll logic
useEventListener(window, "wheel", (e) => {
  const normalized = normalizeWheel(e);
  setPosition(normalized.pixelY);
});

// Animate
const { pause, resume } = useRafFn(() => {
  if (!maxScroll.value) {
    return;
  }

  const position = Number(
    interpolate(positionX.value, x.value, 0.08).toFixed(2)
  );

  if (position === positionX.value) {
    pause();
  }

  positionX.value = position;

  const closest = sliderItems.value
    .map((item) => item.progress)
    .reduce((a, b) => {
      return Math.abs(b - 0) < Math.abs(a - 0) ? b : a;
    });

  activeIndex.value = sliderItems.value.findIndex(
    (item) => item.progress === closest
  );
});
</script>

<template>
  <div>
    <Cross />

    <div
      class="flex items-center h-screen"
      :class="{ 'pointer-events-none': dragging }"
    >
      <div
        class="flex items-center justify-start w-full space-x-7 pl-[50vw] select-none"
        :style="{ transform: `translateX(calc(${positionX}px + -9vw))` }"
      >
        <Item
          :key="item.image"
          v-for="(item, index) in items"
          :ref="(comp) => (sliderItems[index] = comp)"
          :item="item"
          :index="index"
          :x="positionX"
        />
      </div>
    </div>

    <Pagination :count="items.length" :active="activeIndex" />
  </div>
</template>
