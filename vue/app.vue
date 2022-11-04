<template>
  <div>
    <div class="container">
      <div v-for="(card, index) in cardsData" :key="index">
        <CardComponent :card="card">
          <template v-if="card.usage" v-slot:cpu>
            <slot>{{card.usage}}</slot>
          </template>
        </CardComponent>
      </div>
    </div>
  </div>
</template>
<script>
import CardComponent from '../components/VuciDashboard/CardComponent.vue'
import UsageLine from '../components/VuciDashboard/UsageLine.vue'
export default {
components: { CardComponent, UsageLine },
data () {
  return {
    uptime: 0,
    ramUsage: 0,
    flashUsage: 0,
    lastCPUTime: null,
    cpuUsage: 0,
    cardsData: []
  }
},
methods: {
  async creatingFinalObject () {
    // Final Object
    this.cardsData.push(await this.getSystemInfo())
    await this.getNetworkInterfaces()
    this.cardsData.push(await this.getSystemLog())
    this.cardsData.push(await this.getNetworkLog())
  },

  async getSystemInfo () {
    const dataObj = { title: 'System', data: [], usage: [] }
    const system = await this.$system.getInfo()
    dataObj.data.push({ title: 'Router uptime', value: '%t'.format(system.uptime) })
    dataObj.data.push({ title: 'Local Device Time', value: this.formatLocalTimeUCI(system.localtime) })
    dataObj.data.push({ title: 'Firmware Version', value: system.release.revision })
    dataObj.usage.push({ title: 'CPU load', value: this.cpuUsage })
    dataObj.usage.push({ title: 'RAM', value: this.ramUsage })
    dataObj.usage.push({ title: 'FLASH', value: this.flashUsage })
    console.log(dataObj)
    return dataObj
  },

  async getNetworkInterfaces () {
    // Network Details
    let interfaces = []
    await this.$network.load()
    interfaces = this.$network.getInterfaces()
    for (const netinterface of interfaces) {
      const dataObj = { title: netinterface.name, data: [] }
      dataObj.data.push({ title: 'Type', value: netinterface.status.device })
      const address = !netinterface.status['ipv4-address'] ? '-' : netinterface.status['ipv4-address'][0].address
      dataObj.data.push({ title: 'IP address', value: address })
      this.cardsData.push(dataObj)
    }
  },
  async getNetworkLog () {
    // Network Log
    const dataObj = { title: 'Network Logs', data: [] }
    await this.$log.get({ table: 'NETWORK', limit: 5 }).then(r => {
      for (const record of r.log) {
        dataObj.data.push({ title: this.formatLocalTimeUCI(record.TIME), value: record.TEXT })
      }
    })
    return dataObj
  },
  async getSystemLog () {
    const dataObj = { title: 'System logs', data: [] }
    await this.$log.get({ limit: 5 }).then(r => {
      for (const record of r.log) {
        dataObj.data.push({ title: this.formatLocalTimeUCI(record.TIME), value: record.TEXT })
      }
    })
    return dataObj
  },
  async update () {
    this.system = await this.$system.getInfo()
    this.ramUsage = Math.floor(
      ((this.system.memory.total - this.system.memory.free) /
          this.system.memory.total) *
          100
    )
    const { root } = await this.$system.getDiskInfo()
    this.flashUsage = Math.floor((root.used / root.total) * 100)
  },
  updateCpuUsage () {
    this.$rpc.call('system', 'cpu_time').then((times) => {
      if (!this.lastCPUTime) {
        this.cpuUsage = 0
        this.lastCPUTime = times
        return
      }
      const idle1 = this.lastCPUTime[3]
      const idle2 = times[3]
      let total1 = 0
      let total2 = 0
      this.lastCPUTime.forEach((t) => {
        total1 += t
      })
      times.forEach((t) => {
        total2 += t
      })
      this.cpuUsage = Math.floor(
        ((total2 - total1 - (idle2 - idle1)) / (total2 - total1)) * 100
      )
      this.lastCPUTime = times
    })
  },
  formatLocalTimeUCI (time) {
    const d = new Date(time * 1000)
    const year = d.getFullYear().toString().padStart(2, 0)
    const month = (d.getMonth() + 1).toString().padStart(2, 0)
    const date = d.getDate().toString().padStart(2, 0)
    const hour = d.getHours()
    const min = d.getMinutes()
    const sec = d.getSeconds()
    return `${year}-${month}-${date} %02d:%02d:%02d`.format(hour, min, sec)
  }
},
timers: {
  update: { time: 1000, autostart: true, repeat: true, immediate: true },
  updateCpuUsage: {
    time: 1000,
    autostart: true,
    immediate: true,
    repeat: true
  }
},
async created () {
  await this.creatingFinalObject()
}
}
</script>

<style>
.vuci-main-content{
background-color: unset!important;
}
.container {
display: grid;
grid-template-columns: repeat(auto-fill, minmax(32%, 1fr));
grid-auto-rows: 1fr;
grid-gap: 1rem;
}
</style>
