<template>
  <view>
    <view class="search-box">
      <!-- 使用 uni-ui 提供的搜索组件 -->
      <uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
      <!-- 当搜索结果列表的长度不为 0的时候（searchResults.length !== 0），需要展示搜索建议区域，隐藏搜索历史区域
      当搜索结果列表的长度等于 0的时候（searchResults.length === 0），需要隐藏搜索建议区域，展示搜索历史区域 -->
    </view>
    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="(item,i) in  searchResults" :key=i @click="gotoDetail(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>

      </view>
    </view>

    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <!-- 标题区域 -->
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="clean"></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <!-- 不再使用historyList 而使用计算属性histories来实现页面的渲染 -->
        <uni-tag :text="item" v-for="(item, i) in histories" :key="i" @click="gotoGoodsList(item)"></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        timer: null, //定时器的id
        kw: '', //搜索关键词
        searchResults: [], //搜索的结果列表
        //搜索历史的数组
        historyList: ['a', 'app', 'apple']
      };
    },
    onLoad() {
      //JSON.parse() 方法用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。
    this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    methods: {
      // 定义一个input输入事件的处理函数
      input(e) {
        // 清除 timer 对应的延时器
        clearTimeout(this.timer)
        // 重新启动一个延时器，并把 timerId 赋值给 this.timer
        this.timer = setTimeout(() => {
          // 如果 1000 毫秒内，没有触发新的输入事件，则为搜索关键词赋值
          this.kw = e.valueOf()
          //调用获取搜索结果列表的方法
          this.getSearchList()
        }, 1000)
      },
      async getSearchList() {
        //判断搜索关键词是否为空
        if (this.kw.trim().length === 0) {
          this.searchResults = []
          return
        }
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/goods/qsearch', {
          query: this.kw
        }) //查文档可知 这次请求我们需要传一个名为query的参数
        if (res.meta.status !== 200) return uni.$showMsg()
        this.searchResults = res.message
        this.saveSearchHistory()
      },
      // 保存搜索历史记录
      saveSearchHistory() {
        //Array.from(new Set(arr))数组去重
        const set = new Set(this.historyList)
        set.delete(this.kw) //把kw删掉（如果有的话）
        set.add(this.kw)
        this.historyList = Array.from(set)
        //对搜索历史记录进行持久化存储
        uni.setStorageSync('kw', JSON.stringify(this.historyList))

      },

      gotoDetail(item) {
        uni.navigateTo({
          // 指定详情页面的 URL 地址，并给页面传递 goods_id 参数
          url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
        })

      },
      clean() {
        this.historyList = []
        uni.setStorageSync('kw', [])
      },
      gotoGoodsList(kw){
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query=' + kw
        })
      }
    },
    computed: {
      histories() {
        return [...this.historyList].reverse()

      }
    }
  }
</script>

<style lang="scss">
  /*  设置搜索框吸顶效果 */
  .search-box {
    position: sticky;
    top: 0;
    z-index: 999;
    /* 提高层级 防止被覆盖 */
  }

  .sugg-list {
    padding: 0 5px;

    .sugg-item {
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;
      display: flex;
      /* 使得文字和小箭头左右布局 */
      align-items: center;
      /* 纵向居中 */
      justify-content: space-between;
      /* 横向上是两边贴边对齐 */

      .goods-name {
        /* 文字不允许换行（单行文本） */
        white-space: nowrap;
        /* white-space CSS 属性是用来设置如何处理元素中的空白 */
        /* 溢出部分隐藏 */
        overflow: hidden;
        /* 文本溢出后，使用 ... 代替 */
        text-overflow: ellipsis;
        margin-right: 3px;
      }
    }
  }

  .history-box {
    padding: 0 5px;

    .history-title {
      display: flex;
      justify-content: space-between;
      align-items: center;
      height: 40px;
      font-size: 13px;
      border-bottom: 1px solid #efefef;
    }

    .history-list {
      display: flex;
      flex-wrap: wrap;

      .uni-tag {
        margin-top: 5px;
        margin-right: 5px;
      }
    }
  }
</style>
