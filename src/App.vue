<template>
  <div class="app">
    <div class="app__task-links">
      <div
        v-for="i in [1, 2]"
        :key="i"
        class="app__task-link"
        :class="{ 'app__task-link_selected': task === i }"
        @click="task = i"
      >
        Task {{ i }}
      </div>
    </div>
    <First v-if="task === 1" />
    <Second v-if="task === 2" />
  </div>
</template>

<script>
import First from "@/components/First/index.vue";
import Second from "@/components/Second/index.vue";

export default {
  components: { First, Second },

  data() {
    return {
      task: 2,
    };
  },
};
</script>

<style lang="scss">
.app {
  position: relative;
  &__task-links {
    position: absolute;
    top: 0;
    left: 0;
    background: white;
    z-index: 5;
    padding: 10px 15px;
    display: flex;
  }

  &__task-link {
    background: lightgrey;
    padding: 3px 7px;
    border-radius: 5px;
    cursor: pointer;

    &:not(:first-child) {
      margin-left: 15px;
    }

    &_selected {
      background: lightblue;
      border: solid 1px grey;
    }
  }
}
</style>
