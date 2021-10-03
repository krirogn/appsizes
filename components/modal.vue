<template>
  <div class="modal" @mousedown="close">
    <div class="inner" :style="{ height: height, width: width, maxWidth: maxWidth, backgroundColor: bg }">
      <h1 class="head" :style="{ color: headColor }">{{ head }}</h1><br>
      <slot></slot>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: "modal",
  props: {
    width: {
      type: String,
      default: "calc(100% - 160px)",
    },
    maxWidth: {
      type: String,
      default: "100%",
    },
    height: {
      type: String,
      default: "calc(100% - 160px)",
    },
    head: {
      type: String,
    },
    headColor: {
      type: String,
      default: "white",
    },
    bg: {
      type: String,
      default: "white",
    }
  },
  methods: {
    close(event: any) {
      if (event.currentTarget === event.target) {
        this.$emit('close');
      }
    }
  }
})
</script>

<style lang="scss">
.modal {
  width: 100vw;
  height: 100vh;

  position: fixed;
  left: 0;
  top: 0;
  z-index: 9999;

  background-color: rgba(0, 0, 0, 0.5);
  cursor: pointer;

  .inner {
    border-radius: 20px;
    padding: 40px;
    
    user-select: none;
    cursor: default;

    position: relative;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    .head {
      width: 100%;

      text-align: center;
    }
  }
}
</style>