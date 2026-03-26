<script setup lang="ts">
import { NiceModal } from '@rpa/components'
import { inject } from 'vue'

import { PointsModal } from '@/components/PointsModal'
import { useUserStore } from '@/stores/useUserStore'

import PointsDropdown from './PointsDropdown.vue'
import { OPEN_HEADER_UPGRADE_CONSULT_KEY } from './headerUpgradeConsult'
import { TENANT_EDITION } from './tenantEdition'

const userStore = useUserStore()
const openHeaderUpgradeConsult = inject(OPEN_HEADER_UPGRADE_CONSULT_KEY, () => {})

/** 与订阅/租户接口对齐后改为实际套餐 */
const planEdition = TENANT_EDITION.Personal

const points = 200
const badges = 50
const isNegative = points < 0

function handlePointsUpgrade() {
  openHeaderUpgradeConsult()
}

function handleUsageDetails() {
  NiceModal.show(PointsModal, {
    workspaceName: userStore.currentTenant?.name,
  })
}
</script>

<template>
  <a-dropdown placement="bottom">
    <template #overlay>
      <PointsDropdown
        :plan-edition="planEdition"
        :points="points"
        :dialogue-count="badges"
        @upgrade="handlePointsUpgrade"
        @usage-details="handleUsageDetails"
      />
    </template>

    <div class="inline-flex items-center gap-[2px] cursor-pointer">
      <div
        class="inline-flex items-center gap-3 px-3 py-1.5 rounded-l-full"
        :class="isNegative ? 'bg-[#FBDAD8]' : 'bg-[#D7D7FF]/[.4]'"
      >
        <div class="flex items-center gap-1">
          <rpa-icon 
            name="ai" 
            :color="isNegative ? '#FF4D4F' : '#726fff'" 
            size="16" 
          />
          <div class="text-center text-xs font-normal leading-4" :class="isNegative ? 'text-[#FF4D4F]' : 'text-primary'">
            {{ points.toLocaleString() }}
          </div>
        </div>
      </div>

      <div class="inline-flex items-center gap-3 px-3 py-1.5 bg-[#D7D7FF]/[.4] rounded-r-full">
        <div class="flex items-center gap-1">
          <rpa-icon name="magic-wand" color="#726fff" size="16" />
          <div class="text-center text-primary text-xs font-normal leading-4">
            {{ badges }}
          </div>
        </div>
      </div>
    </div>
  </a-dropdown>
</template>
