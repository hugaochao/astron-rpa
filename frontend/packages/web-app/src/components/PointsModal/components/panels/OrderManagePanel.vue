<script setup lang="ts">
import { CalendarOutlined } from '@ant-design/icons-vue'
import { useResizeObserver } from '@vueuse/core'
import type { Dayjs } from 'dayjs'
import dayjs from 'dayjs'
import type { ColumnsType } from 'ant-design-vue/es/table'
import { Table } from 'ant-design-vue'
import { computed, h, ref } from 'vue'

import InvoiceApplyModal from '../InvoiceApplyModal.vue'

type OrderActionType = 'apply' | 'invoicing' | 'applied' | 'applied_proxy'

interface OrderRow {
  key: string
  orderId: string
  amountYuan: number
  points: number
  purchasedAt: string
  action: OrderActionType
}

const MOCK_ORDERS: OrderRow[] = [
  {
    key: '1',
    orderId: 'ORD20260308001',
    amountYuan: 100,
    points: 10_000,
    purchasedAt: '2026-03-08 09:15',
    action: 'apply',
  },
  {
    key: '2',
    orderId: 'ORD20260308001',
    amountYuan: 100,
    points: 10_000,
    purchasedAt: '2026-03-08 09:15',
    action: 'invoicing',
  },
  {
    key: '3',
    orderId: 'ORD20260308001',
    amountYuan: 100,
    points: 10_000,
    purchasedAt: '2026-03-08 09:15',
    action: 'applied_proxy',
  },
  {
    key: '4',
    orderId: 'ORD20260308001',
    amountYuan: 100,
    points: 10_000,
    purchasedAt: '2026-03-08 09:15',
    action: 'applied',
  },
  {
    key: '5',
    orderId: 'ORD20260308001',
    amountYuan: 100,
    points: 10_000,
    purchasedAt: '2026-03-08 09:15',
    action: 'apply',
  },
]

const dateRange = ref<[Dayjs, Dayjs] | null>(null)

const ORDER_PICKER_POPUP_STYLE = { zIndex: 1200 } as const

function orderPickerGetPopupContainer(node: HTMLElement): HTMLElement {
  const wrap = node.closest('.ant-modal-wrap')
  return wrap instanceof HTMLElement ? wrap : document.body
}

const filteredOrders = computed(() => {
  let rows = MOCK_ORDERS
  if (dateRange.value) {
    const [start, end] = dateRange.value
    const startMs = start.startOf('day').valueOf()
    const endMs = end.endOf('day').valueOf()
    rows = rows.filter((r) => {
      const t = dayjs(r.purchasedAt, 'YYYY-MM-DD HH:mm').valueOf()
      return t >= startMs && t <= endMs
    })
  }
  return rows
})

const ORDER_TABLE_ROW_PX = 48
const ORDER_TABLE_HEAD_PX = 46

const orderTableWrapRef = ref<HTMLElement | null>(null)
const orderTableBodyMaxPx = ref(280)

useResizeObserver(orderTableWrapRef, (entries) => {
  const h = entries[0]?.contentRect.height ?? 0
  const body = Math.floor(h - ORDER_TABLE_HEAD_PX)
  if (body < 80)
    return
  orderTableBodyMaxPx.value = body
})

const orderTableScroll = computed(() => {
  const n = filteredOrders.value.length
  if (n === 0)
    return undefined
  const maxY = orderTableBodyMaxPx.value
  if (n * ORDER_TABLE_ROW_PX <= maxY)
    return undefined
  return { y: maxY } as const
})

function formatAmount(yuan: number) {
  return `¥${yuan.toLocaleString('zh-CN', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
}

const invoiceModalOpen = ref(false)

function openInvoiceModal() {
  invoiceModalOpen.value = true
}

function renderActionCell(record: OrderRow) {
  const { action } = record
  if (action === 'apply') {
    return h(
      'button',
      {
        type: 'button',
        class:
          'border-0 bg-transparent p-0 text-sm font-semibold leading-[22.4px] text-[#726FFF] hover:opacity-80',
        onClick: () => openInvoiceModal(),
      },
      '申请开票',
    )
  }
  if (action === 'invoicing') {
    return h(
      'span',
      { class: 'text-sm font-normal leading-[22.4px] text-[rgba(0,0,0,0.65)]' },
      '开票中',
    )
  }
  if (action === 'applied_proxy') {
    return h('span', { class: 'inline-flex items-center gap-2' }, [
      h(
        'span',
        { class: 'text-sm font-normal leading-[22.4px] text-[rgba(0,0,0,0.25)]' },
        '已申请',
      ),
      h(
        'span',
        {
          class:
            'inline-flex h-[22px] items-center rounded-md bg-[rgba(215,215,255,0.40)] px-2.5 py-0.5 text-xs font-semibold leading-[18px] text-[#726FFF]',
        },
        '代开',
      ),
    ])
  }
  return h(
    'span',
    { class: 'text-sm font-normal leading-[22.4px] text-[rgba(0,0,0,0.25)]' },
    '已申请',
  )
}

const columns = computed<ColumnsType<OrderRow>>(() => [
  {
    title: '订单编号',
    dataIndex: 'orderId',
    key: 'orderId',
    ellipsis: true,
  },
  {
    title: '交易金额',
    dataIndex: 'amountYuan',
    key: 'amount',
    customRender: ({ record }) =>
      h(
        'span',
        { class: 'text-[13px] font-normal leading-[20.8px] text-[rgba(0,0,0,0.65)]' },
        formatAmount(record.amountYuan),
      ),
  },
  {
    title: '到账积分',
    dataIndex: 'points',
    key: 'points',
    customRender: ({ text }) =>
      h(
        'span',
        { class: 'text-sm font-normal leading-[22.4px] text-[rgba(0,0,0,0.65)]' },
        Number(text).toLocaleString(),
      ),
  },
  {
    title: '购买时间',
    dataIndex: 'purchasedAt',
    key: 'purchasedAt',
    ellipsis: true,
  },
  {
    title: '操作',
    key: 'action',
    width: 140,
    customRender: ({ record }) => renderActionCell(record),
  },
])
</script>

<template>
  <div class="flex min-h-0 flex-1 flex-col gap-4 self-stretch overflow-hidden">
    <!-- 说明 -->
    <div
      class="shrink-0 rounded-xl bg-[rgba(215,215,255,0.40)] px-3 py-3 dark:bg-[rgba(215,215,255,0.12)]"
    >
      <p class="m-0 text-[13px] leading-[20.8px] text-[#726FFF]">
        注：您在该空间的充值记录，空间管理员亦可查看并进行统一财务管理。
      </p>
    </div>

    <!-- 充值订单 -->
    <div
      class="flex min-h-0 min-w-0 flex-1 flex-col gap-4 self-stretch overflow-hidden rounded-2xl border border-solid border-[rgba(0,0,0,0.10)] bg-white px-6 pb-6 pt-6 dark:border-[rgba(255,255,255,0.14)] dark:bg-[#1a1a1a]"
    >
      <div class="flex shrink-0 items-start justify-between gap-4 self-stretch">
        <span
          class="text-base font-semibold leading-[25.6px] text-[rgba(0,0,0,0.85)] dark:text-[rgba(255,255,255,0.85)]"
        >
          充值订单
        </span>
        <a-range-picker
          v-model:value="dateRange"
          :bordered="false"
          class="order-range-picker h-8 w-[240px] shrink-0 rounded-md"
          :placeholder="['开始日期', '结束日期']"
          :get-popup-container="orderPickerGetPopupContainer"
          :popup-style="ORDER_PICKER_POPUP_STYLE"
        >
          <template #suffixIcon>
            <CalendarOutlined class="text-[rgba(0,0,0,0.25)]" />
          </template>
        </a-range-picker>
      </div>

      <div
        ref="orderTableWrapRef"
        class="order-table-wrap min-h-0 min-w-0 flex-1 overflow-x-auto overflow-y-hidden rounded-xl"
      >
        <Table
          row-key="key"
          :columns="columns"
          :data-source="filteredOrders"
          :pagination="false"
          :scroll="orderTableScroll"
          size="small"
          class="order-manage-table"
        />
      </div>
    </div>

    <InvoiceApplyModal v-model:open="invoiceModalOpen" />
  </div>
</template>

<style scoped lang="scss">
.order-range-picker {
  background: #f3f3f7 !important;
  border: 1px solid #f0f0f0 !important;
  border-radius: 6px !important;

  :deep(.ant-picker) {
    background: transparent !important;
    border: none !important;
    box-shadow: none !important;
    padding-inline: 11px !important;
  }

  :deep(.ant-picker-input > input) {
    font-size: 14px !important;
    color: rgba(0, 0, 0, 0.25) !important;
  }

  :deep(.ant-picker-input input::placeholder) {
    color: rgba(0, 0, 0, 0.25) !important;
  }
}

.order-table-wrap {
  display: flex;
  flex-direction: column;
}

.order-manage-table {
  flex: 1;
  min-height: 0;

  &:deep(.ant-spin-nested-loading),
  &:deep(.ant-spin-container),
  &:deep(.ant-table-wrapper) {
    height: 100%;
  }

  &:deep(.ant-table) {
    background: transparent;
  }

  &:deep(.ant-table-container) {
    border-radius: 12px;
    overflow: hidden;
    border: none;
  }

  &:deep(.ant-table.ant-table-small .ant-table-thead > tr > th) {
    background: #f3f3f7;
    color: rgba(0, 0, 0, 0.65);
    font-size: 12px;
    font-weight: 600;
    line-height: 19.2px;
    letter-spacing: 0.6px;
    text-transform: uppercase;
    border-bottom: none;
    padding: 11px 16px;
  }

  &:deep(.ant-table.ant-table-small .ant-table-thead > tr > th:first-child) {
    padding-left: 20px;
    padding-right: 20px;
  }

  &:deep(.ant-table.ant-table-small .ant-table-tbody > tr > td) {
    padding: 16px;
    font-size: 14px;
    line-height: 22.4px;
    color: rgba(0, 0, 0, 0.65);
    border-bottom: 1px solid #f9f4ef;
  }

  &:deep(.ant-table.ant-table-small .ant-table-tbody > tr:last-child > td) {
    border-bottom: none;
  }
}
</style>
