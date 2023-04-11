<template>
  <div class="parameters-change-window">
    <button @click="$emit('close')" class="close-button"></button>
    <div class="parameters-change-window__title">{{ windowName }}</div>
    <div class="parameters-change-window__content">
      <div class="base-input-group" v-for="(input, i) in inputs">
        <label class="base-input-group__label" :for="windowName + `_` + i"
          >{{ input.name }} (between {{ input.min }} and {{ input.max }})</label
        >
        <input
          class="base-input-group__input"
          type="number"
          @input="onInput(i, $event.target.value)"
          :id="windowName + `_` + i"
          :value="defaults[i]"
          :step="input.step"
          :min="input.min"
          :max="input.max"
        />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    windowName: {
      type: String,
      required: true,
    },

    inputs: {
      type: Array,
      required: true,
      validator: (inputs) =>
        inputs.every(
          (input) =>
            typeof input?.min === "number" &&
            typeof input?.max === "number" &&
            typeof input?.name === "string"
        ),
    },
  },

  emits: ["close"],

  data() {
    return {
      defaults: {},
    };
  },

  mounted() {
    this.computeDefaults();
  },

  methods: {
    computeDefaults() {
      this.defaults = this.inputs.map((el) => el.getter());
    },

    onInput(index, value) {
      const input = this.inputs[index];

      if (
        !Number.isNaN(Number(value)) &&
        input.min <= Number(value) &&
        Number(value) <= input.max
      ) {
        input.setter(Number(value));
      } else {
        this.computeDefaults();
      }
    },
  },
};
</script>

<style lang="scss">
.parameters-change-window {
  background: white;
  padding: 15px;
  border-radius: 5px;
  position: relative;

  &__title {
    margin: 5px 0 10px;
    padding-right: 15px;
    font-weight: 600;
  }

  &__content {
    display: grid;
    grid-row-gap: 15px;
  }
}

.close-button {
  position: absolute;
  right: 8px;
  top: 6px;
  background: transparent;
  border: none;
  height: 20px;
  width: 20px;
  border-radius: 50%;
  cursor: pointer;

  &::after,
  &::before {
    content: "";
    display: block;
    width: 100%;
    height: 3px;
    background: black;
    position: absolute;
    left: 50%;
    top: 50%;
    border-radius: 2px;
  }

  &::after {
    transform: translate(-50%, -50%) rotate(45deg);
  }

  &::before {
    transform: translate(-50%, -50%) rotate(-45deg);
  }
}

.base-input-group {
  &__label {
    display: block;
    margin-bottom: 5px;
  }

  &__input {
    width: 100%;
  }
}
</style>
