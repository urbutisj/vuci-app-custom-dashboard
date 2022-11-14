<template>
  <div>
    <div class="container">
      <div v-for="(card, index) in cardsData" :key="index">
        <CardComponent :card="card" />
      </div>
      <button-group @drawerOpen="toggleDrawer" :drawerVisible="drawerVisible"></button-group>
    </div>
    <a-drawer
      title="Filter"
      placement="right"
      :closable="false"
      :visible="drawerVisible"
    >
      <SideFilter
        :cardsData="cardsData"
      />
    </a-drawer>
  </div>
</template>
<script>
import CardComponent from '../components/VuciDashboard/CardComponent.vue'
import SideFilter from '../components/VuciDashboard/SideFilter.vue'
import ButtonGroup from '../components/VuciDashboard/ButtonGroup.vue'
export default {
components: { CardComponent, SideFilter, ButtonGroup },
data () {
  return {
    uptime: 0,
    ramUsage: 0,
    flashUsage: 0,
    lastCPUTime: null,
    cpuUsage: 0,
    cardsData: [],
    filtersArray: [],
    drawerVisible: false
  }
},
methods: {
  async creatingFinalObject () {
    // Final Object
    this.cardsData.push(await this.getSystemInfo())
    await this.getNetworkInterfaces()
    this.cardsData.push(await this.getSystemLog())
    this.cardsData.push(await this.getNetworkLog())
    await this.getWireless()
  },

  async getSystemInfo () {
    const dataObj = { title: 'System', extraHeader: {}, data: [], isVisible: true }
    const system = await this.$system.getInfo()
    dataObj.extraHeader = ({ title: 'CPU load', value: this.cpuUsage })
    dataObj.data.push({ title: 'Router uptime', value: '%t'.format(system.uptime) })
    dataObj.data.push({ title: 'Local Device Time', value: this.formatLocalTimeUCI(system.localtime) })
    dataObj.data.push({
      title: 'Memory Usage',
      data: [
        { title: 'RAM', value: this.ramUsage },
        { title: 'FLASH', value: this.flashUsage }]
    })
    dataObj.data.push({ title: 'Firmware Version', value: system.release.revision })
    return dataObj
  },

  getPercentage () {
    const dataObj = { title: 'Memory Usage', data: [] }
    dataObj.data.push({ title: 'RAM', value: this.ramUsage })
    dataObj.data.push({ title: 'FLASH', value: this.flashUsage })
    return dataObj
  },

  async getNetworkInterfaces () {
    // Network Details
    let interfaces = []
    await this.$network.load()
    interfaces = this.$network.getInterfaces()
    for (const netinterface of interfaces) {
      const dataObj = { title: netinterface.name, data: [], isVisible: true }
      dataObj.data.push({ title: 'Type', value: netinterface.status.device })
      const address = !netinterface.status['ipv4-address'] ? '-' : netinterface.status['ipv4-address'][0].address
      dataObj.data.push({ title: 'IP address', value: address })
      this.cardsData.push(dataObj)
    }
  },
  async getNetworkLog () {
    // Network Log
    const dataObj = { title: 'Network Logs', data: [], isVisible: true }
    await this.$log.get({ table: 'NETWORK', limit: 5 }).then(r => {
      for (const record of r.log) {
        dataObj.data.push({ title: this.formatLocalTimeUCI(record.TIME), value: record.TEXT })
      }
    })
    return dataObj
  },
  async getSystemLog () {
    const dataObj = { title: 'System logs', data: [], isVisible: true }
    await this.$log.get({ limit: 5 }).then(r => {
      for (const record of r.log) {
        dataObj.data.push({ title: this.formatLocalTimeUCI(record.TIME), value: record.TEXT })
      }
    })
    return dataObj
  },
  async update () {
    const { root } = await this.$system.getDiskInfo()
    this.flashUsage = Math.floor((root.used / root.total) * 100)
    // if card is not visible, no update is required. Fetch optimization
    const newCard = { info: null, index: null }
    this.cardsData.find(async (card, i) => {
      if (card.title === 'System') {
        newCard.index = i
        newCard.info = await this.$system
          .getInfo()
          .then(({ release, localtime, uptime, memory }) => {
            this.ramUsage = Math.floor(
              ((memory.total - memory.free) / memory.total) * 100
            )
            return {
              ...card,
              extraHeader: {
                title: 'CPU load',
                value: this.cpuUsage
              },
              data: [
                { title: 'Router uptime', value: '%t'.format(uptime) },
                { title: 'Local Device Time', value: this.formatLocalTimeUCI(localtime) },
                {
                  title: 'Memory Usage',
                  data: [
                    { title: 'RAM', value: this.ramUsage },
                    { title: 'FLASH', value: this.flashUsage }]
                },
                { title: 'Firmware Version', value: release.revision }
              ]
            }
          })
        if (newCard.info && newCard.index !== null) {
          this.cardsData.splice(newCard.index, 1, newCard.info)
        }
      }
    })
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
  // Network Info
  async getWireless () {
    const devices = await this.$wireless.getDevices().then((res) => {
      return res
    })
    const promises = []
    devices.forEach((dev) => {
      promises.push(this.$rpc.ubus('iwinfo', 'info', { device: dev }))
    })
    let wirelessDev = []
    wirelessDev = await Promise.all(promises).then((res) => {
      return res
    })
    wirelessDev.forEach((dev) => {
      const name =
        dev.frequency > 2500 ? `${dev.ssid} (5GHZ)` : `${dev.ssid} (2.4GHZ)`
      const dataObj = {
        title: name,
        extraHeaderWifi: { title: 'ON', value: 'wifi', on: true },
        data: [
          { title: 'SSID', value: dev.ssid },
          { title: 'Mode', value: dev.mode },
          { title: 'Channel', value: dev.channel },
          { title: 'Clients', value: '0 (hard coded)' }
        ]
      }
      this.cardsData.push(dataObj)
    })
  },
  toggleDrawer () {
    this.drawerVisible = !this.drawerVisible
  }
},
timers: {
  update: { time: 5000, autostart: true, repeat: true, immediate: true },
  updateCpuUsage: {
    time: 5000,
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
.ant-drawer-content-wrapper {
width: 300px!important;
}
</style>
