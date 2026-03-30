<script setup lang="ts">
import { CalendarOutlined } from '@ant-design/icons-vue'
import type { Dayjs } from 'dayjs'
import type { ColumnsType } from 'ant-design-vue/es/table'
import { Table } from 'ant-design-vue'
import { computed, h, ref, watch } from 'vue'

import type { PaymentOrderRecord } from '@/api/points'
import { getPaymentOrders } from '@/api/points'

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

function mapInvoiceToAction(inv: string): OrderActionType {
  const s = inv.toLowerCase()
  if (s.includes('proxy') || s === 'agency' || s === 'applied_proxy')
    return 'applied_proxy'
  if (s.includes('ing') || s === 'processing' || s === 'invoicing')
    return 'invoicing'
  if (s.includes('applied') || s === 'done' || s === 'success' || s === 'issued')
    return 'applied'
  return 'apply'
}

function mapOrderRecord(r: PaymentOrderRecord, index: number): OrderRow {
  const biz = String(r.bizorderno ?? r.orderNo ?? `order-${index}`)
  const amount = Number(r.amount ?? r.payAmount ?? r.totalAmount ?? 0)
  const pts = Number(r.points ?? r.payPoints ?? 0)
  const purchasedAt = String(r.payTime ?? r.gmtCreate ?? r.createTime ?? '')
  const inv = String(r.invoiceStatus ?? r.invoiceApplyStatus ?? '')
  return {
    key: biz,
    orderId: biz,
    amountYuan: amount,
    points: pts,
    purchasedAt,
    action: mapInvoiceToAction(inv),
  }
}

const dateRange = ref<[Dayjs, Dayjs] | null>(null)
const orders = ref<OrderRow[]>([])
const orderTotal = ref(0)
const orderPageNo = ref(1)
const orderPageSize = ref(10)
const orderLoading = ref(false)

const ORDER_PICKER_POPUP_STYLE = { zIndex: 1200 } as const

function orderPickerGetPopupContainer(node: HTMLElement): HTMLElement {
  const wrap = node.closest('.ant-modal-wrap')
  return wrap instanceof HTMLElement ? wrap : document.body
}

async function fetchOrders() {
  orderLoading.value = true
  try {
    const params: {
      pageNo: number
      pageSize: number
      startDate?: string
      endDate?: string
    } = { pageNo: orderPageNo.value, pageSize: orderPageSize.value }
    if (dateRange.value) {
      params.startDate = dateRange.value[0].format('YYYY-MM-DD')
      params.endDate = dateRange.value[1].format('YYYY-MM-DD')
    }
    const data = await getPaymentOrders(params)
    const records = data?.records ?? []
    orders.value = records.map((r, i) => mapOrderRecord(r, i))
    orderTotal.value = data?.total ?? records.length
  }
  finally {
    orderLoading.value = false
  }
}

watch(dateRange, () => {
  orderPageNo.value = 1
})

watch([dateRange, orderPageNo, orderPageSize], () => {
  fetchOrders()
}, { immediate: true })

function formatAmount(yuan: number) {
  return `¥${yuan.toLocaleString('zh-CN', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
}

const invoiceModalOpen = ref(false)
const invoiceBizorderno = ref('')

function openInvoiceModal(record: OrderRow) {
  invoiceBizorderno.value = record.orderId
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
        onClick: () => openInvoiceModal(record),
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
  <div
    class="flex min-h-0 min-w-0 flex-1 flex-col gap-4 self-stretch overflow-y-auto overscroll-contain"
  >
    <!-- 说明 -->
    <div
      class="shrink-0 rounded-xl bg-[rgba(215,215,255,0.40)] px-3 py-3 dark:bg-[rgba(215,215,255,0.12)]"
    >
      <p class="m-0 text-[13px] leading-[20.8px] text-[#726FFF]">
        注：您在该空间的充值记录，空间管理员亦可查看并进行统一财务管理。
      </p>
    </div>

    <!-- 充值订单：卡片最小高度 412px -->
    <div
      class="order-manage-section flex min-w-0 flex-col gap-4 self-stretch overflow-hidden rounded-2xl border border-solid border-[rgba(0,0,0,0.10)] bg-white px-6 pb-6 pt-6 dark:border-[rgba(255,255,255,0.14)] dark:bg-[#1a1a1a]"
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

      <div class="order-table-wrap w-full min-w-0">
        <a-spin :spinning="orderLoading" class="w-full">
          <Table
            row-key="key"
            :columns="columns"
            :data-source="orders"
            :pagination="false"
            size="small"
            class="order-manage-table"
          />
        </a-spin>
      </div>

      <div v-if="orderTotal > 0" class="flex shrink-0 justify-end pt-1">
        <a-pagination
          v-model:current="orderPageNo"
          v-model:page-size="orderPageSize"
          :total="orderTotal"
          :show-size-changer="true"
          size="small"
        />
      </div>
    </div>

    <InvoiceApplyModal v-model:open="invoiceModalOpen" :bizorderno="invoiceBizorderno" />
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

.order-manage-section {
  box-sizing: border-box;
  min-height: 412px;
}

.order-table-wrap {
  display: flex;
  flex-direction: column;
}

.order-manage-table {
  &:deep(.ant-table) {
    background: transparent;
  }

  &:deep(.ant-table-container) {
    border-radius: 8px;
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
