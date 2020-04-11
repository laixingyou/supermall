<template>
  <div id="detail">
    <detail-nav class="detail-nav" @titleClick="titleClick" ref="nav"></detail-nav>
    <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll">
      <!-- <ul>
        <li v-for="item in $store.state.cartList">{{item}}</li>
      </ul>-->
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods" />
      <detail-shop-info :shop="shop" />
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad" ref="goodsInfo" />
      <detail-param-info :param-info="paramInfo" ref="params" />
      <detail-comment-info :comment-info="commentInfo" ref="comment" />
      <good-list :goods="recommends" ref="recommend"></good-list>
    </scroll>
    <detail-bottom-bar @addCart="addCart" @ToBuy="ToBuy"></detail-bottom-bar>
    <back-top @click.native="backClick" v-show="topIsShow"></back-top>
  </div>
</template>

<script>
import DetailNav from "./childComps/DetailNav";
import DetailSwiper from "./childComps/DetailSwiper";
import DetailBaseInfo from "./childComps/DetailBaseInfo";
import DetailShopInfo from "./childComps/DetailShopInfo";
import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
import DetailParamInfo from "./childComps/DetailParamInfo";
import DetailCommentInfo from "./childComps/DetailCommentInfo";
import DetailBottomBar from "./childComps/DetailBottomBar";

import GoodList from "components/content/goods/GoodList";

import Scroll from "components/common/scroll/Scroll";
import { itemListenerMixin, backTopMixin } from "common/mixin";
import { mapActions } from "vuex";

import {
  getDetail,
  Goods,
  Shop,
  GoodsParam,
  getRecommend
} from "network/detail";

export default {
  name: "Detail",
  components: {
    DetailNav,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    GoodList,
    DetailBottomBar,
    Scroll
  },
  mixins: [itemListenerMixin, backTopMixin],

  data() {
    return {
      iid: null,
      topImages: [],
      goods: {},
      shop: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      recommends: [],
      themeTopYs: [],
      currentIndex: 0
    };
  },
  created() {
    this.iid = this.$route.params.iid;
    getDetail(this.iid).then(res => {
      // console.log(res);
      const data = res.result;

      this.topImages = data.itemInfo.topImages;
      this.goods = new Goods(
        data.itemInfo,
        data.columns,
        data.shopInfo.services
      );
      this.shop = new Shop(data.shopInfo);
      this.detailInfo = data.detailInfo;
      this.paramInfo = new GoodsParam(
        data.itemParams.info,
        data.itemParams.rule
      );
      if (data.rate.cRate !== 0) {
        this.commentInfo = data.rate.list[0];
      }
    });
    getRecommend().then(res => {
      this.recommends = res.data.list;
    });
  },
  methods: {
    ...mapActions({ add: "addCart" }),
    // 点击navbar滚动到对应的位置
    titleClick(index) {
      this.$refs.scroll.scroll.scrollTo(0, -this.themeTopYs[index], 500);
      console.log(-this.themeTopYs[index]);
    },
    imageLoad() {
      this.$refs.scroll.refresh();
      this.themeTopYs = [];
      this.themeTopYs.push(0);
      this.themeTopYs.push(this.$refs.params.$el.offsetTop - 44);
      this.themeTopYs.push(this.$refs.comment.$el.offsetTop - 44);
      this.themeTopYs.push(this.$refs.recommend.$el.offsetTop - 44);
      this.themeTopYs.push(Number.MAX_VALUE);

      console.log(this.themeTopYs);
    },
    contentScroll(position) {
      // 获取Y值
      const positionY = -position.y;
      // 与themeTopY进行比对，滚动时让navbar中标题和内容对应
      let length = this.themeTopYs.length;
      for (let i = 0; i < length - 1; i++) {
        if (
          this.currentIndex !== i &&
          positionY >= this.themeTopYs[i] &&
          positionY < this.themeTopYs[i + 1]
        ) {
          this.currentIndex = i;
          this.$refs.nav.currentIndex = this.currentIndex;
          console.log(this.currentIndex);
        }
      }
      // 判断backtop是否显示
      this.topIsShow = -position.y > this.$refs.goodsInfo.$el.offsetTop;
    },
    addCart() {
      // 获取购物车需要展示的信息
      const product = {};
      product.image = this.topImages[0];
      product.title = this.goods.title;
      product.desc = this.goods.desc;
      product.price = this.goods.realPrice;
      product.iid = this.iid;
      this.add(product).then(res => {
        console.log(res);
        this.$toast.show(res, 2000);
      });
      // this.$store.dispatch("addCart", product).then(res => {
      //   console.log(res);
      // });
      // 放入到购物车中
      //   if (product.iid) {
      //     this.$store.dispatch("addCart", product).then(res => {
      //       this.$toast.show(res, 1500);
      //     });
      //   } else {
      //     this.$toast.show("添加失败请刷新重试", 1500);
      //   }
    },
    ToBuy() {
      this.$toast.show("去付款中", 1500);
    }
  },
  destroyed() {
    // 离开页面时取消全局事件的监听
    this.$bus.$off("imageLoad", this.itemImgListener);
  }
};
</script>

<style scoped>
#detail {
  position: relative;
  z-index: 9;
  background-color: #fff;
  height: 100vh;
}
.detail-nav {
  position: relative;
  z-index: 9;
  background-color: #fff;
}
.content {
  height: calc(100% - 93px);
}
</style>
