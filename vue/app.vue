<template>
  <div>
    <div class="container">
      <template v-for="card in cardsData">
        <CardComponent v-for="data in card" :card="data"/>
      </template>
    </div>
  </div>
</template>
<script>
import CardComponent from '../components/VuciDashboard/CardComponent.vue'
export default {
components: { CardComponent },
data () {
  return {
    system: [],
    interfaces: [],
    netlog: [],
    syslog: [],
    eventsCount: 4,
    uptime: 0,
    ramUsage: 0,
    flashUsage: 0,
    lastCPUTime: null,
    cpuUsage: 0,
    cardsData: {}
  }
},
methods: {
  async creatingFinalObject () {
    // Final Object
    this.cardsData = {
      system: [{
        title: 'System',
        data:
        [{ title: 'Router Uptime', value: '%t'.format(this.system.uptime) },
          { title: 'Local Time', value: this.formatLocalTimeUCI(this.system.localtime) },
          { title: 'Firmware Version', value: this.system.release.revision },
          { title: 'CPU load', value: this.cpuUsage },
          { title: 'RAM', value: this.ramUsage },
          { title: 'FLASH', value: this.flashUsage }
        ]
      }],
      network: [...this.interfaces],
      netLog: [{
        title: 'Network Logs',
        data: this.netlog
      }],
      sysLog: [{
        title: 'System Logs',
        data: this.syslog
      }]
    }
    console.log(this.cardsData)
  },

  async getSystemInfo () {
    // System Details
    this.system = await this.$system.getInfo()
  },

  async getNetworkInterfaces () {
    // Network Details
    await this.$network.load().then(() => this.$network.getInterfaces()).then((res) => {
      res.map(i => {
        this.interfaces.unshift({
          title: i.name,
          data: [
            {
              title: 'TYPE',
              value: i.status.device ? i.status.device : '---'
            },
            {
              title: 'IP ADDRESS',
              value: Object.prototype.hasOwnProperty.call(i.status, ['ipv4-address'][0]) ? i.status['ipv4-address'][0].address : '---'
            }]
        })
      })
    })
    console.log(this.interfaces)
  },
  async getNetworkLog () {
    // Network Log
    await this.$rpc.ubus('log', 'read_db', { table: 'NETWORK' }).then(response => {
      const sliced = response.log.slice(-this.eventsCount).reverse()
      const newArr = sliced.map((val) => {
        return { title: this.formatLocalTimeUCI(val.time), value: val.text }
      })
      return this.netlog = newArr
    })
  },
  async getSystemLog () {
    // System Log
    await this.$rpc.call('system', 'syslog').then(response => {
      const sliced = response.log.slice(-this.eventsCount).reverse()
      const newArr = sliced.map((val) => {
        return { title: this.formatDate(val.datetime), value: val.msg }
      })
      this.syslog = newArr
    })
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
  },
  formatDate (time) {
    const d = new Date(time)
    const year = d.getFullYear().toString().padStart(2, 0)
    const month = (d.getMonth() + 1).toString().padStart(2, 0)
    const date = d.getDate().toString().padStart(2, 0)
    const hour = d.getHours()
    const min = d.getMinutes()
    const sec = d.getSeconds()
    return `${year}-${month}-${date} %02d:%02d:%02d`.format(hour, min, sec)
  },
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
  await this.getSystemInfo()
  await this.getNetworkInterfaces()
  await this.getSystemLog()
  await this.getNetworkLog()
  this.creatingFinalObject()
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
