<template>
  <a-card class="card">
    <div class="card-details">
      <div class="card-header">
        <h2>{{singleCard.name}}</h2>
      </div>
      <div class="card-body" v-if="singleCard.id === 3">
        <div class="">
          <label>{{singleCard.routerTime.label}}</label>
          <span>{{ "%t".format(singleCard.routerTime.value) }}</span>
        </div>
        <div class="">
          <label>Local Time:</label>
          <span>{{formatLocalTime}}</span>
        </div>
        <div>
          <label>CPU:</label>
          <span>{{singleCard.cpuUsage}}</span>
        </div>
      </div>
    </div>
  </a-card>
</template>

<script>

export default {
  props: ['singleCard'],
  computed: {
    formatLocalTime () {
      const d = new Date(this.singleCard.routerLocalTime.value * 1000)
      const year = d.getFullYear().toString().padStart(2, 0)
      const month = (d.getMonth() + 1).toString().padStart(2, 0)
      const date = d.getDate().toString().padStart(2, 0)
      const hour = d.getHours()
      const min = d.getMinutes()
      const sec = d.getSeconds()
      return `${year}-${month}-${date} %02d:%02d:%02d`.format(hour, min, sec)
    }
  },
  created () {
    console.log(this.singleCard)
  }
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
