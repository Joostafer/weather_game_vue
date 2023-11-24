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
          :key="item.id"
          @dragstart="onDragStart($event, item)"
          class="draggable"
          :style="item.style"
          draggable="true"
          @touchstart="onTouchStart($event, item)"
          @touchmove="onTouchMove($event, item)"
          @touchend="onTouchEnd($event, item)"
      ></div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";

function generateRandomPosition(maxX, maxY) {
  const randomX = Math.floor(Math.random() * maxX);
  const randomY = Math.floor(Math.random() * maxY);

  return { x: randomX, y: randomY };
}

export default {
  name: "App",

  setup() {
    const items = ref([]);
    const isGroundGreen = ref(false);
    const isSkyWhite = ref(false);
    const categories = ref([
      { id: 0, title: "sky" },
      { id: 1, title: "ground" },
    ]);

    function initializeItems() {
      const count_aqua = Math.floor(Math.random() * 9) + 2;
      const maxX = window.innerWidth-20;
      const maxY = 80;

      for (let i = 0; i < count_aqua; i++) {
        const randomPosition = generateRandomPosition(maxX, maxY);
        items.value.push({
          id: i,
          categoryId: 0,
          style: {
            left: `${randomPosition.x}px`,
            top: `${randomPosition.y}px`,
          },
        });
      }
    }

    onMounted(() => {
      initializeItems();
    });

    function onTouchMove(e, item) {
      e.preventDefault();
      const offsetX = e.touches[0].clientX;
      const offsetY = e.touches[0].clientY;

      item.style = {
        left: offsetX - item.touchStartX + "px",
        top: offsetY - item.touchStartY + "px",
      };
    }

    function onTouchStart(e, item) {
      e.preventDefault();

      const touch = e.touches[0];
      const rect = e.target.getBoundingClientRect();
      const offsetX = touch.clientX - rect.left;
      const offsetY = touch.clientY - rect.top;

      item.touchStartX = offsetX;
      item.touchStartY = offsetY;

      const isDraggable = e.dataTransfer && "effectAllowed" in e.dataTransfer;

      if (isDraggable) {
        e.dataTransfer.effectAllowed = "move";
        e.dataTransfer.setData("itemId", item.id.toString());
        e.dataTransfer.setDragImage(new Image(), 0, 0);
      }
    }

    function onTouchEnd(e, item) {
      e.preventDefault();

      const offsetX = e.changedTouches[0].clientX;
      const offsetY = e.changedTouches[0].clientY;

      item.style = {
        left: offsetX - item.touchStartX + "px",
        top: offsetY - item.touchStartY + "px",
      };

      delete item.touchStartX;
      delete item.touchStartY;

      let vh = window.innerHeight;
      let ground_start = (vh * 85) / 100;
      if(ground_start < offsetY){
        item.categoryId = 1;
      }
      const allDropsOnGround = items.value.every(
          (item) => item.categoryId === 1
      );
      const allDropsOnSky = items.value.every((item) => item.categoryId === 1);


      isGroundGreen.value = allDropsOnGround;
      isSkyWhite.value = allDropsOnSky;
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
            left: e.clientX - parentRect.left + "px",
            top: e.clientY - parentRect.top + "px",
          };
        }
        return x;
      });

      items.value = updatedItems;

      const allDropsOnGround = updatedItems.every(
          (item) => item.categoryId === 1
      );
      const allDropsOnSky = updatedItems.every((item) => item.categoryId === 1);

      isGroundGreen.value = allDropsOnGround;
      isSkyWhite.value = allDropsOnSky;
    }

    function getCategoryClasses(category) {
      return {
        "ground-green": category.title === "ground" && isGroundGreen.value,
        "sky-white": category.title === "sky" && isSkyWhite.value,
        [category.title]: true,
      };
    }

    return {
      items,
      categories,
      isGroundGreen,
      isSkyWhite,
      onTouchMove,
      onTouchStart,
      onTouchEnd,
      onDragStart,
      onDrop,
      getCategoryClasses,
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
  opacity: 0.85;
  background-color: rgba(157, 156, 156, 0.75);
}
.ground {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background-color: rgba(63, 18, 18, 0.75);
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
