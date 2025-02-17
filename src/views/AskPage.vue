<script setup lang="ts">
import { computed, ref } from 'vue'
import { storeToRefs } from 'pinia'
import NewsItemLoader from '@/components/NewsItemLoader.vue'
import { useNewStore } from '@/stores/news'
import AskItem from '@/components/AskItem.vue'
import IconNext from '@/components/icons/Next.vue'
import IconPrevious from '@/components/icons/Previous.vue'
import EmptyNews from '@/components/EmptyNews.vue'

const newsStore = useNewStore()
const { topAsk, isNewsListFetching } = storeToRefs(newsStore)

const currentPage = ref(1)
const totalPage = computed(() => {
  const totalNews = topAsk.value?.length ?? 1
  if (totalNews % 20 === 0)
    return totalNews / 20
  else return Math.floor(totalNews / 20) + 1
})

const filteredNews = computed(() => {
  const fst = (currentPage.value - 1) * 20
  const lst = (currentPage.value * 20) - 1

  return topAsk.value?.filter((val, idx) => {
    if (idx <= lst && idx >= fst)
      return true
    else return false
  })
})

const isNewsEmpty = computed(() => {
  if (filteredNews.value?.length > 0)
    return false
  else return true
})

const onNextClick = () => {
  if (currentPage.value !== totalPage.value)
    currentPage.value += 1
}

const onPreviousClick = () => {
  if (currentPage.value !== 1)
    currentPage.value -= 1
}
</script>

<template>
  <div class="flex flex-col bg-white h-full w-full">
    <template v-if="isNewsListFetching">
      <div v-for="n in 10" :key="n" class="flex-grow">
        <NewsItemLoader :id="n" />
      </div>
      <NewsItemLoader />
    </template>
    <template v-else>
      <div v-for="n in filteredNews" :key="n" class="flex-grow">
        <Suspense>
          <template #default>
            <AskItem :id="n" />
          </template>
          <template #fallback>
            <NewsItemLoader />
          </template>
        </Suspense>
      </div>
      <div v-if="(isNewsEmpty)" class="flex-grow">
        <EmptyNews />
      </div>
      <div class="flex justify-center items-center text-center my-2 py-2 space-x-3">
        <button :disabled="currentPage === 1" @click="onPreviousClick">
          <IconPrevious :width="20" />
        </button>
        <p> {{ currentPage }}/{{ totalPage }}</p>
        <button :disabled="currentPage === totalPage" @click="onNextClick">
          <IconNext :width="20" />
        </button>
      </div>
    </template>
  </div>
</template>
