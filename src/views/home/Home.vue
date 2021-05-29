<template>
  <div id="home" class="wrapper">
    <!-- 首页顶部导航栏 -->
    <nav-bar class="home-nav"><template #center>购物街</template></nav-bar>
    <!-- 狸猫换太子，实现吸附效果 -->
    <tab-control
      :titles="titles"
      @tabClick="tabClick"
      ref="tabControl2"
      v-show="isTabShow"
    />
    <!-- Scroll添加滚动 -->
    <scroll
      class="content"
      ref="scroll"
      :probe-type="3"
      @scroll="contentScroll"
      :pull-up-load="true"
      @pullingUp="loadMore"
      :observeImage="true"
    >
      <!-- 轮播图 -->
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad" :key="home_swiper" />
      <!-- 推荐组件 -->
      <home-recommend-view :recommends="recommends" />
      <!-- 本周流行 -->
      <feature-view />
      <!-- 分类导航栏 -->
      <tab-control
        :titles="titles"
        @tabClick="tabClick"
        ref="tabControl1"
        v-show="!isTabShow"
      />
      <!-- 商品展示 -->
      <goods-list :goods="showGoods" />
    </scroll>
    <!-- 回到顶部按钮 -->
    <back-top @click="backClick" v-show="isBackShow" />
  </div>
</template>

<script>
import NavBar from "@/components/common/navbar/NavBar.vue";
import HomeSwiper from "./childComps/HomeSwiper.vue";
import HomeRecommendView from "./childComps/HomeRecommendView.vue";

import { getHomeMultidata, getHomeGoods } from "@/network/home";
import FeatureView from "./childComps/FeatureView.vue";
import TabControl from "../../components/content/tabControl/TabControl.vue";
import GoodsList from "../../components/content/goods/GoodsList";
import Scroll from "../../components/common/scroll/Scroll.vue";
import {backTopMixin} from '@/common/mixin'

export default {
  name: "Home",
  components: {
    NavBar,
    HomeSwiper,
    HomeRecommendView,
    FeatureView,
    TabControl,
    GoodsList,
    Scroll,
  },
  mixins: [backTopMixin],
  data() {
    return {
      banners: [],
      recommends: [],
      titles: ["流行", "新款", "精选"],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] },
      },
      currentType: "pop",
      tabControlTop: null,
      isTabShow: false,
      saveY: 0,
      home_swiper : 0
    };
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list;
    },
  },
  
  created() {
    // 1.请求多个数据
    this.getHomeMultidata();
    // 2.请求商品数据pop news sell
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  // 回到定位位置
  activated () {
    this.$refs.scroll.scrollTo(0, this.saveY, 0)
  },
  // 记录离开首页此时的高度
  deactivated () {
    this.saveY = this.$refs.scroll.getScrollY()
  },
  mounted() {
    setTimeout(() => {
      this.forceRerender()
    },400)
  },
  methods: {
    /**
     * 关于事件监听的方法
     *
     */
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
          break;
      }
      this.$refs.tabControl1.currentIndex = index
      this.$refs.tabControl2.currentIndex = index
    },
    contentScroll(position){
      // 1.确定BackTop是否显示
      this.isBackShow = (-position.y) > 1000

      // 2.确定TabControl是否吸顶
      this.isTabShow = (-position.y) > this.tabControlTop - 44
    },
    loadMore(){
      this.getHomeGoods(this.currentType)
    },
    swiperImageLoad(){
      this.tabControlTop = this.$refs.tabControl1.$el.offsetTop
    },
    forceRerender() {
      this.home_swiper += 1; 
    },
    /**
     * 关于网络请求的方法
     *
     */
    getHomeMultidata() {
      getHomeMultidata().then((res) => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },

    getHomeGoods(type) {
      const page = this.goods[type].page + 1;
      getHomeGoods(type, page).then((res) => {
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;
        this.$refs.scroll.finishPullUp()
        // console.log(this.goods[type].list);
      });
    },
  },
};
</script>

<style scoped>
.wrapper {
  height: 100vh;
}
.home-nav {
  background-color: var(--color-tint);
  color: #fff;
  font-weight: 700;
  /* 为了在使用原生滚动时，导航栏不随着内容滚动 */
  /* position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 9; */
}
.content {
  overflow: hidden;
  height: calc(100% - 84px);
}
</style>
