<template>
  <div class="py-2 px-6">
    <div class="py-2 border-top border-k-shade" />
    <div class="is-flex is-justify-content-space-between">
      <div class="is-flex">
        <div>
          <BasicImage
            :src="avatar"
            :alt="nft?.name"
            class="border image is-48x48" />
        </div>

        <div
          class="is-flex is-flex-direction-column is-justify-content-space-between ml-4 limit-width">
          <div
            class="has-text-weight-bold has-text-color line-height-1 no-wrap is-clipped is-ellipsis">
            {{ nft.name }}
          </div>
          <div class="line-height-1 no-wrap is-clipped is-ellipsis">
            {{ nft.collection?.name || nft.collection.id }}
          </div>
        </div>
      </div>

      <NeoButton
        class="has-text-grey pt-4"
        variant="text"
        no-shadow
        icon="trash"
        icon-pack="far"
        @click.native="listingCartStore.removeItem(nft.id)" />
    </div>
    <div class="is-flex is-justify-content-space-between pt-2">
      <div class="is-flex is-flex-direction-column">
        <div class="has-text-grey is-size-7">{{ $t('activity.floor') }}</div>
        <span>{{ floor }}</span>
      </div>

      <div class="is-flex is-align-items-end pt-2">
        <ListingCartPriceInput
          v-model="listingCartStore.getItem(nft.id).listPrice" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { parseNftAvatar } from '@/utils/nft'
import BasicImage from '@/components/shared/view/BasicImage.vue'
import ListingCartPriceInput from '@/components/common/listingCart/ListingCartPriceInput.vue'
import { NeoButton } from '@kodadot1/brick'
import { ListCartItem, useListingCartStore } from '@/stores/listingCart'
import { formatBalance } from '@polkadot/util'
const { decimals, chainSymbol } = useChain()

const listingCartStore = useListingCartStore()

const avatar = ref<string>()

const props = defineProps<{
  nft: ListCartItem
}>()

const getAvatar = async () => {
  if (props.nft) {
    avatar.value = await parseNftAvatar(props.nft)
  }
}

const floor = computed(() =>
  formatBalance(props.nft.collection.floor, {
    decimals: decimals.value,
    withUnit: chainSymbol.value,
  })
)

onMounted(() => {
  getAvatar()
})
</script>
<style scoped lang="scss">
.limit-width {
  max-width: 170px;
}

.line-height-1 {
  line-height: 1;
}
</style>
