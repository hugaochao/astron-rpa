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
    :width="980"
    :footer="null"
    :closable="false"
    :z-index="1100"
    centered
    :body-style="{ padding: 0, overflow: 'hidden' }"
    wrap-class-name="points-modal-wrap"
    class="points-modal"
  >
    <div
      class="points-modal-inner flex min-h-0 w-full min-w-0 flex-1 flex-col gap-6 overflow-hidden bg-white dark:bg-[#141414]"
    >
      <PointsModalHeader v-model:active-tab="activeTab" class="shrink-0" @close="handleClose" />

      <PointsManagePanel
        v-if="activeTab === 'manage'"
        class="min-h-0 flex-1"
        :workspace-name="displayWorkspaceName"
      />
      <ConsumeDetailPanel v-else-if="activeTab === 'consume'" class="min-h-0 flex-1" />
      <OrderManagePanel v-else class="min-h-0 flex-1" />
    </div>
  </a-modal>
</template>

<style lang="scss">
/* 弹窗最大 980×740，wrap 上下留白；表格等在各自卡片内滚动 */
.points-modal-wrap.ant-modal-wrap {
  align-items: center;
  padding: 5vh 24px;
}

.points-modal-wrap .ant-modal {
  top: 0 !important;
  max-width: 980px;
  max-height: 740px;
  margin: 0 auto;
  padding-bottom: 0;
}

.points-modal-wrap .ant-modal-content {
  display: flex;
  max-height: 740px;
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
