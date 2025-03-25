<script setup lang="ts">
import { ref, useTemplateRef } from "vue";

const container = useTemplateRef("container");

const events = ref<number[]>([0]);
const currentEventIndex = ref(0);

const debounce = <T>(callback: (args?: T) => void, ms: number) => {
  let id: undefined | number;

  return (...args: any) => {
    clearTimeout(id);

    id = setTimeout(() => {
      callback(...args);
    }, ms);
  };
};

const changeIndex = () => {
  events.value.push(0);
  currentEventIndex.value += 1;
};

const changeIndexDebounced = debounce(changeIndex, 500);

const handleWheel = (event: WheelEvent) => {
  const currentEventIndexValue = events.value[currentEventIndex.value];

  if (currentEventIndexValue > 500) {
    changeIndex();
    return;
  }

  if (currentEventIndexValue < 0 && event.deltaY > 0) {
    changeIndex();
    return;
  } else if (currentEventIndexValue > 0 && event.deltaY < 0) {
    changeIndex();
    return;
  }

  events.value[currentEventIndex.value] += event.deltaY / 50;
  changeIndexDebounced();

  container.value?.scrollTo({
    left: container.value.scrollWidth,
    behavior: 'smooth',
  })
};
</script>

<template>
  <main
    ref="container"
    @wheel="handleWheel"
    class="flex items-center min-h-screen overflow-y-hidden"
  >
    <div
      class="gap-4 relative h-1 overflow-visible"
      :style="{
        width: `${events.length * 80}px`,
      }"
    >
      <div
        v-for="(_, i) in events"
        :key="i"
        class="absolute w-20 shrink-0"
        :style="{
          height: `${Math.abs(events[i])}px`,
          left: `${(80 + 4) * i}px`,
          ...(events[i] < 0
            ? {
                bottom: 0,
                backgroundColor: 'var(--color-green-500)',
              }
            : {
                top: 0,
                backgroundColor: 'var(--color-red-500)',
              }),
        }"
      />
    </div>
  </main>
</template>
