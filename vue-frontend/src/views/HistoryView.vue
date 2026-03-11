<template>
  <div class="page">
    <div class="grid-bg"></div>
    <div class="glow-top"></div>

    <!-- NAV -->
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
          <div>
            <div class="eyebrow"><span class="ew-dot"></span>ประวัติการกิน</div>
            <h1 class="page-title">บันทึก<em>โภชนาการ</em></h1>
            <p class="page-sub">เลือกวันที่เพื่อดูรายละเอียดการรับประทานอาหาร</p>
          </div>

          <!-- Date picker -->
          <div class="date-wrap">
            <div class="date-label">
              <svg width="13" height="13" viewBox="0 0 24 24" fill="none">
                <rect x="3" y="4" width="18" height="18" rx="2" stroke="currentColor" stroke-width="2"/>
                <path d="M16 2v4M8 2v4M3 10h18" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
              เลือกวันที่
            </div>
            <div class="date-iw">
              <input type="date" v-model="selectedDate" @change="fetchHistoryData"
                class="date-inp" />
            </div>
          </div>
        </div>

        <!-- loading -->
        <div class="loading-wrap" v-if="loading">
          <div class="ldots"><span></span><span></span><span></span></div>
        </div>

        <template v-else>

          <!-- SUMMARY ROW -->
          <div class="summary-row">
            <!-- kcal ring card -->
            <div class="card card-kcal">
              <div class="ck-label">
                <span class="ck-dot" :style="`background:${kcalColor}`"></span>
                แคลอรี่รวม
              </div>
              <div class="ring-wrap">
                <svg class="ring-svg" viewBox="0 0 120 120">
                  <circle cx="60" cy="60" r="48" fill="none"
                    stroke="rgba(255,255,255,.06)" stroke-width="10"/>
                  <circle cx="60" cy="60" r="48" fill="none"
                    :stroke="kcalColor" stroke-width="10"
                    stroke-linecap="round"
                    stroke-dasharray="301.59"
                    :stroke-dashoffset="301.59 - (Math.min(kcalPct,100)/100)*301.59"
                    transform="rotate(-90 60 60)"
                    style="transition:stroke-dashoffset .8s cubic-bezier(.22,1,.36,1)"/>
                </svg>
                <div class="ring-inner">
                  <div class="ring-val" :style="`color:${kcalColor}`">{{ totalCalories }}</div>
                  <div class="ring-unit">kcal</div>
                </div>
              </div>
              <div class="ck-tdee">เป้าหมาย <span>{{ userTDEE }} kcal</span></div>
              <div class="ck-status" :class="totalCalories > userTDEE ? 'over' : 'ok'">
                {{ totalCalories > userTDEE ? 'เกินเป้าหมาย' : 'อยู่ในเกณฑ์' }}
              </div>
            </div>

            <!-- macros card -->
            <div class="card card-macros">
              <div class="cm-title">สารอาหาร</div>
              <div class="macro-list">
                <div class="macro-item" v-for="m in macros" :key="m.label">
                  <div class="mi-top">
                    <span class="mi-label">{{ m.label }}</span>
                    <span class="mi-val" :style="`color:${m.color}`">{{ m.val }}g</span>
                  </div>
                  <div class="mi-bar-bg">
                    <div class="mi-bar" :style="`width:${Math.min((m.val/m.target)*100,100)}%;background:${m.color}`"></div>
                  </div>
                  <div class="mi-meta">
                    <span>{{ Math.round((m.val/m.target)*100) }}%</span>
                    <span>เป้า {{ m.target }}g</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- date info card -->
            <div class="card card-date">
              <div class="cd-date-big">{{ dayNum }}</div>
              <div class="cd-month">{{ monthThai }} {{ yearBE }}</div>
              <div class="cd-divider"></div>
              <div class="cd-stats">
                <div class="cds">
                  <div class="cds-val">{{ foodLogs.length }}</div>
                  <div class="cds-lbl">รายการ</div>
                </div>
                <div class="cds">
                  <div class="cds-val">{{ mealCount }}</div>
                  <div class="cds-lbl">มื้อ</div>
                </div>
              </div>
            </div>
          </div>

          <!-- FOOD LOG -->
          <div class="section">
            <div class="sec-head">
              <div class="sh-left">
                <svg width="15" height="15" viewBox="0 0 24 24" fill="none">
                  <path d="M3 6h18M3 12h18M3 18h18" stroke="#30d158" stroke-width="2" stroke-linecap="round"/>
                </svg>
                รายการอาหาร
              </div>
              <span class="sh-count">{{ foodLogs.length }} รายการ</span>
            </div>

            <!-- empty state -->
            <div class="empty-state" v-if="foodLogs.length === 0">
              <svg width="40" height="40" viewBox="0 0 24 24" fill="none" opacity=".25">
                <circle cx="12" cy="12" r="10" stroke="white" stroke-width="1.5"/>
                <path d="M8 12h8M12 8v8" stroke="white" stroke-width="1.5" stroke-linecap="round"/>
              </svg>
              <p>ไม่พบข้อมูลการกินของวันนี้</p>
              <span>ลืมบันทึก หรือกำลังอดอาหารอยู่?</span>
            </div>

            <!-- grouped by meal -->
            <template v-else>
              <div class="meal-group" v-for="group in mealGroups" :key="group.key">
                <div class="mg-head">
                  <div class="mg-icon">
                    <svg v-if="group.key==='breakfast'" width="14" height="14" viewBox="0 0 24 24" fill="none">
                      <circle cx="12" cy="12" r="5" stroke="currentColor" stroke-width="2"/>
                      <path d="M12 2v2M12 20v2M4.22 4.22l1.42 1.42M18.36 18.36l1.42 1.42M2 12h2M20 12h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                    <svg v-else-if="group.key==='lunch'" width="14" height="14" viewBox="0 0 24 24" fill="none">
                      <path d="M18 8h1a4 4 0 0 1 0 8h-1M2 8h16v9a4 4 0 0 1-4 4H6a4 4 0 0 1-4-4V8zM6 1v3M10 1v3M14 1v3" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                    <svg v-else-if="group.key==='dinner'" width="14" height="14" viewBox="0 0 24 24" fill="none">
                      <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                    <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none">
                      <path d="M18 2h-3a5 5 0 0 0-5 5v3H7v4h3v8h4v-8h3l1-4h-4V7a1 1 0 0 1 1-1h3z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                  </div>
                  <span class="mg-label">{{ group.label }}</span>
                  <span class="mg-kcal">{{ group.totalKcal }} kcal</span>
                </div>

                <div class="log-item" v-for="item in group.items" :key="item.log_id">
                  <div class="li-left">
                    <div class="li-name">{{ item.food_name }}</div>
                    <div class="li-meta">
                      <span>x{{ item.quantity }}</span>
                      <span class="li-dot"></span>
                      <span class="li-p">P {{ item.total_protein ?? Math.round((item.total_calories||0)*0.25/4) }}g</span>
                      <span class="li-c">C {{ item.total_carbs  ?? Math.round((item.total_calories||0)*0.50/4) }}g</span>
                      <span class="li-f">F {{ item.total_fat    ?? Math.round((item.total_calories||0)*0.25/9) }}g</span>
                    </div>
                  </div>
                  <div class="li-kcal">{{ item.total_calories }} <span>kcal</span></div>
                </div>
              </div>
            </template>
          </div>

        </template>

      </div>
    </div>

    <transition name="toast">
      <div class="toast err" v-if="toast.show">{{ toast.msg }}</div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const loading = ref(false)

const today = new Date().toLocaleDateString('en-CA', { timeZone: 'Asia/Bangkok' })
const selectedDate = ref(today)

const userTDEE     = ref(2000)
const totalCalories= ref(0)
const totalProtein = ref(0)
const totalCarbs   = ref(0)
const totalFat     = ref(0)
const foodLogs     = ref([])

const toast = ref({ show: false, msg: '' })
function showToast(msg) {
  toast.value = { show: true, msg }
  setTimeout(() => toast.value.show = false, 2800)
}

// ── date display ──
const MONTHS_TH = ['ม.ค.','ก.พ.','มี.ค.','เม.ย.','พ.ค.','มิ.ย.','ก.ค.','ส.ค.','ก.ย.','ต.ค.','พ.ย.','ธ.ค.']
const dayNum  = computed(() => selectedDate.value ? selectedDate.value.split('-')[2] : '')
const monthThai = computed(() => {
  if (!selectedDate.value) return ''
  return MONTHS_TH[parseInt(selectedDate.value.split('-')[1]) - 1]
})
const yearBE = computed(() => selectedDate.value ? parseInt(selectedDate.value.split('-')[0]) + 543 : '')

// ── derived ──
const kcalPct  = computed(() => userTDEE.value ? (totalCalories.value / userTDEE.value) * 100 : 0)
const kcalColor = computed(() => totalCalories.value > userTDEE.value ? '#ff453a' : '#30d158')

const macros = computed(() => [
  { label:'โปรตีน', val: totalProtein.value, target: 150, color:'#0a84ff' },
  { label:'คาร์บ',  val: totalCarbs.value,   target: 250, color:'#ff9f0a' },
  { label:'ไขมัน',  val: totalFat.value,     target: 70,  color:'#ff453a' },
])

const MEAL_ORDER = ['breakfast','lunch','dinner','snack']
const MEAL_LABEL = { breakfast:'มื้อเช้า', lunch:'มื้อเที่ยง', dinner:'มื้อเย็น', snack:'ของว่าง' }

const mealGroups = computed(() => {
  const map = {}
  foodLogs.value.forEach(log => {
    const k = log.meal_type || 'snack'
    if (!map[k]) map[k] = []
    map[k].push(log)
  })
  return MEAL_ORDER
    .filter(k => map[k])
    .map(k => ({
      key: k,
      label: MEAL_LABEL[k],
      items: map[k],
      totalKcal: map[k].reduce((s,l) => s + (l.total_calories||0), 0)
    }))
})

const mealCount = computed(() => mealGroups.value.length)

// ── fetch ──
async function fetchHistoryData() {
  const userId = localStorage.getItem('userId')
  if (!userId) { router.push('/login'); return }
  loading.value = true
  try {
    const [userRes, summaryRes] = await Promise.all([
      fetch(`/api/users/${userId}`),
      fetch(`/api/daily-summary/${userId}/${selectedDate.value}`)
    ])
    if (userRes.ok) {
      const u = await userRes.json()
      let bmr = 10 * u.weight + 6.25 * u.height - 5 * u.age
      bmr += (u.gender === 'male' || u.gender === 'ชาย') ? 5 : -161
      const mult = { low:1.2, medium:1.375, high:1.55 }
      userTDEE.value = Math.round(bmr * (mult[u.activity_level] || 1.2))
    }
    if (summaryRes.ok) {
      const s = await summaryRes.json()
      totalCalories.value = s.summary?.total_calories || 0
      totalProtein.value  = s.summary?.total_protein  || 0
      totalCarbs.value    = s.summary?.total_carbs    || 0
      totalFat.value      = s.summary?.total_fat      || 0
      foodLogs.value      = s.logs || []
    }
  } catch (e) {
    console.error(e)
    showToast('โหลดข้อมูลไม่สำเร็จ')
  } finally {
    loading.value = false
  }
}

onMounted(fetchHistoryData)
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;700;900&family=Noto+Sans+Thai:wght@300;400;600&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}

.page {
  font-family:'Noto Sans Thai','Prompt',sans-serif;
  background:#000; color:#f5f5f7; min-height:100vh;
  overflow-x:hidden; -webkit-font-smoothing:antialiased;
}
.grid-bg {
  position:fixed; inset:0; pointer-events:none; z-index:0;
  background-image:
    linear-gradient(rgba(255,255,255,.03) 1px,transparent 1px),
    linear-gradient(90deg,rgba(255,255,255,.03) 1px,transparent 1px);
  background-size:72px 72px;
  mask-image:radial-gradient(ellipse 100% 70% at 50% 0%,black 20%,transparent 100%);
}
.glow-top {
  position:fixed; inset:0; pointer-events:none; z-index:0;
  background:radial-gradient(ellipse 55% 28% at 50% 0%,rgba(48,209,88,.07) 0%,transparent 70%);
}

/* NAV */
.nav {
  position:sticky; top:0; z-index:99;
  height:52px; display:flex; align-items:center; justify-content:space-between;
  padding:0 28px;
  background:rgba(0,0,0,.8); backdrop-filter:blur(28px);
  border-bottom:1px solid rgba(255,255,255,.06);
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
.nb-back:hover{background:rgba(255,255,255,.09);color:#f5f5f7}

/* SCROLL */
.scroll-wrap {
  position:relative; z-index:1;
  display:flex; justify-content:center;
  padding:48px 20px 80px;
  min-height:calc(100vh - 52px);
}
.container{width:100%;max-width:760px;display:flex;flex-direction:column;gap:16px}

/* PAGE HEAD */
.page-head {
  display:flex; align-items:flex-start; justify-content:space-between;
  gap:20px; flex-wrap:wrap; margin-bottom:4px;
}
.eyebrow {
  display:inline-flex; align-items:center; gap:7px;
  font-size:.7rem; font-weight:600; color:#30d158;
  letter-spacing:.12em; text-transform:uppercase; margin-bottom:10px;
}
.ew-dot{width:6px;height:6px;border-radius:50%;background:#30d158;box-shadow:0 0 8px #30d158}
.page-title {
  font-family:'Prompt',sans-serif; font-weight:700;
  font-size:clamp(1.8rem,4vw,2.8rem); line-height:1.05; letter-spacing:-.04em; margin-bottom:8px;
}
.page-title em {
  font-style:normal;
  background:linear-gradient(135deg,#30d158,#34aadc);
  -webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text;
}
.page-sub{font-size:.82rem;font-weight:300;color:#6e6e73;line-height:1.7}

/* DATE PICKER */
.date-wrap{display:flex;flex-direction:column;gap:7px;align-items:flex-end}
.date-label {
  display:flex; align-items:center; gap:6px;
  font-size:.68rem; color:#6e6e73; letter-spacing:.08em; text-transform:uppercase;
}
.date-iw {
  background:rgba(255,255,255,.05); border:1px solid rgba(255,255,255,.09);
  border-radius:12px; padding:0 14px; height:44px;
  display:flex; align-items:center; transition:all .22s;
}
.date-iw:focus-within{border-color:rgba(48,209,88,.42);background:rgba(48,209,88,.04);box-shadow:0 0 0 3px rgba(48,209,88,.07)}
.date-inp {
  background:none; border:none; outline:none;
  font-family:'Noto Sans Thai','Prompt',sans-serif;
  font-size:.9rem; color:#f5f5f7; color-scheme:dark; cursor:pointer;
}

/* LOADING */
.loading-wrap{display:flex;justify-content:center;padding:60px 0}
.ldots{display:flex;gap:6px;align-items:center}
.ldots span{width:7px;height:7px;border-radius:50%;background:#30d158;animation:dot 1.2s ease-in-out infinite}
.ldots span:nth-child(2){animation-delay:.2s}
.ldots span:nth-child(3){animation-delay:.4s}
@keyframes dot{0%,80%,100%{transform:scale(.5);opacity:.3}40%{transform:scale(1);opacity:1}}

/* SUMMARY ROW */
.summary-row{display:grid;grid-template-columns:1fr 2fr 1fr;gap:12px}

.card {
  background:rgba(255,255,255,.025); border:1px solid rgba(255,255,255,.07);
  border-radius:20px; padding:22px;
}

/* kcal card */
.card-kcal{display:flex;flex-direction:column;align-items:center;gap:10px}
.ck-label{
  display:flex;align-items:center;gap:7px;
  font-size:.65rem;font-weight:600;color:#6e6e73;
  letter-spacing:.1em;text-transform:uppercase;
}
.ck-dot{width:5px;height:5px;border-radius:50%;transition:background .3s}
.ring-wrap{position:relative;width:110px;height:110px}
.ring-svg{width:100%;height:100%}
.ring-inner{
  position:absolute;inset:0;display:flex;flex-direction:column;
  align-items:center;justify-content:center;gap:2px;
}
.ring-val{font-family:'Prompt',sans-serif;font-size:1.5rem;font-weight:700;line-height:1}
.ring-unit{font-size:.6rem;color:#6e6e73}
.ck-tdee{font-size:.7rem;color:#6e6e73}
.ck-tdee span{color:#a1a1a6}
.ck-status{
  font-size:.7rem;font-weight:600;padding:5px 12px;border-radius:980px;
}
.ck-status.ok{background:rgba(48,209,88,.1);color:#30d158;border:1px solid rgba(48,209,88,.2)}
.ck-status.over{background:rgba(255,69,58,.1);color:#ff453a;border:1px solid rgba(255,69,58,.2)}

/* macros card */
.card-macros{display:flex;flex-direction:column;gap:16px}
.cm-title{font-family:'Prompt',sans-serif;font-size:.82rem;font-weight:600;color:#f5f5f7}
.macro-list{display:flex;flex-direction:column;gap:12px}
.macro-item{display:flex;flex-direction:column;gap:4px}
.mi-top{display:flex;justify-content:space-between;align-items:baseline}
.mi-label{font-size:.75rem;color:#a1a1a6}
.mi-val{font-family:'Prompt',sans-serif;font-size:.9rem;font-weight:600}
.mi-bar-bg{height:5px;background:rgba(255,255,255,.07);border-radius:10px;overflow:hidden}
.mi-bar{height:100%;border-radius:10px;transition:width .7s cubic-bezier(.22,1,.36,1)}
.mi-meta{display:flex;justify-content:space-between;font-size:.6rem;color:#6e6e73}

/* date card */
.card-date{display:flex;flex-direction:column;align-items:center;justify-content:center;gap:4px;text-align:center}
.cd-date-big{
  font-family:'Prompt',sans-serif;font-size:3.5rem;font-weight:900;
  line-height:1;letter-spacing:-.05em;color:#f5f5f7;
}
.cd-month{font-size:.8rem;color:#6e6e73;font-weight:300}
.cd-divider{width:32px;height:1px;background:rgba(255,255,255,.1);margin:10px 0}
.cd-stats{display:flex;gap:20px}
.cds{display:flex;flex-direction:column;align-items:center;gap:2px}
.cds-val{font-family:'Prompt',sans-serif;font-size:1.4rem;font-weight:700;color:#30d158}
.cds-lbl{font-size:.62rem;color:#6e6e73}

/* SECTION */
.section{background:rgba(255,255,255,.025);border:1px solid rgba(255,255,255,.07);border-radius:20px;padding:22px}
.sec-head{display:flex;align-items:center;justify-content:space-between;margin-bottom:20px}
.sh-left{display:flex;align-items:center;gap:8px;font-family:'Prompt',sans-serif;font-size:.88rem;font-weight:600;color:#f5f5f7}
.sh-count{font-size:.72rem;color:#6e6e73;background:rgba(255,255,255,.06);border:1px solid rgba(255,255,255,.08);border-radius:8px;padding:3px 10px}

/* empty */
.empty-state{
  display:flex;flex-direction:column;align-items:center;gap:10px;
  padding:48px 20px;color:#6e6e73;text-align:center;
}
.empty-state p{font-size:.9rem}
.empty-state span{font-size:.75rem;color:#4a4a4f}

/* meal group */
.meal-group{margin-bottom:20px}
.meal-group:last-child{margin-bottom:0}
.mg-head{
  display:flex;align-items:center;gap:9px;
  margin-bottom:8px;padding-bottom:8px;
  border-bottom:1px solid rgba(255,255,255,.06);
}
.mg-icon{
  width:26px;height:26px;border-radius:8px;
  background:rgba(48,209,88,.1);border:1px solid rgba(48,209,88,.15);
  color:#30d158;display:flex;align-items:center;justify-content:center;flex-shrink:0;
}
.mg-label{font-family:'Prompt',sans-serif;font-size:.82rem;font-weight:600;color:#f5f5f7;flex:1}
.mg-kcal{font-size:.75rem;color:#6e6e73}

/* log item */
.log-item{
  display:flex;align-items:center;justify-content:space-between;
  background:rgba(255,255,255,.02);border:1px solid rgba(255,255,255,.05);
  border-radius:12px;padding:12px 14px;margin-bottom:6px;
  transition:background .2s;
}
.log-item:hover{background:rgba(255,255,255,.05)}
.log-item:last-child{margin-bottom:0}
.li-left{display:flex;flex-direction:column;gap:5px;min-width:0}
.li-name{font-size:.86rem;font-weight:600;color:#f5f5f7;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.li-meta{display:flex;align-items:center;gap:6px;flex-wrap:wrap}
.li-meta span{font-size:.68rem;color:#6e6e73}
.li-dot{width:3px;height:3px;border-radius:50%;background:#4a4a4f}
.li-p{color:#0a84ff !important}
.li-c{color:#ff9f0a !important}
.li-f{color:#ff453a !important}
.li-kcal{
  font-family:'Prompt',sans-serif;font-size:.95rem;font-weight:700;
  color:#30d158;flex-shrink:0;margin-left:12px;white-space:nowrap;
}
.li-kcal span{font-size:.6rem;font-weight:300;color:#6e6e73;margin-left:2px}

/* toast */
.toast{
  position:fixed;bottom:28px;left:50%;transform:translateX(-50%);z-index:999;
  font-family:'Prompt',sans-serif;font-size:.85rem;font-weight:500;
  padding:11px 24px;border-radius:980px;box-shadow:0 8px 32px rgba(0,0,0,.5);white-space:nowrap;
}
.toast.err{background:#ff453a;color:#fff}
.toast-enter-active,.toast-leave-active{transition:all .32s cubic-bezier(.22,1,.36,1)}
.toast-enter-from,.toast-leave-to{opacity:0;transform:translateX(-50%) translateY(12px)}

/* responsive */
@media(max-width:680px){
  .summary-row{grid-template-columns:1fr 1fr}
  .card-date{order:-1;grid-column:1/-1;flex-direction:row;justify-content:flex-start;gap:20px;padding:16px 20px}
  .cd-divider{width:1px;height:40px;margin:0}
  .cd-date-big{font-size:2.5rem}
}
@media(max-width:480px){
  .scroll-wrap{padding:28px 14px 60px}
  .page-head{flex-direction:column}
  .summary-row{grid-template-columns:1fr}
  .card-date{order:0}
}
</style>