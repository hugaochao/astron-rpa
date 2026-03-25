<script setup lang="ts">
import { Button, Modal } from 'ant-design-vue'
import { ref, watch } from 'vue'

import { Icon as RpaIcon } from '../../../../Icon'
import type { AuthType } from '../../../interface'

import ConsultModal from './ConsultModal.vue'
import ConsultUpgradeTrigger from './ConsultUpgradeTrigger.vue'

const props = defineProps({
  /** 仅挂载咨询弹窗，由外部通过 ref.openModal() 打开（与触发器 UI 解耦） */
  modalOnly: {
    type: Boolean,
    default: false,
  },
  authType: {
    type: String as () => AuthType,
    default: 'uap',
  },
  trigger: {
    type: String as () => 'button' | 'modal',
    default: 'button',
  },
  buttonConf: {
    type: Object as () => {
      buttonType: 'tag' | 'text' | 'button'
      buttonTxt?: string
      currentEdition?: 'personal' | 'professional' | 'enterprise'
      expirationDate?: string
      shouldAlert?: boolean
    } | undefined,
    default: undefined,
  },
  customClass: {
    type: String,
    default: undefined,
  },
  modalConfirm: {
    type: Object as () => {
      title: string
      content: string
      okText: string
      cancelText: string
    } | undefined,
    default: undefined,
  },
  consult: {
    type: Object as () => {
      consultTitle?: string
      consultEdition?: 'professional' | 'enterprise'
      consultType: 'consult' | 'renewal'
    } | undefined,
    default: undefined,
  },
})

const confData = ref(props)

watch(
  () => props,
  (p) => {
    if (p.modalOnly)
      confData.value = { ...p }
  },
  { deep: true },
)
const consultModalRef = ref<InstanceType<typeof ConsultModal> | null>(null)
function openModal() {
  if (confData.value.authType !== 'casdoor')
    consultModalRef.value?.showModal()
}

function init(config: Omit<typeof props, 'modalOnly'>) {
  confData.value = { modalOnly: false, ...config } as typeof props
  if (confData.value.trigger === 'modal') {
    Modal.confirm({
      ...confData.value.modalConfirm!,
      onOk() {
        openModal()
      },
    })
  }
}

defineExpose({
  init,
  openModal,
})
</script>

<template>
  <template v-if="modalOnly">
    <ConsultModal ref="consultModalRef" v-bind="confData?.consult" />
  </template>
  <div v-else class="w-full" :class="confData?.customClass">
    <template v-if="confData?.trigger === 'button'">
      <ConsultUpgradeTrigger
        v-if="confData?.buttonConf?.buttonType === 'tag'"
        :auth-type="confData.authType"
        :button-conf="confData.buttonConf"
        @open="openModal"
      />
      <span v-else-if="confData?.buttonConf?.buttonType === 'text'" @click="openModal">{{ confData?.buttonConf?.buttonTxt }}</span>
      <Button v-else type="primary" ghost block class="border !border-[#0000001A] dark:!border-[#FFFFFF29]" @click="openModal">
        <span class="!flex items-center justify-center text-[12px] text-[#000000D9] dark:text-[#FFFFFFD9]">
          <RpaIcon class="w-[16px] h-[16px] mr-[4px]" name="python-package-plus" />
          <span>{{ confData?.buttonConf?.buttonTxt || '创建新的空间' }}</span>
        </span>
      </Button>
    </template>
    <ConsultModal ref="consultModalRef" v-bind="confData?.consult" />
  </div>
</template>
