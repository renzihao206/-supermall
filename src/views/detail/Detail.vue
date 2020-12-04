<template>
<div id="detail">
<detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
 <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll">
   <detail-swiper :top-images="topImages"/>
   <detail-base-info :goods="goods"></detail-base-info>
   <detail-shop-info :shop="shop"></detail-shop-info>
   <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"></detail-goods-info>
   <detail-param-info :param-info="paramInfo" ref="params"></detail-param-info>
   <detail-comment-info :comment-info="commentInfo" ref="comment"></detail-comment-info>
   <goods-list :goods="recommends" ref="recommend"/>
 </scroll>
  <detail-bottom-bar @addCart="addToCart"/>
  <back-top @click.native="backClick" v-show="isShowBackTop" ></back-top>
<!--  <toast :message="message" :show="show"/>-->
</div>

</template>

<script>
import DetailNavBar from "@/views/detail/childComps/DetailNavBar";
import DetailSwiper from "@/views/detail/childComps/DetailSwiper";
import DetailBaseInfo from "@/views/detail/childComps/DetailBaseInfo";
import DetailShopInfo from "@/views/detail/childComps/DetailShopInfo";
import DetailGoodsInfo from "@/views/detail/childComps/DetailGoodsInfo";
import DetailParamInfo from "@/views/detail/childComps/DetailParamInfo";
import DetailCommentInfo from "@/views/detail/childComps/DetailCommentInfo";
import DetailBottomBar from "@/views/detail/childComps/DetailBottomBar";

import Scroll from "@/components/common/scroll/Scroll";
import GoodsListItem from "@/components/content/goods/GoodsListItem";
// import Toast from "@/components/common/toast/Toast";

import {getDetail,Goods,Shop,GoodsParam,getRecommend} from "@/network/detail";
import GoodsList from "@/components/content/goods/GoodsList";
import {debounce} from "@/common/utils";
import {itemListenerMixin,backTopMixin} from "@/common/mixin"
import {mapActions} from 'vuex'



export default {
  name: "Detail",
  components:{
    GoodsList,
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    Scroll,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    GoodsListItem,
    DetailBottomBar,
    // Toast
  },
  mixins:[itemListenerMixin,backTopMixin],
  data(){
    return {
      iid:null,
      topImages:[],
      goods:{},
      shop:{},
      detailInfo:{},
      paramInfo:{},
      commentInfo:{},
      recommends:[],
      itemImgListener:null,
      themeTopYs:[],
      getThemeYopY:null,
      currentIndex:0,
      // message:'',
      // show:false,
    }
  },
  created() {
    //1.保存传入的iid
    this.iid = this.$route.params.iid

    //2.根据iid请求数据
    getDetail(this.iid).then(res => {
        //1.获取顶部的图片轮播数据
      const data =res.result
      this.topImages = data.itemInfo.topImages
        //2.获取商品信息
      this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)
        //3.获取店铺信息对象
      this.shop = new Shop(data.shopInfo,)
        //4.获取商品详细数据
      this.detailInfo = data.detailInfo
        //5.获取参数信息
      this.paramInfo = new  GoodsParam(data.itemParams.info,data.itemParams.rule)
        //6.取出评论信息
       if(data.rate.cRate !==0){
        this.commentInfo = data.rate.list[0]
          this.$nextTick(() => {
            //根据最新数据，对应的dom是已经被渲染出来
            //但是图片依然是没有加载完(目前获取到offsetTop不包含其中的图片)
            // offsetTop值不对，都是因为图片的问题
            // this.themeTopYs=[]
            // this.themeTopYs.push(0);
            // this.themeTopYs.push(this.$refs.params.$el.offsetTop);
            // this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
            // this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);
            // console.log(this.themeTopYs);
           this.getThemeYopY= debounce(() =>{
             this.themeTopYs=[]
             this.themeTopYs.push(0);
             this.themeTopYs.push(this.$refs.params.$el.offsetTop);
             this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
             this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);

           },200)
})
      }
    })
    //3.请求推荐数据
    getRecommend().then(res =>{
     this.recommends  = res.data.list
    })
  },
  methods:{
    ...mapActions(['addCart']),
    imageLoad(){
      this.$refs.scroll.refresh()
      this.getThemeYopY()
    },
    titleClick(index){
     this.$refs.scroll.scrollTo(0,-this.themeTopYs[index],100)
    },
    contentScroll(position){
      // console.log(position);
      //1.获取y值
      const positionY = -position.y
      //2.positionY和主题中的值进行对比
      let length = this.themeTopYs.length
     for(let i=0;i<length;i++){
       if(this.currentIndex !== i&&((i<length-1 && positionY >= this.themeTopYs[i] && positionY <
         this.themeTopYs[i+1]) || (i === length-1 && positionY >= this.themeTopYs[i]))){
         this.currentIndex = i;
         this.$refs.nav.currentIndex = this.currentIndex
       }
     }
      this.isShowBackTop=-(position.y)>1000
    },
    addToCart(){
      //1.获取购物车需要展示商品信息
           const product = {}
            product.image = this.topImages[0]
            product.title = this.goods.title;
            product.desc = this.goods.desc;
            product.price = this.goods.realPrice;
            product.iid = this.iid;
          //2.将商品添加到购物车
      this.addCart(product).then(res => {
        // this.show = true
        // this.message = res
        // setTimeout(() =>{
        // this.show = false;
        // this.message = ''
        //     },1500)
        this.$toast.show(res,1000)
      })
       // this.$store.dispatch('addCart',product).then(res => {
       //   console.log(res);
       // })
    }
  },
  mounted(){
  },
  updated(){
  },
  destroyed() {
    this.$bus.$off('itemImageLoad',  this.itemImgListener)
  }
}
</script>
<style scoped>
#detail{
  position:relative;
  z-index:9;
  background-color: #ffffff;
  height:100vh;
}
.content{
  height: calc(100% - 44px - 49px );
}
.detail-nav{
  position:relative;
  z-index:9;
  background-color: #ffffff;
}
</style>

