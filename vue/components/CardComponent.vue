<template>
  <a-card class="card">
    <div class="card-details">
      <div class="card-header">
        <h2>{{card.title}}</h2>
        <slot name="cpu"></slot>
      </div>
      <div class="card-body">
        <div v-for="(cardData, index) in card.data" :key="index" >
          <h4>{{cardData.title}}</h4>
          <label>{{cardData.value}}</label>
        </div>
      </div>
    </div>
  </a-card>
</template>

<script>
export default {
  props: ['card']
}
</script>
<style>
.card {
  border-radius: 15px;
  height: 100%;
}
.card:hover{
  border-color: rgba(0, 0, 0, 0.09);
  box-shadow: 0 2px 8px rgb(0 0 0 / 9%);
}
.card h2 {
  font-family: 'Oswald', sans-serif;
  display: flex;
  align-items: center;
  margin-bottom: 0;
}
.card h4 {
  font-weight: bold;
  color: #222;
}
.card h2 {
  text-transform: uppercase;
}
.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.card-body>div {
  padding: .5rem 0;
}
.card-body>div:not(:last-child){
  border-bottom: 1px solid #bebebe;
}
</style>
