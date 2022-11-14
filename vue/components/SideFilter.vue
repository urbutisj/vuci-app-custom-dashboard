<template>
    <a-form>
      <a-form-item>
        <a-checkbox-group :options="options" class="checkboxGroup">
        </a-checkbox-group>
      </a-form-item>
    </a-form>
  </template>
  <script>
  export default {
    props: ['cardsData'],
    computed: {
      options () {
        return this.cardsData.map((card) => ({
          label: card.title.toUpperCase(),
          value: card.title
        }))
      }
    }
  }
  </script>
  
  <style scoped>
    .checkboxGroup {
      display: flex;
      flex-direction: column;
    }
  </style>
  