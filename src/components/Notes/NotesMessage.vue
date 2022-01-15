<template>
  <div style="width: 334px; height: 742px; background: white" class="phone_app">
    <PhoneTitle :title="channelName" backgroundColor="#f8d344" @back="onQuit"/>
    <div class="phone_content">
      <div class="elements" ref="elementsDiv">
          <div class="element" v-for='(elem) in notesMessages' 
            v-bind:key="elem.id"
            >
            <div class="time">{{formatTime(elem.time)}}</div>
            <div class="name">[{{elem.user_id}}] {{elem.name}}</div>
            <div class="message">
              {{elem.message}}
            </div>
          </div>
      </div>

      <div class='notes_write'>
          <input type="text" placeholder="..." v-model="message" @keyup.enter.prevent="sendMessage">
          <span class='notes_send' @click="sendMessage">></span>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import PhoneTitle from './../PhoneTitle'

export default {
  components: { PhoneTitle },
  data () {
    return {
      message: '',
      channel: '',
      currentSelect: 0,
      isAutoScroll: true
    }
  },
  computed: {
    ...mapGetters(['notesMessages', 'notesCurrentChannel', 'useMouse']),
    channelName () {
      return '# ' + this.channel
    }
  },
  watch: {
    notesMessages () {
      let _this = this
      setTimeout(function () {
        if (_this.isAutoScroll) {
          const c = _this.$refs.elementsDiv
          c.scrollTop = c.scrollHeight
        }
      }, 50)
    }
  },
  methods: {
    setChannel (channel) {
      this.channel = channel
      this.notesSetChannel({ channel })
    },
    ...mapActions(['notesSetChannel', 'notesSendMessage']),
    scrollIntoViewIfNeeded () {
      this.$nextTick(() => {
        const $select = this.$el.querySelector('.select')
        if ($select !== null) {
          $select.scrollIntoViewIfNeeded()
        }
      })
    },
    onUp () {
      const c = this.$refs.elementsDiv
      c.scrollTop = c.scrollTop - 120
      this.isAutoScroll = false
    },
    onDown () {
      const c = this.$refs.elementsDiv
      c.scrollTop = c.scrollTop + 120
      console.log(c.scrollTop, c.scrollHeight)
      if (c.scrollTop + 600 >= c.scrollHeight) {
        this.isAutoScroll = true
      }
    },
    async onEnter () {
      const rep = await this.$phoneAPI.getReponseText()
      if (rep !== undefined && rep.text !== undefined) {
        const message = rep.text.trim()
        if (message.length !== 0) {
          this.notesSendMessage({
            channel: this.channel,
            message: message
          })
        }
      }
    },
    sendMessage () {
      const message = this.message.trim()
      if (message.length !== 0) {
        this.notesSendMessage({
          channel: this.channel,
          message: message
        })
        this.message = ''
      }
    },
    onBack () {
      if (this.useMouse === true && document.activeElement.tagName !== 'BODY') return
      this.onQuit()
    },
    onQuit () {
      this.$router.push({ name: 'notes' })
    },
    formatTime (time) {
      const d = new Date(time)
      return d.toLocaleTimeString()
    }
  },
  created () {
    if (!this.useMouse) {
      this.$bus.$on('keyUpArrowDown', this.onDown)
      this.$bus.$on('keyUpArrowUp', this.onUp)
      this.$bus.$on('keyUpEnter', this.onEnter)
    } else {
      this.currentSelect = -1
    }
    this.$bus.$on('keyUpBackspace', this.onBack)
    this.setChannel(this.$route.params.channel)
  },
  mounted () {
    window.c = this.$refs.elementsDiv
    const c = this.$refs.elementsDiv
    c.scrollTop = c.scrollHeight
  },
  beforeDestroy () {
    this.$bus.$off('keyUpArrowDown', this.onDown)
    this.$bus.$off('keyUpArrowUp', this.onUp)
    this.$bus.$off('keyUpEnter', this.onEnter)
    this.$bus.$off('keyUpBackspace', this.onBack)
  }
}
</script>

<style scoped>

.elements{
  height: calc(100% - 56px);
  background-color: #dae0e6;
  color: white;
  display: flex;
  flex-direction: column;
  padding: 0 10px 12px 10px;
  overflow-y: auto;
}

.element{
  color: #a6a28c;
  flex: 0 0 auto;
  flex-wrap:wrap;
  width: 100%;
  display: flex;
  margin: 0 0 5px 0;
  padding:3px 10px;
  line-height: 18px;
  font-size: 18px;
  flex-direction: row;
  border-radius:5px;
  background-color:rgba(255,255,255,0.8)
}
.time{
  width:50%;
  font-size: 12px;
  text-align:left;
}
.name{
  width:50%;
  font-size: 14px;
  text-align:right;
}
.message{
  width: 100%;
  color: black;
  font-size: 14px;
  color:#333;
  word-break: break-all;
}

.notes_write{
    height: 56px;
    widows: 100%;
    background: #dae0e6;
    display: flex;
    justify-content: space-around;
    align-items: center;
}
.notes_write input{
     width: 75%;
    margin-left: 6%;
    border: none;
    outline: none;
    font-size: 16px;
    padding: 3px 5px;
    float: left;
    height: 36px;
    background-color: white;
    color: black;
}
.notes_write input::placeholder {
  color: #ccc;
}
.notes_send{
    width: 32px;
    height: 32px;
    float: right;
    border-radius: 50%;
    background-color: #f8d344;
    margin-right: 12px;
    margin-bottom: 2px;
    color: white;
    line-height: 32px;
    text-align: center;
}
.elements::-webkit-scrollbar-track
  {
      box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
      background-color: #a6a28c;
  }
.elements::-webkit-scrollbar
  {
      width: 3px;
      background-color: transparent;
  }
.elements::-webkit-scrollbar-thumb
  {
      background-color: #FFC629;
  }
</style>
