<template>
  <view class="goods-list">
    <view v-for="(item, i) in goodsList" :key="i" @click="gotoDetail(item)">
      <!-- 为 my-goods 组件动态绑定 goods 属性的值 -->
      <my-goods :goods="item"></my-goods>
    </view>
  </view>
</template>

<script>
export default {
  data () {
    return {
      // 请求参数对象
      queryObj: {
        // 查询关键词
        query: '',
        // 商品分类Id
        cid: '',
        // 页码值
        pagenum: 1,
        // 每页显示多少条数据
        pagesize: 10,
      },
      // 商品列表的数据
      goodsList: [],
      // 总数量，用来实现分页
      total: 0,
      isloading: false,
    }
  },
  onLoad ( options ) {
    this.queryObj.query = options.query || ''
    this.queryObj.cid = options.cid || ''
    this.getGoodsList()
  },
  methods: {
    // 获取商品列表数据的方法
    async getGoodsList ( cb ) {
      this.isloading = true
      const { data: res } = await uni.$http.get( '/api/public/v1/goods/search', this.queryObj )
      this.isloading = false
      cb && cb()
      if ( res.meta.status !== 200 ) return uni.$showMsg()
      this.goodsList = [ ...this.goodsList, ...res.message.goods ]
      this.total = res.message.total
    },
    gotoDetail ( item ) {
      uni.navigateTo( {
        url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
      } )
    }

  },
  // 觸底
  onReachBottom () {
    if ( this.queryObj.pagenum * this.queryObj.pagesize >= this.total )
      return uni.$showMsg( '数据加载完毕！' )
    if ( this.isloading ) return
    // 让页码值自增 +1
    this.queryObj.pagenum += 1
    // 重新获取列表数据
    this.getGoodsList()
  },
  // 下拉刷新
  onPullDownRefresh () {
    // 重置数据
    this.queryObj.pagenum = 1
    this.total = 0
    this.isloading = false
    this.goodsList = []
    // 重新发起请求
    this.getGoodsList( () => uni.stopPullDownRefresh() )
  },
}
</script>

<style lang="scss"></style>
