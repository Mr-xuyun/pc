<template>
  <a-config-provider :locale="locale">
    <div id="app">
      <router-view />
    </div>
  </a-config-provider>
</template>
<script>
import zhCN from 'ant-design-vue/lib/locale-provider/zh_CN'
import enquireScreen from '@/utils/device'

export default {
  data() {
    return {
      locale: zhCN
    }
  },
  created() {
    let that = this
    enquireScreen(deviceType => {
      // tablet
      if (deviceType === 0) {
        that.$store.commit('TOGGLE_DEVICE', 'mobile')
        that.$store.dispatch('setSidebar', false)
      }
      // mobile
      else if (deviceType === 1) {
        that.$store.commit('TOGGLE_DEVICE', 'mobile')
        that.$store.dispatch('setSidebar', false)
      } else {
        that.$store.commit('TOGGLE_DEVICE', 'desktop')
        that.$store.dispatch('setSidebar', true)
      }

    })
  }
}
</script>
<style>
#app {
  height: 100%;
}

.cark1 {
  background-color: #fff;
  border-radius: 10px;
  padding: 10px 10px;
}

.sdxqtitle {
  height: 40px;
  background-image: linear-gradient(to right, #1B8FFF, #6BB5FF);
  text-align: center;
  line-height: 40px;
  font-size: 18px;
  font-weight: 550;
  color: #fff;
  margin: 10px 10px;
  border-radius: 10px;
}
#app .ant-pagination {
  text-align: right;
  margin-top: 10px;
}
</style>