<template>
    <div
      class="box"
      :class="{ buttonContainerAnimationOpen: buttonPosition, buttonContainerAnimationClose: !buttonPosition }"
    >
      <div type="primary" @click="toggleDrawer">
        <a-icon class="icon" type="setting" />
      </div>
    </div>
  </template>

<script>
export default {
  props: {
    drawerVisible: {
      type: Boolean,
      required: true
    }
  },
  computed: {
    buttonPosition () {
      return this.drawerVisible
    }
  },
  methods: {
    toggleDrawer () {
      this.$emit('drawerOpen')
    }
  }
}
</script>
  <style scoped>
  .box {
    position: fixed;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 5px;
    height: 60px;
    width: 40px;
    background-color: #005ec1;
    cursor: pointer;
    top: 80px;
    border-top-left-radius: 0.8em;
    border-bottom-left-radius: 0.8em;
  }
  .icon {
    font-size: 1.4em;
    color: white;
  }
  .buttonContainerAnimationOpen {
    animation-name: openDrawer;
    animation-duration: 0.3s;
    animation-fill-mode: forwards;
    transition-timing-function: cubic-bezier(0.7, 0.3, 0.1, 1), ease, ease;
  }
  .buttonContainerAnimationClose {
    animation-name: closeDrawer;
    animation-duration: 0.3s;
    animation-fill-mode: forwards;
    transition-timing-function: cubic-bezier(0.7, 0.3, 0.1, 1), ease, ease;
  }
  @keyframes openDrawer {
    0% {
      right: 31.777px;
    }
    100% {
      right: 300px;
      z-index: 999999;
    }
  }
  @keyframes closeDrawer {
    0% {
      right: 300px;
      z-index: 999999;
    }
    100% {
      right: 31.777px;
    }
  }
  </style>
