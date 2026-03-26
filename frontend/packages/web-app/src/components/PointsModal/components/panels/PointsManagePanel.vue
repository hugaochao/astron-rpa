<script setup lang="ts">
import { computed, ref } from 'vue'

const props = withDefaults(
  defineProps<{
    workspaceName?: string
    currentBalance?: number
    balanceCap?: number
    spaceTotal?: number
    quotaPercent?: number
    monthConsumed?: number
  }>(),
  {
    workspaceName: '星辰科技工作空间',
    currentBalance: 824,
    balanceCap: 1250,
    spaceTotal: 5000,
    quotaPercent: 25,
    monthConsumed: 3420,
  },
)

/** 积分 : 现金 = 100 : 1（元） */
const POINTS_PER_YUAN = 100
const CUSTOM_POINTS_MIN = 100
const CUSTOM_POINTS_MAX = 99_999_999

const packageOptions = [
  { id: '500', points: 500 },
  { id: '2000', points: 2000 },
  { id: '5000', points: 5000 },
  { id: '10000', points: 10000 },
] as const

const selectedPackageId = ref<string>('2000')
const customPoints = ref('')

const selectedPackage = computed(
  () => packageOptions.find(p => p.id === selectedPackageId.value) ?? packageOptions[1],
)

/** 自定义框有内容时视为走自定义档位，套餐仅保留高亮逻辑上的「未选中」 */
const isCustomMode = computed(() => customPoints.value.length > 0)

/** 仅数字；去掉前导 0；限制上限位数 */
function onCustomPointsInput(e: Event) {
  const el = e.target as HTMLInputElement
  let v = el.value.replace(/\D/g, '')
  v = v.replace(/^0+(?=\d)/, '')
  if (v.length > String(CUSTOM_POINTS_MAX).length)
    v = v.slice(0, String(CUSTOM_POINTS_MAX).length)
  if (v && Number(v) > CUSTOM_POINTS_MAX)
    v = String(CUSTOM_POINTS_MAX)
  customPoints.value = v
  if (el.value !== v)
    el.value = v
}

const parsedCustomPoints = computed(() => {
  const s = customPoints.value
  if (!s)
    return null
  const n = Number(s)
  if (!Number.isFinite(n) || n < 1)
    return null
  return Math.min(n, CUSTOM_POINTS_MAX)
})

/** 实际充值积分：有合法自定义用自定义，否则用当前选中套餐 */
const effectiveRechargePoints = computed(() =>
  parsedCustomPoints.value ?? selectedPackage.value.points,
)

const formattedRechargeCash = computed(() =>
  formatCashFromPoints(effectiveRechargePoints.value),
)

/** 与文案「100-99,999,999」一致，用于按钮可用态 */
const canRecharge = computed(() => {
  const p = effectiveRechargePoints.value
  return p >= CUSTOM_POINTS_MIN && p <= CUSTOM_POINTS_MAX
})

function formatCashFromPoints(points: number) {
  const yuan = points / POINTS_PER_YUAN
  return `¥${yuan.toLocaleString('zh-CN', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
}

/**
 * 进度条占比：当前余额 / 可用总额度（balanceCap，即配额折算后的额度上限），与右上角「配额 xx%」展示解耦。
 */
const balanceUsagePercent = computed(() => {
  const cap = props.balanceCap
  if (!cap || cap <= 0)
    return 0
  return Math.min(100, Math.max(0, (props.currentBalance / cap) * 100))
})

/** 进度条颜色：0–20% 红、20–40% 橙、40–100% 品牌紫（按余额占用比例分段） */
const balanceProgressBarColor = computed(() => {
  const p = balanceUsagePercent.value
  if (p < 20)
    return '#F14439'
  if (p < 40)
    return '#F79007'
  return '#726FFF'
})

function selectPackage(id: string) {
  selectedPackageId.value = id
  customPoints.value = ''
}
</script>

<template>
  <div class="flex min-h-0 flex-1 flex-col gap-6 self-stretch overflow-y-auto">
    <div class="text-[15px] leading-[25.5px] !text-[rgba(0,0,0,0.65)] dark:!text-[rgba(255,255,255,0.65)]">
      <span>充值后可使用星辰RPA平台提供的AI智能、OCR、验证码等扩展服务。</span>
      <button
        type="button"
        class="ml-1 border-0 bg-transparent p-0 text-[15px] font-normal leading-[25.5px] text-[#726FFF] hover:opacity-80"
      >
        查看计费项规则
      </button>
    </div>

    <div class="flex flex-col gap-6 self-stretch">
      <div class="grid w-full grid-cols-2 gap-4 self-stretch">
        <!-- 当前余额 -->
        <div
          class="flex min-h-[144px] min-w-0 flex-col gap-[14px] rounded-2xl border border-solid bg-white p-6 !border-[rgba(0,0,0,0.10)] dark:bg-[#1a1a1a] dark:!border-[rgba(255,255,255,0.14)]"
        >
          <div class="relative h-5 w-full shrink-0">
            <span
              class="absolute left-0 top-0 text-sm font-normal leading-[22px] !text-[rgba(0,0,0,0.65)] dark:!text-[rgba(255,255,255,0.65)]"
            >
              当前余额
            </span>
            <div
              class="absolute right-0 top-[-1px] inline-flex items-center justify-start gap-1"
            >
              <span
                class="text-xs font-normal leading-[22px] !text-[rgba(0,0,0,0.65)] dark:!text-[rgba(255,255,255,0.65)]"
              >
                空间总额 {{ spaceTotal.toLocaleString() }}
              </span>
              <div
                class="flex h-[22px] items-center justify-center gap-2.5 rounded-md bg-[rgba(215,215,255,0.40)] px-2.5 py-0.5 dark:bg-[rgba(215,215,255,0.15)]"
              >
                <span class="text-xs font-semibold leading-[18px] text-[#726FFF]">
                  配额{{ quotaPercent }}%
                </span>
              </div>
            </div>
          </div>
          <div class="inline-flex w-full items-end justify-start gap-2.5 self-stretch">
            <span
              class="font-sans text-[44px] font-bold leading-10 !text-[rgba(0,0,0,0.85)] dark:!text-[rgba(255,255,255,0.85)]"
            >
              {{ currentBalance.toLocaleString() }}
            </span>
            <span
              class="text-sm font-normal leading-[22px] !text-[rgba(0,0,0,0.45)] dark:!text-[rgba(255,255,255,0.45)]"
            >
              /{{ balanceCap.toLocaleString() }}积分
            </span>
          </div>
          <div class="h-2 w-full overflow-hidden rounded-full bg-[#ECEDF4] dark:bg-white/[0.08]">
            <div
              class="h-full min-w-0 rounded-full transition-[width,background-color] duration-300"
              :style="{ width: `${balanceUsagePercent}%`, backgroundColor: balanceProgressBarColor }"
            />
          </div>
        </div>

        <!-- 本月消耗 -->
        <div
          class="flex min-h-[144px] min-w-0 flex-col gap-[14px] rounded-2xl border border-solid bg-white p-6 !border-[rgba(0,0,0,0.10)] dark:bg-[#1a1a1a] dark:!border-[rgba(255,255,255,0.14)]"
        >
          <div class="relative h-5 w-full shrink-0">
            <span
              class="absolute left-0 top-0 text-sm font-normal leading-[22px] !text-[rgba(0,0,0,0.65)] dark:!text-[rgba(255,255,255,0.65)]"
            >
              本月消耗
            </span>
          </div>
          <div class="inline-flex w-full flex-1 items-end justify-start gap-2.5">
            <span
              class="font-sans text-[44px] font-bold leading-10 !text-[rgba(0,0,0,0.85)] dark:!text-[rgba(255,255,255,0.85)]"
            >
              {{ monthConsumed.toLocaleString() }}
            </span>
            <span
              class="text-sm font-normal leading-[22px] !text-[rgba(0,0,0,0.45)] dark:!text-[rgba(255,255,255,0.45)]"
            >
              积分
            </span>
          </div>
        </div>
      </div>

      <!-- 充值积分 -->
      <div class="flex flex-col gap-4 self-stretch">
        <div class="text-xl font-semibold leading-8 text-[#1A1512] dark:text-white">
          充值积分
        </div>

        <div
          class="inline-flex items-center justify-start gap-2.5 self-stretch rounded-xl bg-[rgba(215,215,255,0.40)] px-3 py-2.5 dark:bg-[rgba(215,215,255,0.12)]"
        >
          <p class="text-[13px] leading-[20.8px] text-[#726FFF]">
            <span class="font-normal">注：您正在为 </span>
            <span class="font-semibold">{{ workspaceName }}</span>
            <span class="font-normal"> 充值，充值后积分将由空间内成员共享使用。</span>
          </p>
        </div>

        <div class="flex w-full gap-[16px] self-stretch">
          <button
            v-for="pkg in packageOptions"
            :key="pkg.id"
            type="button"
            class="flex h-[120px] min-h-[120px] min-w-0 flex-1 flex-col items-center justify-center gap-1 rounded-2xl border border-solid bg-white transition-colors dark:bg-[#1a1a1a]"
            :class="
              !isCustomMode && selectedPackageId === pkg.id
                ? '!border-[#726FFF] bg-[rgba(215,215,255,0.40)] dark:bg-[rgba(215,215,255,0.12)]'
                : '!border-[rgba(0,0,0,0.10)] dark:!border-[rgba(255,255,255,0.14)]'
            "
            @click="selectPackage(pkg.id)"
          >
            <span class="text-center font-sans text-[32px] font-semibold text-[#1A1512] dark:text-white">
              {{ pkg.points.toLocaleString() }}
            </span>
            <span
              class="text-center font-sans text-sm font-normal leading-[22px] !text-[rgba(0,0,0,0.65)] dark:!text-[rgba(255,255,255,0.65)]"
            >
              {{ formatCashFromPoints(pkg.points) }}
            </span>
          </button>
        </div>

        <div
          class="inline-flex h-10 min-w-0 items-center gap-2 self-stretch rounded-xl border border-solid bg-white px-3 !border-[rgba(0,0,0,0.10)] dark:bg-[#1a1a1a] dark:!border-[rgba(255,255,255,0.14)]"
        >
          <input
            :value="customPoints"
            type="text"
            inputmode="numeric"
            pattern="[0-9]*"
            autocomplete="off"
            placeholder="请输入自定义积分（100-99,999,999）"
            class="min-w-0 flex-1 border-0 bg-white p-0 text-[15px] font-normal !text-[rgba(0,0,0,0.85)] outline-none ring-0 placeholder:!text-[rgba(0,0,0,0.25)] focus:ring-0 dark:bg-[#1a1a1a] dark:!text-[rgba(255,255,255,0.85)] dark:placeholder:!text-[rgba(255,255,255,0.25)]"
            @input="onCustomPointsInput"
          >
          <span
            class="shrink-0 text-sm font-normal leading-[22.4px] !text-[rgba(0,0,0,0.25)] dark:!text-[rgba(255,255,255,0.25)]"
          >
            积分
          </span>
        </div>

        <div
          class="inline-flex min-h-[88px] items-center justify-between gap-4 self-stretch rounded-2xl border border-solid bg-white px-6 py-4 !border-[rgba(0,0,0,0.10)] dark:bg-[#1a1a1a] dark:!border-[rgba(255,255,255,0.14)]"
        >
          <div class="inline-flex min-w-0 flex-1 flex-col items-start justify-start gap-1">
            <div>
              <span
                class="font-sans text-2xl font-semibold !text-[rgba(0,0,0,0.85)] dark:!text-[rgba(255,255,255,0.85)]"
              >
                {{ formattedRechargeCash }}
              </span>
            </div>
            <div class="text-center">
              <span
                class="text-xs font-normal !text-[rgba(0,0,0,0.65)] dark:!text-[rgba(255,255,255,0.65)]"
              >
                充值即表示您同意《
              </span>
              <button
                type="button"
                class="border-0 bg-transparent p-0 text-xs font-normal text-[#726FFF] underline"
              >
                星辰RPA积分充值购买协议
              </button>
              <span
                class="text-xs font-normal !text-[rgba(0,0,0,0.65)] dark:!text-[rgba(255,255,255,0.65)]"
              >
                》
              </span>
            </div>
          </div>
          <button
            type="button"
            class="flex h-[46px] w-[140px] shrink-0 items-center justify-center rounded-[10px] border-0 bg-[#726FFF] px-8 hover:opacity-90 disabled:cursor-not-allowed disabled:opacity-40"
            :disabled="!canRecharge"
          >
            <span class="text-center text-[15px] font-semibold text-white">立即充值</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
