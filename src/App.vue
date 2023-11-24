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
          draggable="true"
      ></div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  name: "App",

  setup() {
    let items_ar = [];
    let count_aqua = Math.floor(Math.random() * 9) + 2;
    for (let i = 0; i < count_aqua; i++) {
      items_ar.push({ id: i, categoryId: 0 });
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

    function onDragStart(e, item) {
      e.dataTransfer.dropEffect = "move";
      e.dataTransfer.effectAllowed = "move";
      e.dataTransfer.setData("itemId", item.id.toString());
    }

    function onDrop(e, categoryId) {
      const itemId = parseInt(e.dataTransfer.getData("itemId"));
      items.value = items.value.map((x) => {
        if (x.id == itemId) x.categoryId = categoryId;
        return x;
      });


      const allDropsOnGround = items.value.every(
          (item) => item.categoryId === 1
      );

      const allDropsOnSky = items.value.every((item) => item.categoryId === 1);

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
}

.sky {
  position: absolute;
  top: 0; opacity: 0.85;
  background-color: rgba(157, 156, 156, 0.75);
}


.ground {
  position: absolute;
  bottom: 0; left: 0;
  width: 100%;
  background-color: rgba(63, 18, 18, 0.75);
}

.droppable {
  left: 0;
  width: 100%;
  height: 15%;
  display: flex;
  justify-content: space-around;
  align-items: center;
  transition: all 1s ease;
}

.draggable {
  padding: 2%;
  margin: 1%;
  display: inline-block;
  border-radius: 50%;
  background: #0c36e3;
  transition: all 1s ease;
}

.draggable h5 {
  margin: 0;
}


.ground-green {
  background-color: green;
}
.sky-white {
  background-color: white;
}

</style>