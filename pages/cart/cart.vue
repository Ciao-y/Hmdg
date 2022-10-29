<template>
  <view class="cart-container" v-if="cart.length !== 0">
    <my-address></my-address>
    <!-- 购物车商品列表的标题区域 -->
    <view class="cart-title">
      <!-- 左侧的图标 -->
      <uni-icons type="shop" size="18"></uni-icons>
      <!-- 描述文本 -->
      <text class="cart-title-text">购物车</text>
      <!-- 商品列表区域 -->
    </view>
    <!-- 商品列表区域 -->

    <uni-swipe-action>
      <block v-for="(goods, index) in cart" :key="index">
        <uni-swipe-action-item
          :right-options="options"
          @click="swipeActionClickHandler(goods)"
          :auto-close="autoClose"
        >
          <my-goods
            :goods="goods"
            :showRadio="true"
            @radio-change="radioChangeHandler"
            @num-change="numberChangeHandler"
            :show-num="true"
          ></my-goods>
        </uni-swipe-action-item>
      </block>
    </uni-swipe-action>

    <my-settle></my-settle>
  </view>
  <view class="empty-cart" v-else>
    <image src="/static/cart_empty@2x.png" class="empty-img"></image>
    <text class="tip-text">空空如也~</text>
  </view>
</template>

<script>
import { mapState, mapMutations } from 'vuex'
import badgeMix from '@/mixins/tabbar-badge.js'
import myAddress from '../../components/my-address/my-address.vue'
export default {
  components: { myAddress },
  mixins: [badgeMix],
  data() {
    return {
      options: [
        {
          text: '删除', // 显示的文本内容
          style: {
            backgroundColor: '#C00000', // 按钮的背景颜色
          },
        },
      ],
    }
  },
  computed: {
    // 将 m_cart 模块中的 total 映射为当前页面的计算属性
    ...mapState('m_cart', ['cart']),
  },

  methods: {
    ...mapMutations('m_cart', ['updateGoodsState', 'updateGoodsCount', 'removeGoodsById']),
    setBadge() {
      uni.setTabBarBadge({
        index: 2, // 索引
        text: this.total + '', // 注意：text 的值必须是字符串，不能是数字
      })
    },
    radioChangeHandler(e) {
      this.updateGoodsState(e)
    },
    // 商品的数量发生了变化
    numberChangeHandler(e) {
      this.updateGoodsCount(e)
    },
    swipeActionClickHandler(e) {
      console.log(e)
      this.removeGoodsById(e.goods_id)
    },
  },
}
</script>

<style lang="scss">
.cart-title {
  height: 40px;
  display: flex;
  align-items: center;
  font-size: 14px;
  padding-left: 5px;
  border-bottom: 1px solid #efefef;
  .cart-title-text {
    margin-left: 10px;
  }
}
.goods-item {
  // 让 goods-item 项占满整个屏幕的宽度
  width: 750rpx;
  // 设置盒模型为 border-box
  box-sizing: border-box;
  display: flex;
  padding: 10px 5px;
  border-bottom: 1px solid #f0f0f0;
}
.cart-container {
  padding-bottom: 50px;
}
.empty-cart {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 150px;

  .empty-img {
    width: 90px;
    height: 90px;
  }

  .tip-text {
    font-size: 12px;
    color: gray;
    margin-top: 15px;
  }
}
</style>
