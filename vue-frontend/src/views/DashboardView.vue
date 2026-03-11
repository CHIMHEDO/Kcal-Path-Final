<template>
  <div class="page">

    <!-- backgrounds -->
    <div class="grid-bg"></div>
    <div class="glow-top"></div>

    <!-- ══ NAV ══ -->
    <nav class="nav" :class="{ scrolled }">
      <a class="logo" @click="router.push('/')">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
          <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"
            stroke="#30d158" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        KcalPath
      </a>
      <div class="nav-center">
        <div class="nav-greeting">
          สวัสดี, <span>{{ userName }}</span>
        </div>
        <div class="nav-date">{{ todayDateDisplay }}</div>
      </div>
      <div class="nav-end">
        <button class="nb-icon" title="แก้ไขโปรไฟล์" @click="router.push('/edit-profile')">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
            <circle cx="12" cy="8" r="4" stroke="currentColor" stroke-width="2"/>
            <path d="M4 20c0-4 3.6-7 8-7s8 3 8 7" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>
          โปรไฟล์
        </button>
        <button class="nb-icon" @click="router.push('/history')">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
            <path d="M12 8v4l3 3" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
            <circle cx="12" cy="12" r="9" stroke="currentColor" stroke-width="2"/>
          </svg>
          ประวัติ
        </button>
        <button class="nb-logout" @click="logout">ออกจากระบบ</button>
      </div>
    </nav>

    <!-- ══ CONTENT ══ -->
    <div class="content">

      <!-- ─── ROW 1: kcal ring + macros ─── -->
      <div class="row-hero">

        <!-- KCAL RING CARD -->
        <div class="card card-kcal" v-observe>
          <div class="card-label">
            <span class="label-dot" :style="`background:${totalCalories > userTDEE ? '#ff453a' : '#30d158'}`"></span>
            เป้าหมายแคลอรี่วันนี้
          </div>

          <div class="ring-wrap">
            <svg class="ring-svg" viewBox="0 0 200 200">
              <!-- track -->
              <circle cx="100" cy="100" r="80"
                fill="none" stroke="rgba(255,255,255,.06)" stroke-width="14"/>
              <!-- fill -->
              <circle cx="100" cy="100" r="80"
                fill="none"
                :stroke="totalCalories > userTDEE ? '#ff453a' : '#30d158'"
                stroke-width="14" stroke-linecap="round"
                :stroke-dasharray="`${Math.min((totalCalories/(userTDEE||1)),1)*502} 502`"
                stroke-dashoffset="125"
                :style="`filter:drop-shadow(0 0 8px ${totalCalories > userTDEE ? '#ff453a' : '#30d158'}44)`"
                class="ring-arc"/>
            </svg>
            <div class="ring-inner">
              <div class="ring-val" :class="{ danger: totalCalories > userTDEE }">
                {{ totalCalories }}
              </div>
              <div class="ring-unit">/ {{ userTDEE }} kcal</div>
              <div class="ring-pct" :class="{ danger: totalCalories > userTDEE }">
                {{ Math.round((totalCalories / (userTDEE || 1)) * 100) }}%
              </div>
            </div>
          </div>

          <div class="kcal-status" :class="totalCalories > userTDEE ? 'st-danger' : 'st-ok'">
            <span v-if="totalCalories > userTDEE">
              ⚠ เกินเป้าหมาย {{ totalCalories - userTDEE }} kcal
            </span>
            <span v-else>
              ✓ เหลืออีก {{ userTDEE - totalCalories }} kcal
            </span>
          </div>
        </div>

        <!-- MACROS CARD -->
        <div class="card card-macros" v-observe style="transition-delay:.08s">
          <div class="card-label">
            <span class="label-dot" style="background:#0a84ff"></span>
            สารอาหาร (Macros)
          </div>

          <div class="macro-list">
            <div class="mac" v-for="m in macroDisplay" :key="m.name">
              <div class="mac-top">
                <span class="mac-name">{{ m.name }}</span>
                <span class="mac-val" :style="`color:${m.color}`">{{ m.val }}<span>g</span></span>
              </div>
              <div class="mac-track">
                <div class="mac-fill"
                  :style="`width:${Math.min(m.pct,100)}%;background:${m.color}`">
                </div>
              </div>
              <div class="mac-bot">
                <span class="mac-pct">{{ Math.round(m.pct) }}% ของเป้าหมาย</span>
                <span class="mac-target">เป้า {{ m.target }}g</span>
              </div>
            </div>
          </div>
        </div>

        <!-- BODY STATS CARD -->
        <div class="card card-body" v-observe style="transition-delay:.16s">
          <div class="card-label">
            <span class="label-dot" style="background:#ff9f0a"></span>
            ข้อมูลร่างกาย
          </div>
          <div class="body-grid">
            <div class="bg-item" v-for="b in bodyStats" :key="b.label">
              <div class="bg-val" :style="b.color ? `color:${b.color}` : ''">{{ b.val }}</div>
              <div class="bg-label">{{ b.label }}</div>
              <div class="bg-sub" v-if="b.sub">{{ b.sub }}</div>
            </div>
          </div>
        </div>

      </div>

      <!-- ─── ROW 2: food logs ─── -->
      <div class="card card-logs" v-observe style="transition-delay:.1s">
        <div class="logs-header">
          <div>
            <div class="card-label" style="margin-bottom:4px">
              <span class="label-dot" style="background:#30d158"></span>
              รายการอาหารวันนี้
            </div>
            <div class="logs-count">{{ foodLogs.length }} รายการ</div>
          </div>
          <button class="btn-add" @click="router.push('/add-food')">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none">
              <path d="M12 5v14M5 12h14" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"/>
            </svg>
            เพิ่มมื้ออาหาร
          </button>
        </div>

        <!-- empty state -->
        <div class="empty-state" v-if="foodLogs.length === 0">
          <div class="es-icon">🍽</div>
          <div class="es-title">ยังไม่มีรายการอาหาร</div>
          <div class="es-sub">เริ่มบันทึกมื้อแรกของวันนี้กันเลย</div>
          <button class="btn-add" @click="router.push('/add-food')" style="margin-top:16px">
            + เพิ่มมื้ออาหาร
          </button>
        </div>

        <!-- log list -->
        <div class="log-list" v-else>

          <!-- group by meal -->
          <div v-for="meal in mealGroups" :key="meal.key" class="meal-group">
            <div class="mg-header">
              <span class="mg-name">{{ meal.label }}</span>
              <span class="mg-kcal">{{ meal.totalKcal }} kcal</span>
            </div>
            <div class="log-item" v-for="food in meal.items" :key="food.log_id">
              <div class="li-left">
                <div class="li-name">{{ food.food_name }}</div>
                <div class="li-macros">
                  <span style="color:#0a84ff">P {{ food.total_protein }}g</span>
                  <span style="color:#ff9f0a">C {{ food.total_carbs }}g</span>
                  <span style="color:#ff453a">F {{ food.total_fat }}g</span>
                </div>
              </div>
              <div class="li-right">
                <div class="li-kcal">{{ food.total_calories }}<span>kcal</span></div>
                <button class="btn-del" @click="deleteLog(food.log_id)">
                  <svg width="13" height="13" viewBox="0 0 24 24" fill="none">
                    <path d="M18 6L6 18M6 6l12 12" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                  </svg>
                </button>
              </div>
            </div>
          </div>

        </div>
      </div>

    </div>

    <!-- loading overlay -->
    <transition name="fade">
      <div class="loading-overlay" v-if="pageLoading">
        <div class="loading-dots">
          <span></span><span></span><span></span>
        </div>
      </div>
    </transition>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'

const router    = useRouter()
const scrolled  = ref(false)
const pageLoading = ref(true)

// ── user data ──
const userName   = ref('...')
const userWeight = ref(0)
const userHeight = ref(0)
const userAge    = ref(0)
const userGender = ref('')
const userGoal   = ref('')
const userBMI    = ref(0)
const bmiText    = ref('')
const userBMR    = ref(0)
const userTDEE   = ref(0)

// ── food ──
const todayDateDisplay = ref(new Date().toLocaleDateString('th-TH', { dateStyle: 'long' }))
const totalCalories    = ref(0)
const totalProtein     = ref(0)
const totalCarbs       = ref(0)
const totalFat         = ref(0)
const foodLogs         = ref([])

// ── macros display ──
const macroDisplay = computed(() => [
  { name: 'โปรตีน', val: totalProtein.value, target: 150, pct: (totalProtein.value / 150) * 100, color: '#0a84ff' },
  { name: 'คาร์บ',  val: totalCarbs.value,   target: 250, pct: (totalCarbs.value   / 250) * 100, color: '#ff9f0a' },
  { name: 'ไขมัน',  val: totalFat.value,     target: 70,  pct: (totalFat.value     / 70)  * 100, color: '#ff453a' },
])

// ── body stats ──
const bodyStats = computed(() => [
  { label: 'BMI',       val: userBMI.value, sub: bmiText.value,   color: bmiColor(userBMI.value) },
  { label: 'BMR',       val: `${userBMR.value}`, sub: 'kcal/วัน', color: '' },
  { label: 'TDEE',      val: `${userTDEE.value}`, sub: 'kcal/วัน', color: '#30d158' },
  { label: 'เป้าหมาย', val: goalText(userGoal.value), sub: '', color: '#0a84ff' },
  { label: 'น้ำหนัก',  val: `${userWeight.value}`, sub: 'กก.', color: '' },
  { label: 'ส่วนสูง',  val: `${userHeight.value}`, sub: 'ซม.', color: '' },
])

function bmiColor(b) {
  if (b < 18.5) return '#ff9f0a'
  if (b < 23)   return '#30d158'
  if (b < 25)   return '#ff9f0a'
  return '#ff453a'
}
function goalText(g) {
  return { lose: 'ลดน้ำหนัก', maintain: 'รักษาน้ำหนัก', gain: 'เพิ่มกล้ามเนื้อ' }[g] || g
}

// ── meal groups ──
const mealOrder = ['breakfast','lunch','dinner','snack']
const mealLabel = { breakfast:'มื้อเช้า', lunch:'มื้อเที่ยง', dinner:'มื้อเย็น', snack:'ของว่าง' }

const mealGroups = computed(() => {
  const groups = {}
  foodLogs.value.forEach(f => {
    if (!groups[f.meal_type]) groups[f.meal_type] = []
    groups[f.meal_type].push(f)
  })
  return mealOrder
    .filter(k => groups[k]?.length)
    .map(k => ({
      key: k,
      label: mealLabel[k],
      items: groups[k],
      totalKcal: groups[k].reduce((s, f) => s + (f.total_calories || 0), 0),
    }))
})

// ── health calc ──
function calculateHealth(weight, height, age, gender, activity) {
  const h = height / 100
  const bmi = weight / (h * h)
  userBMI.value = +bmi.toFixed(1)
  bmiText.value = bmi < 18.5 ? 'น้ำหนักน้อย' : bmi < 23 ? 'สมส่วน' : bmi < 25 ? 'น้ำหนักเกิน' : 'โรคอ้วน'

  let bmr = 10 * weight + 6.25 * height - 5 * age
  bmr += (gender === 'male' || gender === 'ชาย') ? 5 : -161
  userBMR.value = Math.round(bmr)

  const mult = { low: 1.2, medium: 1.375, high: 1.55, น้อย: 1.2, ปานกลาง: 1.375, มาก: 1.55 }
  userTDEE.value = Math.round(bmr * (mult[activity] || 1.2))
}

// ── fetch ──
async function fetchAll() {
  const userId = localStorage.getItem('userId')
  if (!userId) { router.push('/login'); return }

  try {
    const uRes = await fetch(`http://localhost:3000/api/users/${userId}`)
    if (uRes.ok) {
      const u = await uRes.json()
      userName.value   = u.username
      userWeight.value = u.weight
      userHeight.value = u.height
      userAge.value    = u.age
      userGender.value = u.gender
      userGoal.value   = u.goal
      calculateHealth(u.weight, u.height, u.age, u.gender, u.activity_level)
    }

    const today = new Intl.DateTimeFormat('en-CA', {
      timeZone: 'Asia/Bangkok', year: 'numeric', month: '2-digit', day: '2-digit'
    }).format(new Date())

    const sRes = await fetch(`http://localhost:3000/api/daily-summary/${userId}/${today}`)
    if (sRes.ok) {
      const s = await sRes.json()
      totalCalories.value = s.summary.total_calories || 0
      totalProtein.value  = s.summary.total_protein  || 0
      totalCarbs.value    = s.summary.total_carbs    || 0
      totalFat.value      = s.summary.total_fat      || 0
      foodLogs.value      = s.logs || []
    }
  } catch (e) {
    console.error(e)
  } finally {
    pageLoading.value = false
  }
}

async function deleteLog(id) {
  if (!confirm('ลบรายการนี้?')) return
  try {
    const r = await fetch(`http://localhost:3000/api/food-logs/${id}`, { method: 'DELETE' })
    if (r.ok) fetchAll()
  } catch (e) { console.error(e) }
}

function logout() {
  localStorage.clear()
  router.push('/')
}

// ── lifecycle ──
const onScroll = () => scrolled.value = window.scrollY > 10
onMounted(() => { window.addEventListener('scroll', onScroll); fetchAll() })
onUnmounted(() => window.removeEventListener('scroll', onScroll))

// ── scroll reveal directive ──
const vObserve = {
  mounted(el) {
    el.style.opacity = '0'
    el.style.transform = 'translateY(28px)'
    el.style.transition = 'opacity .85s cubic-bezier(.22,1,.36,1), transform .85s cubic-bezier(.22,1,.36,1)'
    const io = new IntersectionObserver(([e]) => {
      if (e.isIntersecting) { el.style.opacity = '1'; el.style.transform = 'none'; io.disconnect() }
    }, { threshold: .06 })
    io.observe(el)
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Prompt:wght@200;300;400;700;900&family=Noto+Sans+Thai:wght@200;300;400;600&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

.page {
  font-family: 'Noto Sans Thai', 'Prompt', sans-serif;
  background: #000; color: #f5f5f7;
  min-height: 100vh; position: relative;
  overflow-x: hidden; -webkit-font-smoothing: antialiased;
}

/* ── backgrounds ── */
.grid-bg {
  position: fixed; inset: 0; pointer-events: none; z-index: 0;
  background-image:
    linear-gradient(rgba(255,255,255,.032) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,.032) 1px, transparent 1px);
  background-size: 72px 72px;
  mask-image: radial-gradient(ellipse 100% 100% at 50% 0%, black 30%, transparent 100%);
}
.glow-top {
  position: fixed; inset: 0; pointer-events: none; z-index: 0;
  background: radial-gradient(ellipse 55% 30% at 50% 0%, rgba(48,209,88,.07) 0%, transparent 70%);
}

/* ── NAV ── */
.nav {
  position: sticky; top: 0; z-index: 99;
  height: 54px; display: flex; align-items: center; justify-content: space-between;
  padding: 0 32px; gap: 20px;
  background: rgba(0,0,0,.7); backdrop-filter: blur(28px);
  border-bottom: 1px solid rgba(255,255,255,.05); transition: background .3s;
}
.nav.scrolled { background: rgba(0,0,0,.9); }
.logo {
  font-family: 'Prompt', sans-serif; font-weight: 700; font-size: .95rem;
  color: #f5f5f7; display: flex; align-items: center; gap: 7px;
  letter-spacing: .02em; cursor: pointer; flex-shrink: 0;
}
.nav-center {
  display: flex; flex-direction: column; align-items: center; flex: 1;
}
.nav-greeting {
  font-family: 'Prompt', sans-serif; font-size: .82rem; font-weight: 400; color: #a1a1a6;
}
.nav-greeting span { color: #30d158; font-weight: 600; }
.nav-date { font-size: .68rem; color: #6e6e73; font-family: 'Prompt', sans-serif; }
.nav-end { display: flex; align-items: center; gap: 8px; flex-shrink: 0; }
.nb-icon {
  display: flex; align-items: center; gap: 5px;
  font-size: .75rem; color: #a1a1a6; background: none; border: none; cursor: pointer;
  padding: 5px 10px; border-radius: 8px; transition: all .2s;
  font-family: inherit;
}
.nb-icon:hover { background: rgba(255,255,255,.06); color: #f5f5f7; }
.nb-logout {
  font-size: .75rem; font-family: inherit; color: #ff453a; background: rgba(255,69,58,.06);
  border: 1px solid rgba(255,69,58,.15); border-radius: 8px; padding: 5px 14px; cursor: pointer; transition: all .2s;
}
.nb-logout:hover { background: rgba(255,69,58,.14); border-color: rgba(255,69,58,.3); }

/* ── CONTENT ── */
.content {
  position: relative; z-index: 1;
  max-width: 1160px; margin: 0 auto;
  padding: 32px 28px 80px;
  display: flex; flex-direction: column; gap: 20px;
}

/* ── CARD BASE ── */
.card {
  background: rgba(255,255,255,.03);
  border: 1px solid rgba(255,255,255,.08);
  border-radius: 22px; padding: 28px;
  backdrop-filter: blur(24px);
  box-shadow: 0 20px 60px rgba(0,0,0,.45);
}
.card-label {
  display: flex; align-items: center; gap: 7px;
  font-family: 'Prompt', sans-serif; font-size: .7rem; font-weight: 600;
  color: #6e6e73; letter-spacing: .1em; text-transform: uppercase;
  margin-bottom: 20px;
}
.label-dot { width: 6px; height: 6px; border-radius: 50%; flex-shrink: 0; }

/* ── ROW HERO ── */
.row-hero {
  display: grid;
  grid-template-columns: 280px 1fr 1fr;
  gap: 20px;
}

/* ── KCAL CARD ── */
.card-kcal { display: flex; flex-direction: column; align-items: center; }
.ring-wrap { position: relative; width: 180px; height: 180px; margin-bottom: 16px; }
.ring-svg { width: 100%; height: 100%; transform: rotate(-90deg); }
.ring-arc { transition: stroke-dasharray 1.2s cubic-bezier(.22,1,.36,1); }
.ring-inner {
  position: absolute; inset: 0;
  display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 2px;
}
.ring-val {
  font-family: 'Prompt', sans-serif; font-size: 2.1rem; font-weight: 700;
  color: #30d158; line-height: 1; letter-spacing: -.02em;
}
.ring-val.danger { color: #ff453a; }
.ring-unit { font-size: .68rem; color: #6e6e73; font-weight: 300; }
.ring-pct { font-family: 'Prompt', sans-serif; font-size: .82rem; font-weight: 600; color: #a1a1a6; margin-top: 2px; }
.ring-pct.danger { color: #ff453a; }
.kcal-status {
  width: 100%; font-size: .78rem; font-weight: 500; text-align: center;
  padding: 9px 14px; border-radius: 10px; letter-spacing: .01em;
}
.st-ok   { background: rgba(48,209,88,.08);  color: #30d158; border: 1px solid rgba(48,209,88,.15); }
.st-danger { background: rgba(255,69,58,.08); color: #ff453a; border: 1px solid rgba(255,69,58,.15); }

/* ── MACROS CARD ── */
.macro-list { display: flex; flex-direction: column; gap: 20px; }
.mac-top { display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 7px; }
.mac-name { font-size: .82rem; color: #a1a1a6; font-weight: 300; }
.mac-val {
  font-family: 'Prompt', sans-serif; font-size: 1.3rem; font-weight: 700;
  letter-spacing: -.02em; line-height: 1;
}
.mac-val span { font-size: .65rem; color: #6e6e73; font-weight: 300; margin-left: 2px; }
.mac-track {
  height: 5px; background: rgba(255,255,255,.07); border-radius: 3px;
  overflow: hidden; margin-bottom: 5px;
}
.mac-fill { height: 100%; border-radius: 3px; transition: width 1.2s cubic-bezier(.22,1,.36,1); }
.mac-bot { display: flex; justify-content: space-between; }
.mac-pct { font-size: .62rem; color: #6e6e73; }
.mac-target { font-size: .62rem; color: #4a4a4f; }

/* ── BODY CARD ── */
.body-grid {
  display: grid; grid-template-columns: repeat(3,1fr);
  gap: 1px; background: rgba(255,255,255,.07);
  border-radius: 14px; overflow: hidden;
}
.bg-item {
  background: rgba(255,255,255,.02);
  padding: 16px 14px; display: flex; flex-direction: column; gap: 3px;
  transition: background .2s;
}
.bg-item:hover { background: rgba(255,255,255,.05); }
.bg-val {
  font-family: 'Prompt', sans-serif; font-size: 1.25rem; font-weight: 700;
  color: #f5f5f7; letter-spacing: -.02em; line-height: 1;
}
.bg-label { font-size: .6rem; color: #6e6e73; text-transform: uppercase; letter-spacing: .07em; margin-top: 2px; }
.bg-sub { font-size: .62rem; color: #4a4a4f; }

/* ── LOGS CARD ── */
.card-logs { padding: 0; overflow: hidden; }
.logs-header {
  display: flex; justify-content: space-between; align-items: center;
  padding: 24px 28px 16px; border-bottom: 1px solid rgba(255,255,255,.06);
}
.logs-count { font-size: .72rem; color: #6e6e73; margin-top: 3px; }
.btn-add {
  display: flex; align-items: center; gap: 6px;
  font-family: 'Prompt', sans-serif; font-size: .82rem; font-weight: 600;
  background: rgba(48,209,88,.1); color: #30d158;
  border: 1px solid rgba(48,209,88,.2); border-radius: 980px;
  padding: 8px 18px; cursor: pointer; transition: all .22s;
}
.btn-add:hover { background: rgba(48,209,88,.18); border-color: rgba(48,209,88,.4); transform: scale(.98); }

/* empty state */
.empty-state {
  padding: 60px 24px; text-align: center;
  display: flex; flex-direction: column; align-items: center; gap: 8px;
}
.es-icon { font-size: 2.8rem; margin-bottom: 6px; }
.es-title { font-family: 'Prompt', sans-serif; font-size: 1.1rem; font-weight: 600; color: #f5f5f7; }
.es-sub { font-size: .82rem; color: #6e6e73; font-weight: 300; }

/* log list */
.log-list { padding: 16px 28px 24px; display: flex; flex-direction: column; gap: 24px; }
.meal-group { display: flex; flex-direction: column; gap: 8px; }
.mg-header {
  display: flex; justify-content: space-between; align-items: center;
  padding: 0 4px; margin-bottom: 4px;
}
.mg-name { font-family: 'Prompt', sans-serif; font-size: .72rem; font-weight: 600; color: #6e6e73; letter-spacing: .08em; text-transform: uppercase; }
.mg-kcal { font-family: 'Prompt', sans-serif; font-size: .72rem; color: #4a4a4f; }

.log-item {
  display: flex; justify-content: space-between; align-items: center; gap: 12px;
  background: rgba(255,255,255,.03); border: 1px solid rgba(255,255,255,.06);
  border-radius: 14px; padding: 14px 18px; transition: all .2s;
}
.log-item:hover { background: rgba(255,255,255,.06); border-color: rgba(255,255,255,.1); }
.li-left { flex: 1; min-width: 0; }
.li-name {
  font-family: 'Prompt', sans-serif; font-size: .92rem; font-weight: 600;
  color: #f5f5f7; margin-bottom: 5px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}
.li-macros { display: flex; gap: 10px; font-size: .68rem; font-weight: 500; }
.li-right { display: flex; align-items: center; gap: 12px; flex-shrink: 0; }
.li-kcal {
  font-family: 'Prompt', sans-serif; font-size: 1.05rem; font-weight: 700;
  color: #30d158; line-height: 1; white-space: nowrap;
}
.li-kcal span { font-size: .6rem; color: #6e6e73; font-weight: 300; margin-left: 2px; }
.btn-del {
  width: 30px; height: 30px; border-radius: 8px;
  background: rgba(255,69,58,.06); border: 1px solid rgba(255,69,58,.1);
  color: #ff453a; cursor: pointer; display: flex; align-items: center; justify-content: center;
  transition: all .2s; flex-shrink: 0;
}
.btn-del:hover { background: rgba(255,69,58,.16); border-color: rgba(255,69,58,.3); }

/* ── LOADING ── */
.loading-overlay {
  position: fixed; inset: 0; z-index: 200;
  background: #000; display: flex; align-items: center; justify-content: center;
}
.loading-dots { display: flex; gap: 6px; align-items: center; }
.loading-dots span {
  width: 8px; height: 8px; border-radius: 50%; background: #30d158;
  animation: dot 1.2s ease-in-out infinite;
}
.loading-dots span:nth-child(2) { animation-delay: .2s; }
.loading-dots span:nth-child(3) { animation-delay: .4s; }
@keyframes dot { 0%,80%,100%{transform:scale(.5);opacity:.3} 40%{transform:scale(1);opacity:1} }

.fade-enter-active, .fade-leave-active { transition: opacity .4s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

/* ── RESPONSIVE ── */
@media (max-width: 960px) {
  .row-hero { grid-template-columns: 1fr 1fr; }
  .card-kcal { grid-column: 1 / -1; flex-direction: row; flex-wrap: wrap; gap: 16px; }
  .ring-wrap { flex-shrink: 0; }
}
@media (max-width: 640px) {
  .nav { padding: 0 16px; }
  .nb-icon span { display: none; }
  .content { padding: 16px 14px 60px; }
  .row-hero { grid-template-columns: 1fr; }
  .card-kcal { flex-direction: column; align-items: center; }
  .body-grid { grid-template-columns: repeat(2,1fr); }
  .logs-header { padding: 18px 18px 12px; }
  .log-list { padding: 12px 14px 18px; }
  .li-macros { flex-wrap: wrap; }
}
</style>