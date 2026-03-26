<script setup lang="ts">
import { message, Modal } from 'ant-design-vue'
import { reactive, watch } from 'vue'

defineOptions({ name: 'InvoiceApplyModal' })

const props = defineProps<{
  open: boolean
}>()

const emit = defineEmits<{
  'update:open': [boolean]
}>()

const form = reactive({
  invoiceTitle: '',
  taxId: '',
  invoiceType: 'normal',
  email: '',
  totalAmount: null as number | null,
})

const invoiceTypeOptions = [{ value: 'normal', label: '普通发票' }]

const emailRules = [
  { required: true, message: '请输入接受邮箱', trigger: 'blur' },
  { type: 'email' as const, message: '请输入有效邮箱', trigger: 'blur' },
]

function selectGetPopupContainer(node: HTMLElement): HTMLElement {
  const wrap = node.closest('.ant-modal-wrap')
  return wrap instanceof HTMLElement ? wrap : document.body
}

watch(
  () => props.open,
  (v) => {
    if (!v)
      return
    form.invoiceTitle = ''
    form.taxId = ''
    form.invoiceType = 'normal'
    form.email = ''
    form.totalAmount = null
  },
)

function close() {
  emit('update:open', false)
}

function onFinish() {
  message.success('已提交申请（示例）')
  close()
}
</script>

<template>
  <Modal
    :open="open"
    :width="400"
    :z-index="1200"
    :footer="null"
    centered
    wrap-class-name="invoice-apply-modal-wrap"
    class="invoice-apply-modal"
    :closable="true"
    destroy-on-close
    @cancel="close"
  >
    <template #title>
      <span class="text-base font-semibold text-[rgba(0,0,0,0.85)] dark:text-[rgba(255,255,255,0.85)]">
        申请开票
      </span>
    </template>

    <div class="invoice-apply-modal-body px-6 pb-6 pt-2">
      <a-form
        :model="form"
        layout="vertical"
        class="invoice-apply-form"
        @finish="onFinish"
      >
        <div class="flex flex-col gap-4">
          <a-form-item
            name="invoiceTitle"
            class="!mb-0"
            :rules="[{ required: true, message: '请输入发票抬头', trigger: 'blur' }]"
          >
            <template #label>
              <span class="text-xs font-medium leading-[22px] text-[rgba(0,0,0,0.45)]">发票抬头</span>
              <span class="ml-1 text-sm font-semibold leading-[22px] text-[#FF4D4F]">*</span>
            </template>
            <a-input
              v-model:value="form.invoiceTitle"
              placeholder="请输入发票抬头"
              :bordered="false"
              class="invoice-apply-input h-8 rounded-lg text-xs"
            />
          </a-form-item>

          <a-form-item
            name="taxId"
            class="!mb-0"
            :rules="[{ required: true, message: '请输入纳税人识别号', trigger: 'blur' }]"
          >
            <template #label>
              <span class="text-xs font-medium leading-[22px] text-[rgba(0,0,0,0.45)]">纳税人识别号</span>
              <span class="ml-1 text-sm font-semibold leading-[22px] text-[#FF4D4F]">*</span>
            </template>
            <a-input
              v-model:value="form.taxId"
              placeholder="请输入纳税人识别号"
              :bordered="false"
              class="invoice-apply-input h-8 rounded-lg text-xs"
            />
          </a-form-item>

          <a-form-item
            name="invoiceType"
            class="!mb-0"
            :rules="[{ required: true, message: '请选择发票类型', trigger: 'change' }]"
          >
            <template #label>
              <span class="text-xs font-medium leading-[22px] text-[rgba(0,0,0,0.45)]">发票类型</span>
              <span class="ml-1 text-sm font-semibold leading-[22px] text-[#FF4D4F]">*</span>
            </template>
            <a-select
              v-model:value="form.invoiceType"
              :options="invoiceTypeOptions"
              :bordered="false"
              :get-popup-container="selectGetPopupContainer"
              :popup-style="{ zIndex: 1250 }"
              class="invoice-apply-select h-8 rounded-md text-xs"
            />
          </a-form-item>

          <a-form-item name="email" class="!mb-0" :rules="emailRules">
            <template #label>
              <span class="text-xs font-medium leading-[22px] text-[rgba(0,0,0,0.45)]">接受邮箱</span>
              <span class="ml-1 text-sm font-semibold leading-[22px] text-[#FF4D4F]">*</span>
            </template>
            <a-input
              v-model:value="form.email"
              placeholder="请输入接受邮箱"
              :bordered="false"
              class="invoice-apply-input h-8 rounded-lg text-xs"
            />
          </a-form-item>

          <a-form-item
            name="totalAmount"
            class="!mb-0"
            :rules="[
              { required: true, message: '请输入开票总金额', trigger: 'change' },
              { type: 'number', min: 0, message: '金额需大于等于 0', trigger: 'change' },
            ]"
          >
            <template #label>
              <span class="text-xs font-medium leading-[22px] text-[rgba(0,0,0,0.45)]">开票总金额</span>
              <span class="ml-1 text-sm font-semibold leading-[22px] text-[#FF4D4F]">*</span>
            </template>
            <a-input-number
              v-model:value="form.totalAmount"
              :min="0"
              :precision="2"
              :step="0.01"
              :controls="false"
              placeholder="请输入开票总金额"
              :bordered="false"
              class="invoice-apply-input-number h-8 w-full rounded-lg text-xs"
            />
          </a-form-item>
        </div>

        <div class="mt-6 flex justify-end gap-2 pb-1">
          <a-button class="h-8 rounded-md px-4" @click="close">
            取消
          </a-button>
          <a-button
            type="primary"
            html-type="submit"
            class="h-8 rounded-lg border-[#726FFF] bg-[#726FFF] px-4 hover:!bg-[#5f5cff]"
          >
            确认
          </a-button>
        </div>
      </a-form>
    </div>
  </Modal>
</template>

<style scoped lang="scss">
.invoice-apply-form {
  :deep(.ant-form-item-label) {
    padding-bottom: 8px;
  }

  :deep(.ant-form-item-label > label) {
    height: auto;
    line-height: 22px;
  }

  :deep(.ant-form-item-label > label::after) {
    display: none;
  }
}

.invoice-apply-input {
  background: #f3f3f7 !important;

  :deep(.ant-input) {
    background: transparent !important;
    font-size: 12px !important;
    color: rgba(0, 0, 0, 0.85) !important;
  }

  :deep(.ant-input::placeholder) {
    color: rgba(0, 0, 0, 0.25) !important;
  }
}

.invoice-apply-input-number {
  width: 100%;
  background: #f3f3f7 !important;

  :deep(.ant-input-number) {
    width: 100%;
    background: #f3f3f7 !important;
    border: none !important;
    border-radius: 8px !important;
    box-shadow: none !important;
  }

  :deep(.ant-input-number-input) {
    height: 32px;
    padding-top: 4px;
    padding-bottom: 4px;
    font-size: 12px !important;
    color: rgba(0, 0, 0, 0.85) !important;
    background: transparent !important;
  }

  :deep(.ant-input-number-input::placeholder) {
    color: rgba(0, 0, 0, 0.25) !important;
  }
}

.invoice-apply-select {
  background: #f3f3f7 !important;

  :deep(.ant-select-selector) {
    height: 32px !important;
    background: #f3f3f7 !important;
    border: none !important;
    border-radius: 6px !important;
    box-shadow: none !important;
  }

  :deep(.ant-select-selection-item) {
    line-height: 30px !important;
    font-size: 12px !important;
  }
}
</style>

<style lang="scss">
.invoice-apply-modal-wrap.ant-modal-wrap .ant-modal-content {
  padding: 0;
  border-radius: 8px;
  overflow: hidden;
  box-shadow:
    0 9px 28px 8px rgba(0, 0, 0, 0.05),
    0 3px 6px -4px rgba(0, 0, 0, 0.12),
    0 6px 16px rgba(0, 0, 0, 0.08);
}

.invoice-apply-modal-wrap .ant-modal-header {
  margin: 0;
  padding: 20px 24px 8px;
  border-bottom: none;
}

.invoice-apply-modal-wrap .ant-modal-body {
  padding: 0;
}

.invoice-apply-modal-wrap .ant-modal-close {
  top: 19px;
  inset-inline-end: 24px;
}
</style>
