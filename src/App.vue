<script setup>
import { ref, onMounted, reactive, computed } from 'vue';
import { Droppable } from '@shopify/draggable';
import hljs from "highlight.js/lib/core";
import javascript from "highlight.js/lib/languages/javascript";

import "highlight.js/styles/github-dark.css";
hljs.registerLanguage("javascript", javascript);

const IMAGE_BASE_WIDTH = 420;

const rect = ref(null);
const posX = ref(0);
const posY = ref(0);
const elementPosX = ref(0);
const elementPosY = ref(0);
const droppedArea = ref(null);
const imageScaleRatio = ref(1);

const pointers = reactive([
  {
    id: 1,
    name: 'Hotdog',
    x: 300,
    y: 240,
    color: 'red',
  },
  {
    id: 3,
    name: 'Kwek kwek',
    x: 100,
    y: 100,
    color: 'orange',
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
  return pointers.filter(pointer => pointer.x && pointer.y);
})

onMounted(() => {
  window.addEventListener('resize', onResize);

  const draggable = new Droppable(document.querySelectorAll('.drag-container'), {
    draggable: '.drag-item',
    dropzone: '.dropzone-container',
  });

  // Gets which area of the legend is clicked to prevent jumping when dragged from legends container to map container
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

  // Dragging from the legends container to the map container
  draggable.on('drag:over:container', (event) => {
    const { source, overContainer } = event;

    if (!(source instanceof SVGElement) && overContainer.dataset.container === 'map') {
      draggable.on('drag:move', (event) => {
        const { sensorEvent, source } = event;
        const containerRect = overContainer.querySelector("image").getBoundingClientRect();

        source.style.zIndex = 1;

        const startX = sensorEvent.clientX;
        const startY = sensorEvent.clientY;

        const relativeX = startX - containerRect.left;
        const relativeY = startY - containerRect.top;

        posX.value = (relativeX - elementPosX.value) * imageScaleRatio.value;
        posY.value = (relativeY - elementPosY.value) * imageScaleRatio.value;
      });
    }
  });

  // Dragging legends within the map
  draggable.on('drag:move', (event) => {
    const { sensorEvent, source } = event;

    if (source instanceof SVGElement) {
      const svgCTM = source.getScreenCTM();
  
      const startX = sensorEvent.clientX;
      const startY = sensorEvent.clientY;
  
      const svgCoord = {
        x: (startX - svgCTM.e) / svgCTM.a,
        y: (startY - svgCTM.f) / svgCTM.d
      };
  
      posX.value = svgCoord.x;
      posY.value = svgCoord.y;
  
      source.setAttributeNS(null, "cx", svgCoord.x);
      source.setAttributeNS(null, "cy", svgCoord.y);
    }
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

    // If dropped outside, put back to legends container
    if (posX.value < 0 || posX.value > IMAGE_BASE_WIDTH || posY.value < 0 || posY.value > IMAGE_BASE_WIDTH) {
      pointers[pointerIndex].x = null;
      pointers[pointerIndex].y = null;
      droppedArea.value = null;
      return;
    }

    if (droppedArea.value === 'map-dropzone') {
      pointers[pointerIndex].x = posX.value;
      pointers[pointerIndex].y = posY.value;
    } else if (droppedArea.value === 'legends-dropzone') {
      pointers[pointerIndex].x = null;
      pointers[pointerIndex].y = null;
    }

    droppedArea.value = null;
  });

  onResize();
  hljs.highlightAll();
});

function onResize () {
  imageScaleRatio.value = IMAGE_BASE_WIDTH / rect.value?.getBoundingClientRect().width;
}
</script>

<template>
  <main class="p-20">
    <div class="grid grid-cols-12 gap-4">
      <div class="drag-container relative order-2 col-span-12 border border-slate-400 ring-2 ring-offset-4 ring-slate-700 rounded-md md:order-1 md:col-span-7" data-container="map">
        <svg
          class="dropzone-container max-w-full"
          :width="IMAGE_BASE_WIDTH"
          :height="IMAGE_BASE_WIDTH"
          :viewBox="`0 0 ${IMAGE_BASE_WIDTH} ${IMAGE_BASE_WIDTH}`"
          data-dropzone="map-dropzone"
        >
          <image
            ref="rect"
            href="./assets/floorplan-asset.png"
            class="pointer-events-none bg-slate-700"
            alt="Floor plan"
            :width="`${IMAGE_BASE_WIDTH}px`"
            :height="`${IMAGE_BASE_WIDTH}px`"
          />
          <g class="svg-draggable-area">
            <circle
              v-for="pointer in mapPointer" :key="pointer.id"
              class="drag-item"
              r="9"
              :cx="pointer.x"
              :cy="pointer.y"
              :fill="pointer.color"
              :style="{ transform: `translate(${18 / 2}px, ${18 / 2}px)` }"
              :data-pointer-id="pointer.id"
            ></circle>
          </g>
        </svg>
      </div>
      <div class="drag-container dropzone-container relative order-1 col-span-12 p-5 bg-neutral-100 rounded-lg md:order-2 md:col-span-5" data-container="legends" data-dropzone="legends-dropzone">
        <ul class="space-y-4">
          <li v-for="pointer in pointers" :key="pointer.id">
            <div class="flex items-center gap-3">
              <div
                class="drag-item rounded-full"
                :class="{'opacity-10': pointer.x && pointer.y}"
                :style="{ backgroundColor: pointer.color, width: `${18 / imageScaleRatio}px`, height: `${18 / imageScaleRatio}px` }"
                :data-pointer-id="pointer.id"
              ></div>
              {{ pointer.name }}
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="grid">
      <pre>
        <code>
{{ { posX, posY } }}
{{ pointers }}
        </code>
      </pre>
    </div>
  </main>
</template>

<style>
  .svg-draggable-area .draggable-mirror {
    display: none;
  }
  [data-container="legends"] .draggable-mirror {
    z-index: 1;
  }
</style>
