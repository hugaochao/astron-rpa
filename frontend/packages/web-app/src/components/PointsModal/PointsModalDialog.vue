<script setup lang="ts">
import { NiceModal } from '@rpa/components'
import { computed, ref } from 'vue'

import { useUserStore } from '@/stores/useUserStore'

import PointsModalHeader from './components/PointsModalHeader.vue'
import ConsumeDetailPanel from './components/panels/ConsumeDetailPanel.vue'
import OrderManagePanel from './components/panels/OrderManagePanel.vue'
import PointsManagePanel from './components/panels/PointsManagePanel.vue'
import type { PointsModalTabKey } from './types'

const props = withDefaults(
  defineProps<{
    workspaceName?: string
  }>(),
  {},
)

const modal = NiceModal.useModal()
const userStore = useUserStore()
const activeTab = ref<PointsModalTabKey>('manage')

const displayWorkspaceName = computed(
  () => props.workspaceName ?? userStore.currentTenant?.name ?? '星辰科技工作空间',
)

function handleClose() {
  modal.hide()
}
</script>

<template>
  <a-modal
    v-bind="NiceModal.antdModal(modal)"
    :width="1080"
    :footer="null"
    :closable="false"
    :z-index="1100"
    centered
    :body-style="{ padding: 0, overflow: 'hidden' }"
    wrap-class-name="points-modal-wrap"
    class="points-modal"
  >
    <div
      class="points-modal-inner flex max-h-full min-h-0 w-full flex-1 flex-col gap-6 overflow-hidden bg-white dark:bg-[#141414]"
    >
      <PointsModalHeader v-model:active-tab="activeTab" class="shrink-0" @close="handleClose" />

      <PointsManagePanel
        v-if="activeTab === 'manage'"
        :workspace-name="displayWorkspaceName"
      />
      <ConsumeDetailPanel v-else-if="activeTab === 'consume'" />
      <OrderManagePanel v-else />
    </div>
  </a-modal>
</template>

<style lang="scss">
/* 弹窗整体最高 90vh，wrap 上下各留 5vh；各 Tab 面板内部自行 overflow-y-auto */
.points-modal-wrap.ant-modal-wrap {
  align-items: center;
  padding: 5vh 24px;
}

.points-modal-wrap .ant-modal {
  top: 0 !important;
  max-height: 90vh;
  margin: 0 auto;
  padding-bottom: 0;
}

.points-modal-wrap .ant-modal-content {
  display: flex;
  max-height: 90vh;
  flex-direction: column;
  overflow: hidden;
}

.points-modal-wrap .ant-modal-body {
  display: flex;
  min-height: 0;
  flex: 1;
  flex-direction: column;
  padding: 0 !important;
  overflow: hidden;
}

.points-modal-inner {
  box-sizing: border-box;
  min-height: 0;
}
</style>
