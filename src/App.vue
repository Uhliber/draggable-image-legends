<script setup>
import { ref, onMounted, reactive, computed } from 'vue';
import { Droppable } from '@shopify/draggable';

const posX = ref(0);
const posY = ref(0);
const elementPosX = ref(0);
const elementPosY = ref(0);
const droppedArea = ref(null);

const pointers = reactive([
  {
    id: 1,
    name: 'Kwek kwek',
    x: 100,
    y: 100,
    color: 'orange',
  },
  {
    id: 3,
    name: 'Hotdog',
    x: 300,
    y: 240,
    color: 'red',
  },
  {
    id: 4,
    name: 'Avocado',
    x: 140,
    y: 380,
    color: 'green',
  },
  {
    id: 5,
    name: 'Blueberry',
    x: null,
    y: null,
    color: 'blue',
  },
  {
    id: 6,
    name: 'Kamote',
    x: null,
    y: null,
    color: 'purple',
  },
])

const mapPointer = computed(() => {
  return pointers.filter(pointer => pointer.x && pointer.y)
})

onMounted(() => {
  const draggable = new Droppable(document.querySelectorAll('.drag-container'), {
    draggable: '.drag-item',
    dropzone: '.dropzone-container',
  });

  draggable.on('drag:start', (event) => {
    const { sensorEvent, source } = event
    const elementRect = source.getBoundingClientRect();

    const startX = sensorEvent.clientX;
    const startY = sensorEvent.clientY;

    const relativeX = startX - elementRect.left;
    const relativeY = startY - elementRect.top;

    elementPosX.value = relativeX;
    elementPosY.value = relativeY;
  });

  draggable.on('drag:move', (event) => {
    const { sensorEvent, sourceContainer } = event;
    const containerRect = sourceContainer.getBoundingClientRect();

    const startX = sensorEvent.clientX;
    const startY = sensorEvent.clientY;

    const relativeX = startX - containerRect.left;
    const relativeY = startY - containerRect.top;

    posX.value = relativeX - elementPosX.value;
    posY.value = relativeY - elementPosY.value;
  });

  draggable.on('droppable:dropped', (event) => {
    event.cancel();
    const { dropzone } = event;
    droppedArea.value = dropzone.dataset.dropzone;
  });

  draggable.on('drag:stop', (event) => {
    const { source } = event
    const pointerId = source.dataset.pointerId;
    const pointerIndex = pointers.findIndex(pointer => pointer.id == pointerId);

    if (droppedArea.value === 'map-dropzone') {
      pointers[pointerIndex].x = posX.value;
      pointers[pointerIndex].y = posY.value;
    } else if (droppedArea.value === 'legends-dropzone') {
      pointers[pointerIndex].x = null;
      pointers[pointerIndex].y = null;
    }
    droppedArea.value = null;
  });
});
</script>

<template>
  <main>
    <div class="drag-container grid grid-cols-12 gap-4">
      <div class="relative col-span-7 border border-slate-400 ring-2 ring-offset-4 ring-slate-700 rounded-md">
        <svg
          class="map"
          width="420"
          height="420"
          viewBox="0 0 420 420"
        >
          <image
            href="./assets/floorplan-asset.png"
            alt="Floor plan"
            width="100%"
            height="100%"
          />
          <g>
            <a v-for="pointer in mapPointer" :key="pointer.id" href="#" class="drag-item">
              <circle
                r="9"
                :cx="pointer.x"
                :cy="pointer.y"
                :fill="pointer.color"
                :style="{ transform: `translate(${18 / 2}px, ${18 / 2}px)` }"
              ></circle>
            </a>
          </g>
        </svg>
        <div class="dropzone-container absolute top-0 left-0 w-full h-full" data-dropzone="map-dropzone">
          <div
            v-for="pointer in mapPointer"
            :key="pointer.id"
            class="drag-item absolute w-[18px] h-[18px] rounded-full"
            :style="[{ backgroundColor: pointer.color, top: `${pointer.y}px`, left: `${pointer.x}px` }]"
            :data-pointer-id="pointer.id"
          ></div>
        </div>
      </div>
      <div class="dropzone-container relative col-span-5 p-5 bg-neutral-100 rounded-lg" data-dropzone="legends-dropzone">
        <ul class="space-y-4">
          <li v-for="pointer in pointers" :key="pointer.id">
            <div class="flex items-center gap-3">
              <div
                class="drag-item w-[18px] h-[18px] rounded-full"
                :class="{'opacity-30': pointer.x && pointer.y}"
                :style="{ backgroundColor: pointer.color }"
                :data-pointer-id="pointer.id"
              ></div>
              {{ pointer.name }}
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="absolute top-0 left-0">
      <pre>
{{ pointers }}
      </pre>
    </div>
  </main>
</template>
