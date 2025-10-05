<template>
  <div class="site" :class="{ 'is-loading': loading }">
    <!-- === Preloader: дождь сердечек === -->
    <div v-if="loading" class="preloader" aria-hidden="true">
      <div class="preloader-inner">
        <div class="heart-rain">
          <span v-for="n in 22" :key="'h'+n" class="drop" :style="dropStyle(n)">❤</span>
        </div>
        <div class="loading-text">Загружаю любовь…</div>
      </div>
    </div>

    <!-- === Hero с 3D-parallax и улучшенным сердцем === -->
    <section class="hero" @mousemove="onMove" @mouseleave="resetMove">
      <div class="parallax" ref="parallax">
        <!-- Глубокий слой: звёзды -->
        <div class="layer layer-back" :style="layerStyle(-35)">
          <div class="stars" ref="stars"></div>
        </div>
        <!-- Средний слой: мягкие светящиеся пятна -->
        <div class="layer layer-mid" :style="layerStyle(-15)">
          <div class="blob blob-1"></div>
          <div class="blob blob-2"></div>
        </div>
        <!-- Передний слой: сердце + счётчик -->
        <div class="layer layer-front" :style="layerStyle(20)">
          <div class="hero-inner">
            <h1 class="title">Мы вместе уже</h1>
            <div class="counter">
              <div class="unit">
                <div class="num">{{ elapsed.days }}</div>
                <div class="label">дней</div>
              </div>
              <div class="sep">·</div>
              <div class="unit">
                <div class="num">{{ pad2(elapsed.hours) }}</div>
                <div class="label">часов</div>
              </div>
              <div class="sep">:</div>
              <div class="unit">
                <div class="num">{{ pad2(elapsed.minutes) }}</div>
                <div class="label">минут</div>
              </div>
              <div class="sep">:</div>
              <div class="unit">
                <div class="num">{{ pad2(elapsed.seconds) }}</div>
                <div class="label">секунд</div>
              </div>
            </div>
            <p class="subtitle">С самого {{ formatDate(startDate) }}</p>
          </div>

          <!-- SVG-сердце с пульсацией, глубиной и искрами -->
          <div class="heart-wrap" aria-hidden="true">
            <svg class="heart-svg" viewBox="0 0 200 180">
              <defs>
                <radialGradient id="g" cx="35%" cy="35%" r="65%">
                  <stop offset="0%" stop-color="#ff9fbc"/>
                  <stop offset="60%" stop-color="#ff4f86"/>
                  <stop offset="100%" stop-color="#ff1f6a"/>
                </radialGradient>
                <filter id="glow" x="-50%" y="-50%" width="200%" height="200%">
                  <feGaussianBlur stdDeviation="6" result="b"/>
                  <feMerge>
                    <feMergeNode in="b"/>
                    <feMergeNode in="SourceGraphic"/>
                  </feMerge>
                </filter>
              </defs>
              <path class="heart-path" filter="url(#glow)" fill="url(#g)"
                d="M100 165 C 40 120, 15 95, 22 60 C 28 30, 55 18, 75 25 C 90 30, 100 45, 100 55 C 100 45, 110 30, 125 25 C 145 18, 172 30, 178 60 C 185 95, 160 120, 100 165 Z"/>
            </svg>
            <div class="sparks">
              <span v-for="s in 10" :key="'s'+s" class="spark" :style="sparkStyle(s)"></span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- === Сообщения с анимацией === -->
    <section class="messages">
      <h2>Почему я тебя люблю</h2>
      <transition name="fade-slide" mode="out-in">
        <p class="message" :key="activeMessageIndex">{{ activeMessage }}</p>
      </transition>
      <div class="chip-row" style="display:none"></div>

      <form class="add-form" @submit.prevent="addCustomMessage">
        <input v-model="newCustomMessage" class="input" type="text" placeholder="Добавить своё тёплое сообщение" maxlength="140" />
        <button class="btn" :disabled="!newCustomMessage">Добавить</button>
      </form>
      <div class="messages-list" v-if="customMessages && customMessages.length">
      <ul class="list">
        <li v-for="(m,i) in customMessages" :key="'m'+i" class="list-item">
          <span class="dot">❤</span>
          <span class="txt">{{ m }}</span>
          <button class="icon-btn" title="Удалить" @click="removeCustomMessage(i)">🗑</button>
        </li>
      </ul>
    </div>
  </section>

    <!-- === Виджет обратного отсчёта === -->
    <section class="countdown">
      <h2>Ждём важный момент</h2>
      <form class="add-form" @submit.prevent="saveCountdown">
        <input v-model="countdownTitle" class="input" type="text" placeholder="Название события (например, Годовщина)" />
        <input v-model="countdownDate" class="input" type="datetime-local" />
        <button class="btn">Сохранить</button>
      </form>

      <div v-if="countdown && countdown.ts" class="countdown-box">
        <h3>{{ countdown.title }}</h3>
        <div class="counter small">
          <div class="unit"><div class="num">{{ countdownLeft.days }}</div><div class="label">дней</div></div>
          <div class="sep">·</div>
          <div class="unit"><div class="num">{{ pad2(countdownLeft.hours) }}</div><div class="label">часов</div></div>
          <div class="sep">:</div>
          <div class="unit"><div class="num">{{ pad2(countdownLeft.minutes) }}</div><div class="label">минут</div></div>
          <div class="sep">:</div>
          <div class="unit"><div class="num">{{ pad2(countdownLeft.seconds) }}</div><div class="label">секунд</div></div>
        </div>
        <div class="date-note">до {{ formatDateTime(new Date(countdown.ts)) }}</div>
      </div>
    </section>

    <!-- === Письма в будущее === -->
    <section class="letters">
      <h2>Письма в будущее</h2>
      <form class="add-form" @submit.prevent="addLetter">
        <textarea v-model="letterText" class="input" rows="4" placeholder="Напиши письмо нам в будущее…"></textarea>
        <input v-model="letterRevealAt" class="input" type="datetime-local" />
        <button class="btn" :disabled="!letterText || !letterRevealAt">Запечатать ✉️</button>
      </form>

      <div class="letters-grid">
        <div v-for="(l, i) in letters" :key="l.id" class="letter-card" :class="{ unlocked: now >= l.revealAt }">
          <div class="letter-head">
            <div class="when">Откроется: {{ formatDateTime(new Date(l.revealAt)) }}</div>
            <button class="icon-btn" title="Удалить" @click="removeLetter(i)">🗑</button>
          </div>
          <div v-if="now < l.revealAt" class="locked">
            <div class="lock">🔒</div>
            <div class="eta">Осталось: {{ leftTo(l.revealAt) }}</div>
          </div>
          <div v-else class="content">{{ l.text }}</div>
        </div>
      </div>
    </section>

    <footer class="footer">Сделано с ❤ специально для нас</footer>
  </div>
</template>

<script>
export default {
  name: 'LoveCountersAndMagic',
  props: { startDate: { type: String, required: true } },
  data(){
    return {
      loading: true,
      now: Date.now(),
      timerId: null,
      tiltX: 0, tiltY: 0,
      messages: [],
      customMessages: [],
      activeMessageIndex: 0,
      newCustomMessage: '',
      countdownTitle: '',
      countdownDate: '',
      countdown: null,
      letterText: '',
      letterRevealAt: '',
      letters: []
    }
  },
  computed: {
    startTs(){
      var t = Date.parse(this.startDate)
      if (isNaN(t)) return Date.now()
      return t
    },
    elapsed(){
      var diff = Math.max(0, Math.floor((this.now - this.startTs) / 1000))
      var days = Math.floor(diff / 86400)
      diff -= days * 86400
      var hours = Math.floor(diff / 3600)
      diff -= hours * 3600
      var minutes = Math.floor(diff / 60)
      var seconds = diff - minutes * 60
      return { days: days, hours: hours, minutes: minutes, seconds: seconds }
    },
    activeMessage(){ var pool = this.customMessages; if (pool.length === 0) return ''; var idx = this.activeMessageIndex % pool.length; if (idx < 0) idx = idx + pool.length; return pool[idx] },
    countdownLeft(){
      if (!this.countdown || !this.countdown.ts) return { days: 0, hours: 0, minutes: 0, seconds: 0 }
      var left = Math.max(0, Math.floor((this.countdown.ts - this.now) / 1000))
      var days = Math.floor(left / 86400)
      left -= days * 86400
      var hours = Math.floor(left / 3600)
      left -= hours * 3600
      var minutes = Math.floor(left / 60)
      var seconds = left - minutes * 60
      return { days: days, hours: hours, minutes: minutes, seconds: seconds }
    }
  },
  mounted(){
    this.loadState()
    var self = this
    this.timerId = setInterval(function(){ self.now = Date.now() }, 1000)
    // Наполняем звёзды
    this.initStars()
    // Красивый выход из прелоадера
    setTimeout(function(){ self.loading = false }, 800)
  },
  beforeUnmount(){ if (this.timerId) clearInterval(this.timerId) },
  methods: {
    // ——— Parallax ———
    onMove(e){
      var rect = this.$refs.parallax ? this.$refs.parallax.getBoundingClientRect() : null
      if (!rect) return
      var cx = rect.left + rect.width/2
      var cy = rect.top + rect.height/2
      var dx = (e.clientX - cx) / rect.width
      var dy = (e.clientY - cy) / rect.height
      this.tiltX = dy * -18
      this.tiltY = dx * 18
    },
    resetMove(){ this.tiltX = 0; this.tiltY = 0 },
    layerStyle(z){
      var tx = this.tiltX; var ty = this.tiltY
      var rx = ' rotateX(' + tx + 'deg)'
      var ry = ' rotateY(' + ty + 'deg)'
      var tz = ' translateZ(' + z + 'px)'
      return { transform: tz + rx + ry }
    },

    // ——— UI helpers ———
    pad2(n){ return (n < 10 ? '0' : '') + n },
    formatDate(d){ var dt = new Date(d); var dd = (dt.getDate()<10?'0':'') + dt.getDate(); var mm = (dt.getMonth()+1<10?'0':'') + (dt.getMonth()+1); var yyyy = dt.getFullYear(); return dd + '.' + mm + '.' + yyyy },
    formatDateTime(dt){ var dd = (dt.getDate()<10?'0':'') + dt.getDate(); var mm = (dt.getMonth()+1<10?'0':'') + (dt.getMonth()+1); var yyyy = dt.getFullYear(); var HH = (dt.getHours()<10?'0':'') + dt.getHours(); var II = (dt.getMinutes()<10?'0':'') + dt.getMinutes(); return dd + '.' + mm + '.' + yyyy + ' ' + HH + ':' + II },

    // ——— Messages ———
    nextMessage(){ this.activeMessageIndex += 1 },
    prevMessage(){ this.activeMessageIndex -= 1 },
    
    addCustomMessage(){ var msg = (this.newCustomMessage || '').trim(); if (!msg) return; this.customMessages.push(msg); this.activeMessageIndex = this.customMessages.length - 1; this.newCustomMessage = ''; this.persist('customMessages', this.customMessages) },

    // ——— Countdown ———
    saveCountdown(){ var title = (this.countdownTitle || 'Событие').trim(); var ts = Date.parse(this.countdownDate); if (isNaN(ts)) return; this.countdown = { title: title, ts: ts }; this.persist('countdown', this.countdown) },

    // ——— Letters ———
    addLetter(){ var txt = (this.letterText || '').trim(); var ts = Date.parse(this.letterRevealAt); if (!txt || isNaN(ts)) return; var letter = { id: 'L' + Math.random().toString(36).slice(2), text: txt, revealAt: ts }; this.letters.unshift(letter); this.letterText = ''; this.letterRevealAt = ''; this.persist('letters', this.letters) },
    removeLetter(i){ if (i < 0 || i >= this.letters.length) return; this.letters.splice(i, 1); this.persist('letters', this.letters) },
    removeCustomMessage(i){ if (i < 0 || i >= this.customMessages.length) return; this.customMessages.splice(i,1); if(this.activeMessageIndex >= this.customMessages.length){ this.activeMessageIndex = this.customMessages.length - 1 } this.persist('customMessages', this.customMessages) },
    leftTo(ts){ var left = Math.max(0, Math.floor((ts - this.now) / 1000)); var d = Math.floor(left / 86400); left -= d * 86400; var h = Math.floor(left / 3600); left -= h * 3600; var m = Math.floor(left / 60); var s = left - m * 60; return (d + 'д ') + this.pad2(h) + ':' + this.pad2(m) + ':' + this.pad2(s) },

    // ——— LocalStorage ———
    persist(key, value){ try{ localStorage.setItem('love.' + key, JSON.stringify(value)) }catch(e){} },
    loadState(){ try{ var cm = localStorage.getItem('love.customMessages'); if (cm) this.customMessages = JSON.parse(cm); var cd = localStorage.getItem('love.countdown'); if (cd) this.countdown = JSON.parse(cd); var ls = localStorage.getItem('love.letters'); if (ls) this.letters = JSON.parse(ls) }catch(e){} },

    // ——— Stars & Sparks ———
    initStars(){
      var el = this.$refs.stars
      if (!el) return
      var i = 0
      while(i < 90){
        var s = document.createElement('span')
        var size = Math.random()*2 + 1
        s.className = 'star'
        s.style.left = Math.floor(Math.random()*100) + '%'
        s.style.top = Math.floor(Math.random()*100) + '%'
        s.style.width = size + 'px'
        s.style.height = size + 'px'
        s.style.animationDelay = (Math.random()*4) + 's'
        el.appendChild(s)
        i++
      }
    },
    sparkStyle(i){
      var angle = (i * 36) % 360
      var dist = 40 + (i % 5) * 6
      var t = 'translate(-50%, -50%) rotate(' + angle + 'deg) translateY(-' + dist + 'px)'
      var d = (i % 5) * 0.12
      return { transform: t, animationDelay: d + 's' }
    },
    dropStyle(i){
      var d = (i % 7) * 0.15
      var l = Math.floor(Math.random()*100) + '%'
      var s = 14 + (i % 5) * 4
      return { left: l, fontSize: s + 'px', animationDelay: d + 's' }
    }
  }
}
</script>

<style scoped>
/* ===== Цвета и фон ===== */
.site{
  --bg:#0b1220; --panel:#0f1a2b; --soft:#172135; --text:#e6ecff; --muted:#9db2d7; --accent:#ff5c8a; --accent2:#7c3aed;
  min-height:100vh; color:var(--text);
  background:
    radial-gradient(1200px 600px at 20% -10%, rgba(124,58,237,.22), transparent 60%),
    radial-gradient(1000px 500px at 90% 10%, rgba(255,92,138,.22), transparent 60%),
    var(--bg);
  font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Inter, Arial, sans-serif;
}
section{ padding: 56px 20px; max-width: 1040px; margin: 0 auto; }
.title{ font-size: clamp(28px, 4vw, 46px); margin: 0 0 8px; letter-spacing: .5px; }
.subtitle{ color: var(--muted); margin: 8px 0 0; }

/* ===== Preloader ===== */
.preloader{ position: fixed; inset:0; display:grid; place-items:center; background: rgba(11,18,32,.9); z-index: 9999; overflow: hidden; }
.preloader-inner{ position: relative; width: 100%; height: 100%; display:grid; place-items:center }
.loading-text{ position:absolute; bottom: 10vh; font-weight:600; letter-spacing:.3px; color:var(--muted) }
.heart-rain{ position:absolute; inset:0; pointer-events:none }
.drop{ position:absolute; top:-10vh; animation: fall 2.4s linear infinite; opacity:.9; text-shadow: 0 0 8px rgba(255,92,138,.6) }
@keyframes fall{ 0%{ transform: translateY(-10vh) scale(.8); opacity: 0 } 10%{ opacity:1 } 100%{ transform: translateY(120vh) scale(1.2); opacity:0 } }

/* Плавный уход прелоадера */
.site:not(.is-loading) .preloader{ display:none }

/* ===== Hero + Parallax ===== */
.hero{ perspective: 900px; position:relative; overflow:hidden; }
.parallax{ position:relative; height: 64vh; min-height:520px }
.layer{ position:absolute; inset:0; transform-style: preserve-3d; transition: transform .12s ease-out }
.layer-front{ display:grid; place-items:center }
.hero-inner{ position:relative; z-index:5; text-align:center; margin-top: 24px; }
.counter{ display:flex; align-items:flex-end; gap:14px; flex-wrap:wrap; justify-content:center }
.counter.small{ gap:10px }
.unit{ text-align:center }
.num{ font-size: clamp(34px, 6vw, 68px); font-weight: 800; line-height:1; }
.counter.small .num{ font-size: clamp(24px, 4vw, 40px) }
.label{ color: var(--muted); font-size: .9rem; margin-top: 4px; }
.sep{ font-size: clamp(28px, 5vw, 40px); opacity:.6; transform: translateY(-4px) }

/* Звёзды */
.stars{ position:absolute; inset:0; z-index:0 }
.star{ position:absolute; display:block; background:#fff; border-radius:50%; opacity:.9; animation: twinkle 3.5s ease-in-out infinite; }
@keyframes twinkle { 0%,100%{ opacity:.15; transform: scale(.9)} 50%{ opacity:1; transform: scale(1.35)} }

/* Световые пятна */
.blob{ position:absolute; filter: blur(60px); opacity:.35 }
.blob-1{ width:420px; height:420px; background: radial-gradient(circle at 40% 40%, #7c3aed, transparent 60%); left:6%; top:8% }
.blob-2{ width:460px; height:460px; background: radial-gradient(circle at 60% 60%, #ff5c8a, transparent 60%); right:4%; bottom:6% }

/* Сердце */
.heart-wrap{ position:absolute; inset:0; display:grid; place-items:center; z-index:1; pointer-events:none }
.heart-svg{ width: 220px; filter: drop-shadow(0 12px 40px rgba(255,92,138,.45)); animation: wobble 6s ease-in-out infinite }
.heart-path{ transform-origin: 50% 60%; animation: beat 1.8s ease-in-out infinite }
@keyframes beat{ 0%,100%{ transform: scale(1)} 30%{ transform: scale(1.06) } 45%{ transform: scale(.98)} 60%{ transform: scale(1.05) } }
@keyframes wobble{ 0%,100%{ transform: rotateY(0deg) } 50%{ transform: rotateY(18deg) } }

/* Искры вокруг сердца */
.sparks{ position:absolute; width: 220px; height: 220px; left:50%; top:50%; transform: translate(-50%,-55%); pointer-events:none }
.spark{ position:absolute; left:50%; top:50%; width:6px; height:6px; background: linear-gradient(90deg,#fff,rgba(255,255,255,.2)); border-radius:999px; box-shadow: 0 0 12px rgba(255,255,255,.8); animation: sparkle 1.6s ease-in-out infinite }
@keyframes sparkle{ 0%{ opacity:0; transform: scale(.6)} 25%{ opacity:1 } 100%{ opacity:0; transform: scale(1) } }

/* ===== Messages ===== */
.messages .message{ font-size: clamp(18px, 3vw, 28px); margin: 16px 0 10px; min-height: 1.6em }
.fade-slide-enter-active, .fade-slide-leave-active{ transition: all .4s ease }
.fade-slide-enter-from{ opacity:0; transform: translateY(6px) }
.fade-slide-leave-to{ opacity:0; transform: translateY(-6px) }

.chip-row{ display:none }

.messages-list .list{ list-style:none; padding:0; margin: 12px auto 0; max-width: 680px }
.messages-list .list-item{ display:flex; align-items:center; gap:10px; background:var(--panel); border:1px solid rgba(255,255,255,.08); padding:10px 12px; border-radius:12px; margin-top:8px }
.messages-list .dot{ opacity:.7 }
.messages-list .txt{ flex:1 }

.chip{ background:var(--soft); color:var(--text); border:1px solid rgba(255,255,255,.08); padding:8px 12px; border-radius:999px; cursor:pointer }
.chip:hover{ background:#1e2a45 }

.add-form{ display:flex; gap:10px; margin-top:16px; flex-wrap:wrap; justify-content:center }
.input{ flex:1 1 260px; background:var(--panel); border:1px solid rgba(255,255,255,.1); color:var(--text); padding:12px 14px; border-radius:12px; outline:none }
.input:focus{ border-color: rgba(255,92,138,.45); box-shadow: 0 0 0 3px rgba(255,92,138,.15) }
.btn{ background: linear-gradient(90deg, var(--accent), var(--accent2)); border:none; color:white; padding:12px 16px; border-radius:12px; cursor:pointer; font-weight:600 }
.btn:disabled{ filter:grayscale(1); opacity:.6; cursor:not-allowed }

/* ===== Countdown ===== */
.countdown-box{ margin-top:16px; background:var(--soft); border:1px solid rgba(255,255,255,.08); padding:16px; border-radius:16px }
.date-note{ color:var(--muted); margin-top:8px }

/* ===== Letters ===== */
.letters-grid{ display:grid; grid-template-columns: repeat(auto-fill, minmax(260px, 1fr)); gap:14px; margin-top:16px }
.letter-card{ background:var(--panel); border:1px solid rgba(255,255,255,.08); border-radius:16px; padding:14px; display:flex; flex-direction:column; gap:10px }
.letter-card.unlocked{ border-color: rgba(124,58,237,.35) }
.letter-head{ display:flex; align-items:center; justify-content:space-between; color:var(--muted); font-size:.95rem }
.icon-btn{ background:transparent; border:none; color:var(--muted); cursor:pointer; font-size:18px }
.lock{ font-size:28px; margin-bottom:8px }
.locked{ color:var(--muted) }
.content{ white-space:pre-wrap; line-height:1.4 }

.footer{ text-align:center; color:var(--muted); padding:24px }
body{
  margin:0;
}
</style>
