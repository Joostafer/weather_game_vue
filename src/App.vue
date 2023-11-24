<template>
  <div id="screen">
    <div
        v-for="category in categories"
        :key="category.id"
        @drop="onDrop($event, category.id)"
        class="droppable"
        @dragover.prevent
        @dragenter.prevent
        :class="getCategoryClasses(category)"
    >
      <div
          v-for="item in items.filter(x => x.categoryId === category.id)"
          @dragstart="onDragStart($event, item)"
          class="draggable"
          :style="item.style"
          draggable="true"
      ></div>
    </div>
  </div>
</template>

<script>

import { ref } from "vue";
function generateRandomPosition() {
  const maxX = window.innerWidth-10;
  const maxY = 40;

  const randomX = Math.floor(Math.random() * maxX);
  const randomY = Math.floor(Math.random() * maxY);

  return { x: randomX, y: randomY };
}
export default {
  name: "App",

  setup() {
    let items_ar = [];
    let count_aqua = Math.floor(Math.random() * 9) + 2;

    for (let i = 0; i < count_aqua; i++) {
      const randomPosition = generateRandomPosition();
      items_ar.push({ id: i, categoryId: 0, style: {
          left: randomPosition.x + 'px',
          top: randomPosition.y + 'px',
        } });
    }

    let items = ref(items_ar);
    const isGroundGreen = ref(false);
    const isSkyWhite = ref(false);
    const categories = ref([
      {
        id: 0,
        title: "sky",
      },
      {
        id: 1,
        title: "ground",
      },
    ]);

    function onTouchStart(e, item) {

    }

    function onTouchMove(e, item) {
      item.style.left = e.touches[0].clientX + 'px';
      item.style.top = e.touches[0].clientY + 'px';
    }

    function onTouchEnd(e, item) {

    }

    function onDragStart(e, item) {
      e.dataTransfer.dropEffect = "move";
      e.dataTransfer.effectAllowed = "move";
      e.dataTransfer.setData("itemId", item.id.toString());
    }

    function onDrop(e, categoryId) {
      const itemId = parseInt(e.dataTransfer.getData("itemId"));

      const parentRect = e.target.getBoundingClientRect();
      const offsetX = e.clientX - parentRect.left;
      const offsetY = e.clientY - parentRect.top;

      const updatedItems = items.value.map((x) => {
        if (x.id === itemId) {
          x.categoryId = categoryId;

          x.style = {
            left: e.clientX - parentRect.left + 'px',
            top: e.clientY - parentRect.top + 'px',
          };
        }
        return x;
      });

      items.value = updatedItems;

      const allDropsOnGround = updatedItems.every((item) => item.categoryId === 1);
      const allDropsOnSky = updatedItems.every((item) => item.categoryId === 0);

      isGroundGreen.value = allDropsOnGround;
      isSkyWhite.value = allDropsOnSky;
    }

    function getCategoryClasses(category) {
      return {
        'ground-green': category.title === 'ground' && isGroundGreen.value,
        'sky-white': category.title === 'sky' && isSkyWhite.value,
        [category.title]: true
      };
    }

    return {
      items,
      categories,
      isGroundGreen,
      isSkyWhite,
      onTouchStart,
      onTouchMove,
      onTouchEnd,
      onDragStart,
      onDrop,
      getCategoryClasses
    };
  },
};
</script>

<style>
body {
  width: 100%;
  height: 100vh;
  background-color: #62afff;
  padding: 0;
  overflow: hidden;
}

.droppable {
  left: 0;
  width: 100%;
  height: 15%;
  transition: all 1s ease;
}
.sky {
  position: absolute;
  top: 0;
  opacity: .85;
  background-color: #9d9c9cbf;
}
.ground {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background-color: #3f1212bf;
}
.draggable {
  position: absolute;
  padding: 30px;
  display: inline-block;
  border-radius: 50%;
  background: #0c36e3;

  touch-action: none;
  user-drag: none;
}

.ground-green {
  background-color: green;
}
.sky-white {
  background-color: white;
}
</style>
