<template>
  <a-card class="card">
    <div class="card-details">
      <div class="card-header">
        <h2>{{card.title}}</h2>
      </div>
      <div class="card-body">
        <div v-for="(cardData, index) in card.data" :key="index" >
          <label>{{cardData.title}}</label>
          <div>{{cardData.value}}</div>
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
}
.card:hover{
  border-color: rgba(0, 0, 0, 0.09);
  box-shadow: 0 2px 8px rgb(0 0 0 / 9%);
}
.card h2,
.card h3 {
  font-family: 'Oswald', sans-serif;
  display: flex;
  align-items: center;
}
.card h2 {
  text-transform: uppercase;
}
</style>
