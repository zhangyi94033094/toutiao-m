<template>
  <div class="article-list">
 <van-pull-refresh
      v-model="isRefreshLoading"
      @refresh="onRefresh"
      :success-text="refreshSuccessText"
      :success-duration="1500"
    >
     <van-list
        v-model="loading"
        :finished="finished"
        finished-text="没有更多了"
        :error.sync="error"
        error-text="请求失败，点击重新加载"
        @load="onLoad"
      >
       <article-item
          v-for="(article, index) in list"
          :key="index"
          :article="article"
        />
      </van-list>
    </van-pull-refresh>
    </div>
</template>

<script>
import { getArticles } from '@/api/article'
import ArticleItem from '@/components/article-item'
export default {
  name: 'ArticleList',
  components: {
    ArticleItem
  },
  props: {
    channel: {
      type: Object,
      required: true
    }
  },
  data () {
    return {
      list: [], // 存储列表数据的数组
      loading: false, // 控制加载中 loading 状态，默认不 loading
      finished: false,
      timestamp: null,
      error: false,
      isRefreshLoading: false,
      refreshSuccessText: '刷新成功' // 下拉刷新成功提示文本
    }
  },
  methods: {
    async onLoad () {
      try {
        // 1. 请求获取数据
        const { data } = await getArticles({
          channel_id: this.channel.id,
          timestamp: this.timestamp || Date.now(),
          with_top: 1 // 是否包含置顶
        })
        // 2. 把请求结果数据放到 list 数组中
        const { results } = data.data
        this.list.push(...results)
        // 3. 本次数据加载完毕后要把 loading 设置为 false，loading 关闭后才能触发下一次的加载更多
        this.loading = false
        // 4. 数据加载完毕后，要把 finished 设置为 true，不再触发加载更多了
        if (results.length) {
          this.timestamp = data.data.pre_timestamp
        } else {
          this.finished = true
        }
      } catch (err) {
        this.error = true
        this.loading = false
      }
    },
    async onRefresh () {
      try {
        // 请求获取数据
        const { data } = await getArticles({
          channel_id: this.channel.id,
          timestamp: Date.now(),
          with_top: 1
        })
        /* if (Math.random() > 0.5) {
          JSON.parse('xxx')
        } */
        // 将数据追加到列表的顶部
        const { results } = data.data
        this.list.unshift(...results)
        // 关闭下拉刷新的 loading 状态
        this.isRefreshLoading = false
        // 更新下拉刷新成功提示的文本
        this.refreshSuccessText = `刷新成功，更新了${results.length}条数据`
      } catch (err) {
        // 关闭下拉刷新的 loading 状态
        this.isRefreshLoading = false
        // 更新下拉刷新成功提示的文本
        this.refreshSuccessText = '刷新失败'
      }
    }
  }
}
</script>

<style scoped lang="less">
.article-list {
    height: 79vh;
    overflow-y: auto;
}
</style>
