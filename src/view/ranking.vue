<template lang="pug">
#ranking-view
  .body-inner
    label
      strong Mode：
      select(v-model="Mode")
        option(value="") 默认·其实就是日榜
        option(value="daily") 日榜
        option(value="weekly") 周榜
        option(value="monthly") 月榜
        option(value="daily_ai") AI日榜
        option(value="rookie") 新人榜
        option(value="original") 原创榜
        option(value="male") 男性向
        option(value="female") 女性向
        option(value="daily_r18") 日榜·R18
        option(value="weekly_r18") 周榜·R18
        option(value="daily_ai_r18") AI日榜·R18
        option(value="male_r18") 男性向·R18
        option(value="female_r18") 女性向·R18
    label
      strong Content：
      select(v-model="Content")
        option(value="") 综合
        option(value="illust") 插画
        option(value="ugoira") 动图
        option(value="manga") 漫画  
    label
      strong Date（int, e.g. 2nd. Nov. 2020 --> 20201102）：
      input(v-model='strDate', type='text')
      
  
    div
      button(@click.prevent='gotoURL') GO
    //- Error
    section(v-if='error')
      h1 排行榜加载失败
      ErrorPage(:description='error' title='出大问题')

    //- Loading
    section(v-if='loading')
      h1 排行榜加载中……
      .loading
        Placeholder

    //- Result
    section(v-if='list')
      h1 {{ list.date.toLocaleDateString('zh', { dateStyle: 'long' }) }}排行榜
      ArtworkLargeList(:rank-list='list.contents')
</template>

<script lang="ts" setup>
import ArtworkLargeList from '@/components/ArtworksList/ArtworkLargeList.vue'
import ErrorPage from '@/components/ErrorPage.vue'
import Placeholder from '@/components/Placeholder.vue'

import type { ArtworkRank } from '@/types'
import { getCache, setCache } from './siteCache'
import { ajax } from '@/utils/ajax'
import { effect } from 'vue'
import { setTitle } from '@/utils/setTitle'

const error = ref('')
const loading = ref(true)
const list = ref<{
  date: Date
  contents: ArtworkRank[]
} | null>(null)
const route = useRoute()
const router = useRouter()

const Mode = ref('')
const Content = ref('')
const strDate = ref('')

async function init({
  strDate,
  Content,
  Mode,
}: {
  strDate?: string
  Content?: string
  Mode?: string
}): Promise<void> {
  loading.value = true
  list.value = getCache('ranking.rankingList')
  if (list.value) {
    loading.value = false
    return
  }
  try {
    const { p, mode, content, date } = route.query
    const searchParams = new URLSearchParams()
    if (p && typeof p === 'string') searchParams.append('p', p)
    if (content && typeof content === 'string') searchParams.append('content', content)
    if (mode && typeof mode === 'string') searchParams.append('mode', mode)
    if (date && typeof date === 'string') searchParams.append('date', date)
    searchParams.append('format', 'json')
    const { data } = await ajax.get<{
      date: string
      contents: ArtworkRank[]
    }>('/ranking.php', { params: searchParams })
    // Date
    const rankingDate = data.date
    const listValue = {
      date: new Date(
        +rankingDate.substring(0, 4),
        +rankingDate.substring(4, 6) - 1,
        +rankingDate.substring(6, 8)
      ),
      contents: data.contents,
    }
    list.value = listValue
    setCache('ranking.rankingList', listValue)
  } catch (err) {
    if (err instanceof Error) {
      error.value = err.message
    } else {
      error.value = '哎呀，出错了！'
    }
  } finally {
    loading.value = false
  }
}

function gotoURL() {
  router.push(`/ranking?mode=${Mode.value}&content=${Content.value}` + (strDate.value==''?``:`&date=${strDate.value}`))
}

effect(() =>
  setTitle(
    list.value?.date?.toLocaleDateString('zh', { dateStyle: 'long' }),
    'Ranking'
  )
)

onBeforeRouteUpdate(async (to) => {
  const params = route.params as {
    strDate?: string
    Content?: string
    Mode?: string
  }
  await init(params)
})

onMounted(() => {
  const params = route.params as {
    strDate?: string
    Content?: string
    Mode?: string
  }
  init(params)
})
</script>

<style scoped lang="sass">

.loading
  text-align: center
</style>
