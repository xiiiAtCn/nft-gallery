<template>
  <div class="is-flex">
    <div class="is-flex price-input border border-k-shade">
      <input
        v-model="model"
        type="text"
        class="price-input height-40 theme-background-color has-text-color"
        :placeholder="$t('offer.price')" />
      <div class="px-5 is-flex is-align-items-center">{{ chainSymbol }}</div>
    </div>
    <NeoButton
      v-if="check"
      no-shadow
      class="shade-border-color ml-2 height-40"
      icon="check"
      @click.native="emit('confirm')" />
  </div>
</template>
<script setup lang="ts">
import { NeoButton } from '@kodadot1/brick'
import { useVModel } from '@vueuse/core'
const props = defineProps<{ value?: number | string; check?: boolean }>()
const emit = defineEmits(['confirm', 'input'])
const model = useVModel(props, 'value', emit, { eventName: 'input' })
const { chainSymbol } = useChain()
watch(model, (newValue) => {
  const sanitizedValue = (newValue?.toString() ?? '').replace(/[^0-9.]/g, '')
  if (sanitizedValue !== newValue) {
    model.value = sanitizedValue
  }
})
</script>

<style scoped lang="scss">
@import '@/styles/abstracts/variables';

.price-input {
  @include ktheme() {
    &:focus-within {
      border-color: theme('border-color');
    }
  }

  input {
    border: none;
    outline: none;
    width: 5em;
    text-indent: 10px;
  }
}

.height-40 {
  height: 40px;
}

.shade-border-color {
  @include ktheme() {
    border-color: theme('k-shade');
    &:hover {
      border-color: theme('border-color');
    }
  }
}
</style>
