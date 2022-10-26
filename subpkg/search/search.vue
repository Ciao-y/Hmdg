<template>
  <view>
    <view class="search-box">
      <uni-search-bar @input="input" :radius="100">搜索</uni-search-bar>
    </view>
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view
        class="sugg-item"
        v-for="(item, index) in searchResults"
        :key="index"
        @click="gotoDetail(item.goods_id)"
      >
        <view class="goods-name">{{ item.goods_name }}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <!-- 标题区域 -->
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="cleanHistory"></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <uni-tag
          :text="item"
          v-for="(item, index) in historys"
          :key="index"
        ></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data () {
    return {
      timer: null,
      kw: '',
      // 搜索关键词列表
      searchResults: [],
      historyList: []
    };
  },
  onLoad () {
    // 页面渲染就从本地存储读取数据 转换成数组 有就读取  没有就是数组
    this.historyList = JSON.parse( uni.getStorageSync( 'kw' ) || '[]' )
  },
  methods: {
    // 请求搜索列表
    async getSearchList () {
      if ( this.kw === '' ) {
        this.searchResults = []
        return
      }
      // 请求数据
      const { data: res } = await uni.$http.get( '/api/public/v1/goods/qsearch', { query: this.kw } )
      if ( res.meta.status !== 200 ) return uni.$showMsg()
      this.searchResults = res.message

    },
    input ( e ) {
      clearTimeout( this.timer )
      this.timer = setTimeout( () => {
        this.kw = e
        this.getSearchList()
        this.saveSearchHistory()

      }, 1000 )
    },
    gotoDetail ( goods_id ) {
      uni.navigateTo( {
        url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods_id
      } )
    },
    saveSearchHistory () {
      const set = new Set( this.historyList )
      set.delete( this.kw )
      set.add( this.kw )
      this.historyList = Array.from( set )
      // 转换成字符串存储到本地 kw是数组名字 后面是值
      uni.setStorageSync( 'kw', JSON.stringify( this.historyList ) )
    },
    // 清空搜索历史记录
    cleanHistory () {
      this.historyList = []
      uni.setStorageSync( 'kw', '[]' )
    }
  },
  computed: {
    historys () {
      // 翻转数组
      return [ ...this.historyList ].reverse()
    }
  }


}
</script>

<style lang="scss">
  .search-box {
    position: sticky;
    top: 0;
    z-index: 999;
  }
  .sugg-list {
    padding: 0 5px;

    .sugg-item {
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;
      display: flex;
      align-items: center;
      justify-content: space-between;

      .goods-name {
        // 文字不允许换行（单行文本）
        white-space: nowrap;
        // 溢出部分隐藏
        overflow: hidden;
        // 文本溢出后，使用 ... 代替
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
