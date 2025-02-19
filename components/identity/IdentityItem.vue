<template>
  <div class="is-flex is-align-items-center">
    <Avatar :size="48" :value="account" />
    <div v-if="variant === 'button'" class="identity-item-button pl-3">
      <div class="has-text-weight-bold identity-item-button-label mb-1">
        {{ label }}
      </div>
      <NeoButton
        v-safe-href="`/${prefix}/u/${account}`"
        no-shadow
        rounded
        tag="a"
        size="small"
        icon="arrow-right-long">
        {{ buttonLabel }}
      </NeoButton>
    </div>
    <div v-else class="p-3">
      <div class="has-text-grey is-size-6">
        {{ label }}
      </div>
      <component
        :is="disableIdentityLink ? 'div' : 'nuxt-link'"
        class="identity-name has-text-weight-bold"
        :to="`/${prefix}/u/${account}`">
        <Identity
          :address="account"
          :hide-identity-popover="hideIdentityPopover" />
      </component>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { defineAsyncComponent } from 'vue'
import { NeoButton } from '@kodadot1/brick'

const Identity = defineAsyncComponent(
  () => import('@/components/identity/IdentityIndex.vue')
)

withDefaults(
  defineProps<{
    label: string
    account: string
    prefix: string
    variant?: 'normal' | 'button'
    buttonLabel?: string
    hideIdentityPopover?: boolean
    disableIdentityLink?: boolean
  }>(),
  {
    variant: 'normal',
    buttonLabel: 'View Profile',
    hideIdentityPopover: false,
    disableIdentityLink: false,
  }
)
</script>

<style lang="scss" scoped>
.identity-item-button {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 3rem;

  &-label {
    line-height: 1;
  }
}
</style>
