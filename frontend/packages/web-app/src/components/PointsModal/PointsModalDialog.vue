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
    :body-style="{ padding: 0 }"
    wrap-class-name="points-modal-wrap"
    class="points-modal"
  >
    <div
      class="flex h-[800px] w-full flex-col gap-6 overflow-hidden bg-white dark:bg-[#141414]"
    >
      <PointsModalHeader v-model:active-tab="activeTab" @close="handleClose" />

      <PointsManagePanel
        v-if="activeTab === 'manage'"
        :workspace-name="displayWorkspaceName"
      />
      <ConsumeDetailPanel
        v-else-if="activeTab === 'consume'"
        class="min-h-0 flex-1"
      />
      <OrderManagePanel v-else class="min-h-0 flex-1" />
    </div>
  </a-modal>
</template>
