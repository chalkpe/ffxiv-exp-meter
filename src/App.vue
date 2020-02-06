<template>
  <div id="app">
    <article v-if="err">
      웹소켓 연결에 실패했습니다…<br>
      오버레이 주소의 HOST_PORT 값을 다시 확인하세요.
    </article>
    <article v-else>
      <p class="title">
        경험치 상태
        <span v-if="paused">&nbsp;(기록 중지됨)</span>
      </p>
      <p>누적 경험치 획득량: {{ Math.floor(points) }}</p>
      <p>현재 경험치 증가율: {{ boost }}%</p>
    </article>
  </div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  color: white;
  font-size: 14px;
  font-family: 'Noto Sans KR', sans-serif;
  text-shadow: 0 0 0.1rem #0090ce, 0 0 0.1rem #0090ce, 0 0 0.1rem #0090ce, 0 0 0.1rem #0090ce;
}

.title {
  font-size: 1.05rem;
  text-shadow: 0 0 0.1rem #ffc72f, 0 0 0.1rem #ffc72f;
}

.title span {
  text-shadow: inherit;
}
</style>

<script>
const KEY = 'expm'

export default {
  data: () => ({
    err: null,
    points: 0,
    boost: 0,
    paused: false
  }),

  mounted () {
    const m = /[?&]HOST_PORT=(wss?:\/\/[^&]+)/.exec(window.location.search)
    if (m !== null) this.initACTWebSocket(new WebSocket(m[1] + 'BeforeLogLineRead'))
    else document.addEventListener('onLogLine', e => this.onLogLine(JSON.parse(e.detail)))

    this.points = this.load('points', 0)
    this.boost = this.load('boost', 0)
  },

  methods: {
    load (key, def, decode = JSON.parse) {
      return decode(localStorage.getItem(`${KEY}-${key}`)) || def
    },

    save (key, val, encode = JSON.stringify) {
      localStorage.setItem(`${KEY}-${key}`, encode(val))
      return val
    },

    initACTWebSocket (ws) {
      window.addEventListener('unload', (ws.onerror = e => (this.err = (ws.close(), e))))
      ws.onmessage = ({ data }) => {
        if (data === '.') return ws.send('.')
        const { type, msgtype, msg } = JSON.parse(data)
        if (type === 'broadcast' && msgtype === 'Chat') this.onLogLine(msg.split('|'))
      }
    },

    onLogLine (e) {
      if (e[0] === '00' && e[2] === '0840') {
        const m = /\D+(\d+)\s*\(\+(\d+)%\)/.exec(e[4])
        return m && !this.paused && this.onExperienceGained(parseInt(m[1], 10), parseInt(m[2], 10))
      }

      if (e[0] === '00' && e[2] === '0038') {
        const m = e[4].toLowerCase().split(' ')
        return m[0] === KEY && this.onCommand(m.slice(1))
      }
    },

    onExperienceGained (point, boost) {
      if (!Number.isSafeInteger(point) || !Number.isSafeInteger(boost)) return
      this.save('points', this.points += this.boost / 100 * point / (1 + boost / 100))
    },

    onCommand (args) {
      const actions = ({
        pause: () => (this.paused = true),
        resume: () => (this.paused = false),
        reset: () => this.save('points', (this.points = 0)),
        boost: ([boost]) => {
          const b = parseInt(boost, 10)
          if (Number.isSafeInteger(b)) this.save('boost', (this.boost = b))
        }
      })

      actions[args[0]] && actions[args[0]](args.slice(1))
    }
  }
}
</script>