<template>
  <div class="phone_app">
    <PhoneTitle :title="IntlString('APP_ADMIN_TITLE')" @back="onBackspace"/>
    <div class='phone_content elements'>
      <div class='element'
          v-for='(elem, key) in paramList' 
          v-bind:class="{ select: key === currentSelect}"
          v-bind:key="key"
          @click.stop="onPressItem(key)"  
        >
        <i class="fa" v-bind:class="elem.icons" v-bind:style="{color: elem.color}" @click.stop="onPressItem(key)"></i>
        <div class="element-content" @click.stop="onPressItem(key)">
          <span class="element-title" @click.stop="onPressItem(key)">{{elem.title}}</span>
          <span v-if="elem.value" class="element-value" @click.stop="onPressItem(key)">{{elem.value}}</span>
        </div>
      </div>
    </div>
    
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import PhoneTitle from './../PhoneTitle'
import Modal from '@/components/Modal/index.js'

export default {
  components: {
    PhoneTitle
  },
  data () {
    return {
      ignoreControls: false,
      currentSelect: 0,
      isBilps: 'off',
      menu: []
    }
  },
  computed: {
    ...mapGetters(['IntlString', 'useMouse', 'myPhoneNumber', 'backgroundLabel', 'coqueLabel', 'sonidoLabel', 'zoom', 'config', 'volume', 'availableLanguages']),
    paramList () {
      const cancelStr = this.IntlString('CANCEL')
      const confirmResetStr = this.IntlString('APP_ADMIN_RESET_CONFIRM')
      const cancelOption = {}
      const confirmReset = {}
      cancelOption[cancelStr] = 'cancel'
      confirmReset[confirmResetStr] = 'accept'
      return this.menu
    },
    valumeDisplay () {
      return `${Math.floor(this.volume * 100)} %`
    }
  },
  methods: {
    ...mapActions(['getIntlString', 'setZoon', 'setBackground', 'setCoque', 'setSonido', 'setVolume', 'setLanguage', 'setMouseSupport']),
    setControlAction: function (params, data) {
      this.$phoneAPI.sendControlAction({'method': data.value})
    },
    scrollIntoViewIfNeeded: function () {
      this.$nextTick(() => {
        document.querySelector('.select').scrollIntoViewIfNeeded()
      })
    },
    onBackspace () {
      if (this.ignoreControls === true) return
      this.$router.push({ name: 'home' })
    },
    onUp: function () {
      if (this.ignoreControls === true) return
      this.currentSelect = this.currentSelect === 0 ? this.paramList.length - 1 : this.currentSelect - 1
      this.scrollIntoViewIfNeeded()
    },
    onDown: function () {
      if (this.ignoreControls === true) return
      this.currentSelect = this.currentSelect === this.paramList.length - 1 ? 0 : this.currentSelect + 1
      this.scrollIntoViewIfNeeded()
    },
    onRight () {
      if (this.ignoreControls === true) return
      let param = this.paramList[this.currentSelect]
      if (param.onRight !== undefined) {
        param.onRight(param)
      }
    },
    onLeft () {
      if (this.ignoreControls === true) return
      let param = this.paramList[this.currentSelect]
      if (param.onLeft !== undefined) {
        param.onLeft(param)
      }
    },
    actionItem (param) {
      if (param.values !== undefined) {
        this.ignoreControls = true
        let choix = Object.keys(param.values).map(key => {
          return {title: param.values[key], value: key, picto: param.values[key]}
        })
        Modal.CreateModal({choix}).then(reponse => {
          this.ignoreControls = false
          if (reponse.title === 'cancel') return
          this[param.onValid](param, reponse)
        })
      }
    },
    onPressItem (index) {
      this.actionItem(this.paramList[index])
    },
    onEnter () {
      if (this.ignoreControls === true) return
      this.actionItem(this.paramList[this.currentSelect])
    },
    async onChangeBackground (param, data) {
      let val = data.value
      if (val === 'URL') {
        this.ignoreControls = true
        Modal.CreateTextModal({
          text: 'https://i.imgur.com/'
        }).then(valueText => {
          if (valueText.text !== '' && valueText.text !== undefined && valueText.text !== null && valueText.text !== 'https://i.imgur.com/') {
            this.setBackground({
              label: 'Custom',
              value: valueText.text
            })
          }
        }).finally(() => {
          this.ignoreControls = false
        })
      } else {
        this.setBackground({
          label: data.title,
          value: data.value
        })
      }
    },
    onChangeCoque: function (param, data) {
      this.setCoque({
        label: data.title,
        value: data.value
      })
    },

    onChangeSonido: function (param, data) {
      this.setSonido({
        label: data.title,
        value: data.value
      })
    },

    setZoom: function (param, data) {
      this.setZoon(data.value)
    },
    ajustZoom (inc) {
      return () => {
        const percent = Math.max(10, (parseInt(this.zoom) || 100) + inc)
        this.setZoon(`${percent}%`)
      }
    },
    setPhoneVolume (param, data) {
      this.setVolume(data.value)
    },
    ajustVolume (inc) {
      return () => {
        const newVolume = Math.max(0, Math.min(1, parseFloat(this.volume) + inc))
        this.setVolume(newVolume)
      }
    },
    onChangeLanguages (param, data) {
      if (data.value !== 'cancel') {
        this.setLanguage(data.value)
      }
    },
    onChangeMouseSupport (param, data) {
      if (data.value !== 'cancel') {
        this.setMouseSupport(data.value)
        this.onBackspace()
      }
    },
    resetPhone: function (param, data) {
      if (data.value !== 'cancel') {
        this.ignoreControls = true
        const cancelStr = this.IntlString('CANCEL')
        const confirmResetStr = this.IntlString('APP_ADMIN_RESET_CONFIRM')
        let choix = [{title: cancelStr}, {title: cancelStr}, {title: confirmResetStr, color: 'red', reset: true}, {title: cancelStr}, {title: cancelStr}]
        Modal.CreateModal({choix}).then(reponse => {
          this.ignoreControls = false
          if (reponse.reset === true) {
            this.$phoneAPI.deleteALL()
          }
        })
      }
    }
  },

  created () {
    if (!this.useMouse) {
      this.$bus.$on('keyUpArrowRight', this.onRight)
      this.$bus.$on('keyUpArrowLeft', this.onLeft)
      this.$bus.$on('keyUpArrowDown', this.onDown)
      this.$bus.$on('keyUpArrowUp', this.onUp)
      this.$bus.$on('keyUpEnter', this.onEnter)
    } else {
      this.currentSelect = -1
    }
    this.$bus.$on('keyUpBackspace', this.onBackspace)

    this.$phoneAPI.getPermissions().then(data => {
      console.log(data)
    })
    this.menu.push({
      icons: 'fa-cloud',
      title: '무적 ON/OFF',
      onValid: 'setControlAction',
      values: {
        'ch_godmode': '실행'
      }
    })
    this.menu.push({
      icons: 'fa-cloud',
      title: '빌립스',
      onValid: 'setControlAction',
      values: {
        'ch_blips': '실행'
      }
    })
    this.menu.push({
      icons: 'fa-cloud',
      title: '노클립',
      onValid: 'setControlAction',
      values: {
        'ch_noclip': '실행'
      }
    })
    this.menu.push({
      icons: 'fa-cloud',
      title: '텔레포트',
      onValid: 'setControlAction',
      values: {
        'ch_tpto': '상대에게 텔레포트',
        'ch_tptome': '나에게 텔레포트',
        'choice_tptowaypoint': '웨이포인트로 텔레포트',
        'ch_tptocoords': '좌표로 텔레포트',
        'ch_coords': '좌표 확인'
      }
    })
    this.menu.push({
      icons: 'fa-cloud',
      title: '플레이어',
      onValid: 'setControlAction',
      values: {
        'ch_addgroup': '권한 추가',
        'ch_removegroup': '권한 삭제',
        'vrp_arevive_a_revive': '부활',
        'ch_kick': '플레이어 킥',
        'ch_ban': '플레이어 벤',
        'ch_unban': '플레이어 벤해제',
        'ch_freeze': '플레이어 얼리기'
      }
    })
    this.menu.push({
      icons: 'fa-cloud',
      title: '생성',
      onValid: 'setControlAction',
      values: {
        'ch_giveitem': '아이템 생성',
        'ch_givemoney': '현금 생성',
        'ch_givebank': '계좌잔고 생성'
      }
    })
    this.menu.push({
      icons: 'fa-cloud',
      title: '서버 제어',
      onValid: 'setControlAction',
      values: {
        'ch_reboot_start': '리붓프로세스 시작',
        'ch_reboot_stop': '리붓프로세스 종료'
      }
    })
    this.menu.push({
      icons: 'fa-cloud',
      title: '관리자 공지',
      onValid: 'setControlAction',
      values: {
        'ch_announces_admin': '공지 알람'
      }
    })
  },
  beforeDestroy () {
    this.$bus.$off('keyUpArrowRight', this.onRight)
    this.$bus.$off('keyUpArrowLeft', this.onLeft)
    this.$bus.$off('keyUpArrowDown', this.onDown)
    this.$bus.$off('keyUpArrowUp', this.onUp)
    this.$bus.$off('keyUpEnter', this.onEnter)
    this.$bus.$off('keyUpBackspace', this.onBackspace)
  }
}
</script>

<style scoped>
.element{
  height: 58px;
  line-height: 58px;
  display: flex;
  align-items: center;
  position: relative;
}
.element .fa{
  color: #0b81ff;
  margin-left: 6px;
  height: 52px;
  width: 52px;
  text-align: center;
  line-height: 52px;
}
.element-content{
  display: block;
  height: 58px;
  width: 100%;
  margin-left: 6px;
  display: flex;
  flex-flow: column;
  justify-content: center;
}
.element-title{
  display: block;
  margin-top: 4px;
  height: 22px;
  line-height: 22px;
  font-size: 20px;
  font-weight: 300;
  font-family: -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;
}
.element-value{
  display: block;
  line-height: 16px;
  height: 8px;
  font-size: 14px;
  font-weight: 100;
  color: #808080;
}
.element.select, .element:hover{
   background-color: #DDD;
}
</style>
