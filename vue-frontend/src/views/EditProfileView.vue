<template>
  <div class="page">
    <div class="grid-bg"></div>
    <div class="glow-top"></div>

    <nav class="nav">
      <a class="logo" @click="router.push('/')">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
          <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"
            stroke="#30d158" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        KcalPath
      </a>
      <button class="nb-back" @click="router.push('/dashboard')">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none">
          <path d="M19 12H5M5 12l7-7M5 12l7 7" stroke="currentColor" stroke-width="2"
            stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        กลับ Dashboard
      </button>
    </nav>

    <div class="scroll-wrap">
      <div class="container">

        <!-- PAGE HEADER -->
        <div class="page-head">
          <div class="head-left">
            <div class="eyebrow"><span class="ew-dot"></span>ตั้งค่าโปรไฟล์</div>
            <h1 class="page-title">แก้ไขข้อมูล<em>ส่วนตัว</em></h1>
            <p class="page-sub">ข้อมูลที่แม่นยำช่วยให้ระบบคำนวณ TDEE และแนะนำโภชนาการได้ตรงกับคุณมากขึ้น</p>
          </div>
          <transition name="pills">
            <div class="preview-row" v-if="previewReady">
              <div class="pv-pill" v-for="s in liveStats" :key="s.label">
                <div class="pvp-val" :style="`color:${s.color}`">{{ s.val }}</div>
                <div class="pvp-lbl">{{ s.label }}</div>
              </div>
              <div class="pv-pill">
                <div class="pvp-val" :style="`color:${bmiColor}`">{{ liveBMI }}</div>
                <div class="pvp-lbl">BMI · {{ bmiText }}</div>
              </div>
            </div>
          </transition>
        </div>

        <!-- 01 ข้อมูลร่างกาย -->
        <div class="section">
          <div class="sec-head">
            <span class="sec-num">01</span>
            <span class="sec-title">ข้อมูลร่างกาย</span>
          </div>
          <div class="metrics-row">
            <div class="field">
              <label>น้ำหนัก <span class="unit">กก.</span></label>
              <div class="iw" :class="{ focused: focused==='weight' }">
                <svg class="ii" width="14" height="14" viewBox="0 0 24 24" fill="none">
                  <path d="M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z" stroke="currentColor" stroke-width="2"/>
                  <line x1="3" y1="6" x2="21" y2="6" stroke="currentColor" stroke-width="2"/>
                </svg>
                <input type="number" v-model="form.weight" placeholder="65" class="inp"
                  @focus="focused='weight'" @blur="focused=''" @input="calcPreview"/>
              </div>
            </div>
            <div class="field">
              <label>ส่วนสูง <span class="unit">ซม.</span></label>
              <div class="iw" :class="{ focused: focused==='height' }">
                <svg class="ii" width="14" height="14" viewBox="0 0 24 24" fill="none">
                  <path d="M12 2v20M8 5l4-3 4 3M8 19l4 3 4-3" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                </svg>
                <input type="number" v-model="form.height" placeholder="170" class="inp"
                  @focus="focused='height'" @blur="focused=''" @input="calcPreview"/>
              </div>
            </div>
            <div class="field">
              <label>อายุ <span class="unit">ปี</span></label>
              <div class="iw" :class="{ focused: focused==='age' }">
                <svg class="ii" width="14" height="14" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="2"/>
                  <path d="M12 6v6l3 3" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                </svg>
                <input type="number" v-model="form.age" placeholder="25" class="inp"
                  @focus="focused='age'" @blur="focused=''" @input="calcPreview"/>
              </div>
            </div>
          </div>
        </div>

        <!-- 02 เพศ -->
        <div class="section">
          <div class="sec-head">
            <span class="sec-num">02</span>
            <span class="sec-title">เพศ</span>
          </div>
          <div class="gender-row">
            <button class="gender-btn" :class="{ active: form.gender==='male' }"
              @click="form.gender='male'; calcPreview()">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
                <circle cx="10" cy="14" r="5" stroke="currentColor" stroke-width="1.8"/>
                <path d="M15 9l5-5M20 4h-4M20 4v4" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
              </svg>
              ชาย
            </button>
            <button class="gender-btn" :class="{ active: form.gender==='female' }"
              @click="form.gender='female'; calcPreview()">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
                <circle cx="12" cy="9" r="5" stroke="currentColor" stroke-width="1.8"/>
                <path d="M12 14v6M9 18h6" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
              </svg>
              หญิง
            </button>
          </div>
        </div>

        <!-- 03 ระดับกิจกรรม -->
        <div class="section">
          <div class="sec-head">
            <span class="sec-num">03</span>
            <span class="sec-title">ระดับการออกกำลังกาย</span>
          </div>
          <div class="act-list">
            <button class="act-btn" :class="{ active: form.activity_level===a.val }"
              v-for="a in activityOptions" :key="a.val"
              @click="form.activity_level=a.val; calcPreview()">
              <div class="act-icon">
                <svg v-if="a.val==='low'" width="22" height="22" viewBox="0 0 24 24" fill="none">
                  <rect x="3" y="8" width="18" height="12" rx="2" stroke="currentColor" stroke-width="1.7"/>
                  <path d="M7 8V6a2 2 0 0 1 2-2h6a2 2 0 0 1 2 2v2" stroke="currentColor" stroke-width="1.7" stroke-linecap="round"/>
                  <path d="M8 14h8M8 17h5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                </svg>
                <svg v-if="a.val==='medium'" width="22" height="22" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="4" r="2" stroke="currentColor" stroke-width="1.7"/>
                  <path d="M12 6v5M8 10l4 1 4-1M10 11l-2 5M14 11l2 5M8 16l2 3M16 16l-2 3" stroke="currentColor" stroke-width="1.7" stroke-linecap="round"/>
                </svg>
                <svg v-if="a.val==='high'" width="22" height="22" viewBox="0 0 24 24" fill="none">
                  <circle cx="12" cy="4" r="2" stroke="currentColor" stroke-width="1.7"/>
                  <path d="M5 10h4l1-2h4l1 2h4" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"/>
                  <path d="M9 10l-1 8M15 10l1 8M8 18h3M13 18h3" stroke="currentColor" stroke-width="1.7" stroke-linecap="round"/>
                </svg>
              </div>
              <div class="act-text">
                <div class="act-title">{{ a.label }}</div>
                <div class="act-desc">{{ a.desc }}</div>
              </div>
              <div class="act-check" v-if="form.activity_level===a.val">
                <svg width="10" height="10" viewBox="0 0 24 24" fill="none">
                  <path d="M20 6L9 17l-5-5" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
            </button>
          </div>
        </div>

        <!-- 04 เป้าหมาย -->
        <div class="section">
          <div class="sec-head">
            <span class="sec-num">04</span>
            <span class="sec-title">เป้าหมายของคุณ</span>
          </div>
          <div class="goal-row">
            <button class="goal-btn" :class="{ active: form.goal===g.val }"
              v-for="g in goalOptions" :key="g.val"
              @click="form.goal=g.val">
              <div class="goal-icon">
                <svg v-if="g.val==='lose'" width="22" height="22" viewBox="0 0 24 24" fill="none">
                  <polyline points="22 17 13.5 8.5 8.5 13.5 2 7" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                  <polyline points="16 17 22 17 22 11" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                <svg v-if="g.val==='maintain'" width="22" height="22" viewBox="0 0 24 24" fill="none">
                  <path d="M5 12h14M12 5l7 7-7 7" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                <svg v-if="g.val==='gain'" width="22" height="22" viewBox="0 0 24 24" fill="none">
                  <polyline points="22 7 13.5 15.5 8.5 10.5 2 17" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                  <polyline points="16 7 22 7 22 13" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
              <div class="goal-label">{{ g.label }}</div>
              <div class="goal-desc">{{ g.desc }}</div>
            </button>
          </div>
        </div>

        <!-- ACTIONS -->
        <div class="actions">
          <button class="btn-cancel" @click="router.push('/dashboard')">ยกเลิก</button>
          <button class="btn-submit" @click="updateProfile" :disabled="saving">
            <span v-if="!saving">บันทึกการเปลี่ยนแปลง</span>
            <span v-else class="ldots"><span></span><span></span><span></span></span>
          </button>
        </div>

      </div>
    </div>

    <transition name="toast">
      <div class="toast" :class="toast.type" v-if="toast.show">{{ toast.msg }}</div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router  = useRouter()
const saving  = ref(false)
const focused = ref('')
const previewReady = ref(false)

const form = ref({ weight:'', height:'', age:'', gender:'male', activity_level:'low', goal:'maintain' })

const activityOptions = [
  { val:'low',    label:'น้อย',    desc:'ทำงานนั่งโต๊ะ ไม่ค่อยออกกำลังกาย' },
  { val:'medium', label:'ปานกลาง', desc:'ออกกำลังกาย 3–5 วัน/สัปดาห์' },
  { val:'high',   label:'มาก',     desc:'ออกกำลังกายหนัก 6–7 วัน/สัปดาห์' },
]
const goalOptions = [
  { val:'lose',     label:'ลดน้ำหนัก',      desc:'ลดไขมัน' },
  { val:'maintain', label:'รักษาน้ำหนัก',   desc:'สุขภาพดี' },
  { val:'gain',     label:'เพิ่มกล้ามเนื้อ', desc:'เพิ่มน้ำหนัก' },
]

const liveBMR  = ref(0)
const liveTDEE = ref(0)
const liveBMI  = ref(0)

const bmiText = computed(() => {
  const b = liveBMI.value
  if (!b) return ''
  if (b < 18.5) return 'น้ำหนักน้อย'
  if (b < 23)   return 'สมส่วน'
  if (b < 25)   return 'น้ำหนักเกิน'
  return 'โรคอ้วน'
})
const bmiColor = computed(() => {
  const b = liveBMI.value
  if (!b)       return '#6e6e73'
  if (b < 18.5) return '#ff9f0a'
  if (b < 23)   return '#30d158'
  if (b < 25)   return '#ff9f0a'
  return '#ff453a'
})
const liveStats = computed(() => [
  { label:'BMR',  val: liveBMR.value  || '—', color:'#0a84ff' },
  { label:'TDEE', val: liveTDEE.value || '—', color:'#30d158' },
])

function calcPreview() {
  const { weight, height, age, gender, activity_level } = form.value
  if (!weight || !height || !age) { previewReady.value = false; return }
  const h = height / 100
  liveBMI.value  = +(weight / (h * h)).toFixed(1)
  let bmr = 10 * weight + 6.25 * height - 5 * age + (gender === 'male' ? 5 : -161)
  liveBMR.value  = Math.round(bmr)
  liveTDEE.value = Math.round(bmr * ({ low:1.2, medium:1.375, high:1.55 }[activity_level] || 1.2))
  previewReady.value = true
}

const toast = ref({ show:false, msg:'', type:'ok' })
function showToast(msg, type='ok') {
  toast.value = { show:true, msg, type }
  setTimeout(() => toast.value.show = false, 2800)
}

onMounted(async () => {
  const userId = localStorage.getItem('userId')
  if (!userId) { router.push('/login'); return }
  try {
    const res = await fetch(`/api/users/${userId}`)
    if (res.ok) { form.value = { ...form.value, ...await res.json() }; calcPreview() }
  } catch (e) { console.error(e) }
})

async function updateProfile() {
  const userId = localStorage.getItem('userId')
  saving.value = true
  try {
    const res = await fetch(`/api/users/${userId}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    })
    if (res.ok) {
      showToast('บันทึกข้อมูลเรียบร้อยแล้ว', 'ok')
      setTimeout(() => router.push('/dashboard'), 1000)
    } else { showToast('บันทึกไม่สำเร็จ กรุณาลองใหม่', 'err') }
  } catch { showToast('เชื่อมต่อเซิร์ฟเวอร์ไม่ได้', 'err') }
  finally { saving.value = false }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;700;900&family=Noto+Sans+Thai:wght@300;400;600&display=swap');
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

.page {
  font-family: 'Noto Sans Thai','Prompt',sans-serif;
  background: #000; color: #f5f5f7; min-height: 100vh;
  overflow-x: hidden; -webkit-font-smoothing: antialiased;
}
.grid-bg {
  position: fixed; inset: 0; pointer-events: none; z-index: 0;
  background-image:
    linear-gradient(rgba(255,255,255,.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,.03) 1px, transparent 1px);
  background-size: 72px 72px;
  mask-image: radial-gradient(ellipse 100% 70% at 50% 0%, black 20%, transparent 100%);
}
.glow-top {
  position: fixed; inset: 0; pointer-events: none; z-index: 0;
  background: radial-gradient(ellipse 55% 28% at 50% 0%, rgba(48,209,88,.07) 0%, transparent 70%);
}

/* NAV */
.nav {
  position: sticky; top: 0; z-index: 99;
  height: 52px; display: flex; align-items: center; justify-content: space-between;
  padding: 0 28px;
  background: rgba(0,0,0,.8); backdrop-filter: blur(28px);
  border-bottom: 1px solid rgba(255,255,255,.06);
}
.logo {
  font-family:'Prompt',sans-serif; font-weight:700; font-size:.95rem;
  color:#f5f5f7; display:flex; align-items:center; gap:7px; cursor:pointer;
}
.nb-back {
  display:flex; align-items:center; gap:6px;
  font-family:inherit; font-size:.78rem; color:#a1a1a6;
  background:rgba(255,255,255,.05); border:1px solid rgba(255,255,255,.08);
  border-radius:8px; padding:6px 14px; cursor:pointer; transition:all .2s;
}
.nb-back:hover { background:rgba(255,255,255,.09); color:#f5f5f7; }

/* SCROLL */
.scroll-wrap {
  position: relative; z-index: 1;
  display: flex; justify-content: center;
  padding: 48px 20px 80px;
  min-height: calc(100vh - 52px);
}
.container { width: 100%; max-width: 640px; display: flex; flex-direction: column; gap: 12px; }

/* PAGE HEAD */
.page-head {
  display: flex; align-items: flex-start; justify-content: space-between;
  gap: 20px; flex-wrap: wrap; margin-bottom: 8px;
}
.eyebrow {
  display: inline-flex; align-items: center; gap: 7px;
  font-size: .7rem; font-weight: 600; color: #30d158;
  letter-spacing: .12em; text-transform: uppercase; margin-bottom: 10px;
}
.ew-dot { width:6px; height:6px; border-radius:50%; background:#30d158; box-shadow:0 0 8px #30d158; }
.page-title {
  font-family:'Prompt',sans-serif; font-weight:700;
  font-size: clamp(1.8rem, 4.5vw, 2.8rem);
  line-height:1.05; letter-spacing:-.04em; margin-bottom:8px;
}
.page-title em {
  font-style:normal;
  background:linear-gradient(135deg,#30d158,#34aadc);
  -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text;
}
.page-sub { font-size:.82rem; font-weight:300; color:#6e6e73; line-height:1.75; max-width:320px; }

/* preview pills */
.preview-row { display:flex; gap:8px; flex-wrap:wrap; padding-top:4px; }
.pv-pill {
  display:flex; flex-direction:column; align-items:center; gap:2px;
  background:rgba(255,255,255,.04); border:1px solid rgba(255,255,255,.08);
  border-radius:12px; padding:10px 18px; min-width:70px;
}
.pvp-val { font-family:'Prompt',sans-serif; font-size:1.1rem; font-weight:700; line-height:1; }
.pvp-lbl { font-size:.58rem; color:#6e6e73; white-space:nowrap; }
.pills-enter-active,.pills-leave-active{transition:all .35s ease}
.pills-enter-from,.pills-leave-to{opacity:0;transform:translateY(-6px)}

/* SECTION */
.section {
  background: rgba(255,255,255,.025);
  border: 1px solid rgba(255,255,255,.07);
  border-radius: 18px; padding: 20px 22px 22px;
}
.sec-head { display:flex; align-items:center; gap:9px; margin-bottom:16px; }
.sec-num {
  font-family:'Prompt',sans-serif; font-size:.65rem; font-weight:700;
  color:#30d158; letter-spacing:.1em;
}
.sec-title { font-family:'Prompt',sans-serif; font-size:.88rem; font-weight:600; color:#f5f5f7; }

/* METRICS */
.metrics-row { display:grid; grid-template-columns:repeat(3,minmax(0,1fr)); gap:10px; }
.field { display:flex; flex-direction:column; gap:6px; min-width:0; }
label { font-size:.72rem; color:#a1a1a6; }
.unit { color:#6e6e73; font-size:.65rem; }
.iw {
  display:flex; align-items:center; gap:7px;
  background:rgba(255,255,255,.05); border:1px solid rgba(255,255,255,.09);
  border-radius:11px; padding:0 11px; height:46px; transition:all .22s;
  min-width:0; overflow:hidden;
}
.iw.focused {
  border-color:rgba(48,209,88,.45); background:rgba(48,209,88,.04);
  box-shadow:0 0 0 3px rgba(48,209,88,.07);
}
.ii { color:#6e6e73; flex-shrink:0; transition:color .22s; }
.iw.focused .ii { color:#30d158; }
.inp {
  flex:1; min-width:0; background:none; border:none; outline:none;
  font-family:'Noto Sans Thai','Prompt',sans-serif;
  font-size:.9rem; color:#f5f5f7; color-scheme:dark;
}
.inp::placeholder { color:#3a3a3f; }

/* GENDER */
.gender-row { display:grid; grid-template-columns:1fr 1fr; gap:10px; }
.gender-btn {
  display:flex; align-items:center; justify-content:center; gap:9px;
  font-family:inherit; font-size:.9rem; font-weight:500; color:#6e6e73;
  background:rgba(255,255,255,.04); border:1px solid rgba(255,255,255,.08);
  border-radius:12px; padding:15px; cursor:pointer; transition:all .22s;
}
.gender-btn.active { background:rgba(48,209,88,.09); border-color:rgba(48,209,88,.32); color:#30d158; }
.gender-btn:hover:not(.active) { background:rgba(255,255,255,.07); color:#a1a1a6; }

/* ACTIVITY */
.act-list { display:flex; flex-direction:column; gap:8px; }
.act-btn {
  display:flex; align-items:center; gap:14px;
  font-family:inherit; text-align:left;
  background:rgba(255,255,255,.03); border:1px solid rgba(255,255,255,.07);
  border-radius:13px; padding:14px 16px; cursor:pointer; transition:all .22s;
}
.act-btn.active { background:rgba(48,209,88,.07); border-color:rgba(48,209,88,.26); }
.act-btn:hover:not(.active) { background:rgba(255,255,255,.06); }
.act-icon { color:#6e6e73; flex-shrink:0; transition:color .22s; }
.act-btn.active .act-icon { color:#30d158; }
.act-text { flex:1; min-width:0; }
.act-title { font-size:.86rem; font-weight:600; color:#f5f5f7; margin-bottom:2px; }
.act-desc  { font-size:.7rem; color:#6e6e73; font-weight:300; }
.act-check {
  width:22px; height:22px; flex-shrink:0; border-radius:7px;
  background:rgba(48,209,88,.16); border:1px solid rgba(48,209,88,.3);
  color:#30d158; display:flex; align-items:center; justify-content:center;
}

/* GOAL */
.goal-row { display:grid; grid-template-columns:repeat(3,minmax(0,1fr)); gap:10px; }
.goal-btn {
  display:flex; flex-direction:column; align-items:center; gap:7px;
  font-family:inherit; text-align:center;
  background:rgba(255,255,255,.03); border:1px solid rgba(255,255,255,.07);
  border-radius:13px; padding:18px 10px; cursor:pointer; transition:all .22s;
}
.goal-btn.active { background:rgba(48,209,88,.07); border-color:rgba(48,209,88,.26); }
.goal-btn:hover:not(.active) { background:rgba(255,255,255,.07); }
.goal-icon { color:#6e6e73; transition:color .22s; }
.goal-btn.active .goal-icon { color:#30d158; }
.goal-label { font-size:.82rem; font-weight:600; color:#f5f5f7; }
.goal-desc  { font-size:.65rem; color:#6e6e73; }

/* ACTIONS */
.actions { display:flex; gap:10px; padding-top:4px; }
.btn-cancel {
  flex:1; font-family:inherit; font-size:.9rem; color:#6e6e73;
  background:rgba(255,255,255,.04); border:1px solid rgba(255,255,255,.08);
  border-radius:13px; padding:15px; cursor:pointer; transition:all .2s;
}
.btn-cancel:hover { background:rgba(255,255,255,.08); color:#a1a1a6; }
.btn-submit {
  flex:2; font-family:'Prompt',inherit; font-size:.9rem; font-weight:600;
  background:#30d158; color:#000; border:none; border-radius:13px; padding:15px;
  cursor:pointer; transition:all .22s;
  display:flex; align-items:center; justify-content:center;
}
.btn-submit:hover:not(:disabled) { background:#28b84c; }
.btn-submit:disabled { opacity:.6; cursor:not-allowed; }

.ldots { display:flex; gap:4px; align-items:center; }
.ldots span { width:5px; height:5px; border-radius:50%; background:#000; animation:dot 1.2s ease-in-out infinite; }
.ldots span:nth-child(2){animation-delay:.2s}
.ldots span:nth-child(3){animation-delay:.4s}
@keyframes dot{0%,80%,100%{transform:scale(.5);opacity:.3}40%{transform:scale(1);opacity:1}}

/* TOAST */
.toast {
  position:fixed; bottom:28px; left:50%; transform:translateX(-50%); z-index:999;
  font-family:'Prompt',sans-serif; font-size:.85rem; font-weight:500;
  padding:11px 24px; border-radius:980px; box-shadow:0 8px 32px rgba(0,0,0,.5); white-space:nowrap;
}
.toast.ok  { background:#30d158; color:#000; }
.toast.err { background:#ff453a; color:#fff; }
.toast-enter-active,.toast-leave-active{transition:all .32s cubic-bezier(.22,1,.36,1)}
.toast-enter-from,.toast-leave-to{opacity:0;transform:translateX(-50%) translateY(12px)}

/* RESPONSIVE */
@media (max-width:480px) {
  .scroll-wrap { padding:28px 14px 60px; }
  .page-head { flex-direction:column; }
  .metrics-row { grid-template-columns:1fr 1fr; }
  .goal-row { grid-template-columns:1fr 1fr; }
}
</style>