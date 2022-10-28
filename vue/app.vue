<template>
    <div>
      <transition-group class="container">
        <template  v-for="(card) of cardsData" >
          <CardComponent :singleCard="singleCard" :key="singleCard.id"  v-for="singleCard in card"/>
        </template>
      </transition-group>
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
      cardsData: {},
      id: 0,
      type: {
        time: 'time',
        date: 'date',
        number: 'number',
        percentage: 'percentage'
      }
    }
  },
  methods: {
    async creatingFinalObject () {
      // Final Object
      this.cardsData = {
        system: [{
          id: this.id++,
          name: 'System',
          routerTime: {
            label: 'Router Uptime: ',
            value: this.system.uptime,
            type: this.type.date
          },
          routerLocalTime: {
            label: 'Local Time: ',
            value: this.system.localtime,
            type: this.type.date
          },
          firmware: {
            label: 'Firmware Version: ',
            value: this.system.release.revision
          },
          cpuUsage: {
            label: 'CPU load: ',
            value: this.cpuUsage,
            type: this.type.percentage
          },
          ramUsage: {
            label: 'RAM: ',
            value: this.ramUsage,
            type: this.type.percentage
          },
          flashUsage: {
            label: 'FLASH: ',
            value: this.flashUsage,
            type: this.type.percentage
          }
        }],
        network: this.interfaces,
        netLog: [{
          id: this.id++,
          name: 'Network Logs',
          logs: this.latestLogs
        }],
        sysLog: [{
          id: this.id++,
          name: 'System Logs',
          logs: this.latestSysLogs
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
        res.map(intrfc => {
          this.interfaces.unshift({
            id: this.id++,
            name: intrfc.name,
            description: [
              {
                title: 'TYPE',
                value: intrfc.status.device ? intrfc.status.device : '---'
              },
              {
                title: 'IP ADDRESS',
                value: Object.prototype.hasOwnProperty.call(intrfc.status, ['ipv4-address'][0]) ? intrfc.status['ipv4-address'][0].address : '---'
              }]
          })
        })
      })
    },
    async getNetworkLog () {
      // Network Log
      const log = await this.$rpc.ubus('log', 'read_db', { table: 'NETWORK' }).then(response => {
        return response.log
      })
      this.netlog = log
    },

    async getSystemLog () {
      // System Log
      const log = await this.$rpc.call('system', 'syslog')
      this.syslog = log.log
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
  computed: {
    latestLogs () {
      return this.netlog.slice(-this.eventsCount).reverse()
    },
    latestSysLogs () {
      return this.syslog.slice(-this.eventsCount).reverse()
    }
  },
  async created () {
    await this.getSystemInfo()
    await this.getNetworkInterfaces()
    await this.getSystemLog()
    await this.getNetworkLog()
    this.creatingFinalObject()
    console.log(this.cardsData)
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
