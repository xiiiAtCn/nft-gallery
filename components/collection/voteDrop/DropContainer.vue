<template>
  <div class="unlockable-container">
    <Loader v-model="isLoading" :minted="justMinted" />
    <CountdownTimer />
    <hr class="text-color my-0" />
    <div class="container is-fluid">
      <div class="columns is-desktop">
        <div class="column is-half-desktop mobile-padding">
          <UnlockableCollectionInfo
            :collection-id="collectionId"
            :description="VOTE_DROP_DESCRIPTION" />
          <hr class="mb-4" />

          <div
            class="is-flex is-justify-content-space-between is-align-items-center my-5">
            <span> {{ $t('mint.unlockable.totalAvailableItem') }}</span>
            <span>{{ totalAvailableMintCount }} / {{ totalCount }}</span>
          </div>
          <UnlockableTag />

          <div>
            <div
              class="is-flex is-justify-content-space-between is-align-items-center my-5">
              <span class="has-text-weight-bold is-size-5">{{
                $t('mint.unlockable.phase')
              }}</span
              ><span
                v-if="mintCountAvailable"
                class="is-flex is-align-items-center">
                <img src="/drop/unlockable-pulse.svg" alt="open" />
                {{ $t('mint.unlockable.open') }}</span
              >
            </div>
            <div
              class="is-flex is-justify-content-space-between is-align-items-center">
              <span>{{ mintedPercent }} %</span
              ><span class="has-text-weight-bold">
                {{ mintedCount }} / {{ totalCount }}
                {{ $t('statsOverview.minted') }}</span
              >
            </div>
          </div>
          <div class="my-5">
            <UnlockableSlider :value="mintedCount / totalCount" />
          </div>
          <div class="my-5">
            <div
              v-if="!userMintedId"
              class="is-flex is-justify-content-space-between">
              <div class="is-flex is-align-items-center">
                <NeoIcon
                  :icon="statusInformation.icon"
                  class="mr-2"
                  :class="statusInformation.iconClass"
                  :pack="statusInformation.iconPack" />
                <span :class="statusInformation.labelClass">{{
                  statusInformation.label
                }}</span>
              </div>
              <div>
                <NeoButton
                  ref="root"
                  class="mb-2 mt-4 mint-button"
                  variant="k-accent"
                  :disabled="mintButtonDisabled"
                  :label="buttonLabel"
                  @click.native="handleMint" />
                <div class="is-flex is-align-items-center mt-2">
                  <NeoIcon icon="timer" pak="far" class="mr-2" />
                  {{ leftTime }}
                </div>
              </div>
            </div>
            <nuxt-link v-else :to="`/${urlPrefix}/gallery/${userMintedId}`">
              <p class="title is-size-4">
                [{{ $t('mint.unlockable.alreadyMinted') }}]
              </p>
            </nuxt-link>
          </div>
        </div>
        <div class="column pt-5 is-flex is-justify-content-center">
          <ImageSlider
            v-if="imageList.length"
            :image-list="imageList"
            :title="$t('mint.unlockable.onlyOneExample')" />
        </div>
      </div>
      <hr class="text-color my-4" />
      <div class="columns is-desktop">
        <div
          class="column is-half-desktop is-flex is-flex-direction-column is-justify-content-center order-1">
          <div
            class="is-flex is-align-items-center has-text-weight-bold is-size-6 mb-2">
            <NeoIcon icon="unlock" class="mr-2" pack="far" />
            {{ $t('mint.unlockable.howItemWork') }}
          </div>
          <div>
            Experience the excitement of unlocking hidden rewards! Get your
            hands on exclusive merchandise (and an NFT!) linked to unlockable
            content. For the next ten hours, the fastest ten individuals can
            mint their very own anime waifu character NFT for free. Simply log
            in with your wallet, click on the "Mint" button, and sign the
            transaction. Afterward, check your profile to find the NFT and click
            "Unlockable Content" to reveal the surprise. Follow the schedule so
            you don't miss this!
          </div>
          <NeoButton
            tag="a"
            href="https://hello.kodadot.xyz/fandom-toolbox/audience-growth/unlockables"
            target="_blank"
            variant="secondary"
            class="mt-2">
            {{ $t('helper.learnMore') }}
          </NeoButton>
        </div>
        <div class="column">
          <img src="/unlockable-introduce.svg" alt="Unlockable" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import CountdownTimer from '@/components/collection/unlockable/CountdownTimer.vue'
import ImageSlider from '@/components/collection/unlockable/ImageSlider.vue'
import UnlockableCollectionInfo from '@/components/collection/unlockable/UnlockableCollectionInfo.vue'
import UnlockableSlider from '@/components/collection/unlockable/UnlockableSlider.vue'
import UnlockableTag from '@/components/collection/unlockable/UnlockableTag.vue'
import { ConnectWalletModalConfig } from '@/components/common/ConnectWallet/useConnectWallet'
import { doWaifu } from '@/services/waifu'
import { sanitizeIpfsUrl } from '@/utils/ipfs'
import { NeoButton, NeoIcon } from '@kodadot1/brick'
import type Vue from 'vue'
import { useCountDown } from '../unlockable/utils/useCountDown'
import {
  VOTE_DROP_AHP_CAMPAIGN,
  VOTE_DROP_AHP_COLLECTION_ID,
  VOTE_DROP_CAMPAIGN,
  VOTE_DROP_COLLECTION_ID,
  VOTE_DROP_DESCRIPTION,
  countDownTime,
} from './const'

import { useCheckReferenDumVote } from '@/composables/drop/useCheckReferenDumVote'

const Loader = defineAsyncComponent(
  () => import('@/components/collection/unlockable/UnlockableLoader.vue')
)

const { $neoModal, $i18n } = useNuxtApp()
const root = ref<Vue>()
const { accountId } = useAuth()

const imageList = ref<string[]>([])
const { urlPrefix } = usePrefix()
const { isLogIn } = useAuth()
const { hours, minutes } = useCountDown(countDownTime)
const justMinted = ref('')
const isLoading = ref(false)
const collectionId = computed(() =>
  urlPrefix.value === 'ahk'
    ? VOTE_DROP_COLLECTION_ID
    : VOTE_DROP_AHP_COLLECTION_ID
)
const { toast } = useToast()

const { isEligibleUser } = useCheckReferenDumVote()

const needCheckEligible = computed(() => {
  return !isLogIn.value
})

const buttonLabel = computed(() => {
  return needCheckEligible.value
    ? 'Check Eligibility'
    : isEligibleUser.value
    ? 'Mint'
    : 'Not Eligible'
})

const statusInformation = computed(() => {
  return needCheckEligible.value
    ? {
        label: $i18n.t('mint.unlockable.ayeVotersOnly'),
        icon: 'circle-info',
        iconClass: 'has-text-grey',
        labelClass: 'has-text-grey',
        iconPack: 'far',
      }
    : isEligibleUser.value
    ? {
        label: $i18n.t('mint.unlockable.eligible'),
        icon: 'circle-check',
        iconPack: 'fas',
        iconClass: 'has-text-success',
      }
    : {
        label: $i18n.t('mint.unlockable.exclusive'),
        icon: 'circle-info',
        iconClass: 'has-text-grey',
        labelClass: 'has-text-grey',
        iconPack: 'far',
      }
})

const leftTime = computed(() => {
  const hoursLeft = hours.value > 0 ? `${hours.value} Hour ` : ''
  const minutesLeft = minutes.value > 0 ? `${minutes.value} Minute ` : ''
  const isFinish = !hoursLeft && !minutesLeft
  return isFinish ? 'Finished' : `${hoursLeft}${minutesLeft}Left`
})

const { data: collectionData, refetch } = useGraphql({
  queryName: 'dropCollectionById',
  variables: {
    id: collectionId.value,
    account: accountId.value,
  },
})

watch(collectionData, () => {
  if (collectionData.value) {
    imageList.value = [
      sanitizeIpfsUrl(collectionData.value?.collectionEntity.image),
    ]
  }
})

const totalCount = 300

const totalAvailableMintCount = computed(
  () => totalCount - collectionData.value?.collectionEntity?.nftCount
)

useSubscriptionGraphql({
  query: `nftEntities(
    orderBy: id_ASC,
    where: { burned_eq: false, collection: { id_eq: "${collectionId.value}" }}
    ) {
      id
  }`,
  onChange: refetch,
})

const mintedCount = computed(() => totalCount - totalAvailableMintCount.value)

const mintedPercent = computed(() => {
  const percent = (mintedCount.value / totalCount) * 100
  return Math.round(percent)
})

const userMintedId = computed(
  () =>
    Boolean(accountId.value) &&
    (collectionData.value?.nftEntitiesConnection?.edges?.[0]?.node?.id ||
      justMinted.value)
)

const mintCountAvailable = computed(() => mintedCount.value < totalCount)

watch(accountId, (id) => {
  justMinted.value = ''
  refetch({
    account: id,
  })
})

const mintButtonDisabled = computed(
  () =>
    Boolean(
      !mintCountAvailable.value || !isEligibleUser.value || userMintedId.value
    ) && !needCheckEligible.value
)

const handleMint = async () => {
  if (!isLogIn.value) {
    $neoModal.open({
      parent: root?.value,
      ...ConnectWalletModalConfig,
    })
    return
  }

  if (isLoading.value) {
    return false
  }
  isLoading.value = true

  const { accountId } = useAuth()

  try {
    const id = await doWaifu(
      {
        address: accountId.value,
        metadata: collectionData.value.collectionEntity.metadata,
        image: collectionData.value.collectionEntity.image,
      },
      urlPrefix.value === 'ahk' ? VOTE_DROP_CAMPAIGN : VOTE_DROP_AHP_CAMPAIGN
    ).then((res) => {
      toast('mint success')
      return `${collectionId.value}-${res.result.sn}`
    })
    // 40s timeout
    setTimeout(() => {
      isLoading.value = false
      justMinted.value = id
      toast('You will be redirected in few seconds')
      return navigateTo(`/${urlPrefix.value}/gallery/${id}`)
    }, 44000)
  } catch (error) {
    toast('failed to mint')
    isLoading.value = false
  }
}
</script>

<style scoped lang="scss">
@import '@/styles/abstracts/variables';

.unlockable-container {
  .mint-button {
    width: 14rem;
    height: 3.5rem;
  }
}

.order-1 {
  order: 1;
}
</style>
