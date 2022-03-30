<template>
  <view>
    <!-- 轮播图的区域 -->
    <swiper :indicator-dots="true" :autoplay="true" :interval="3000" :duration="1000" :circular='true'>
      <swiper-item v-for="(item,index) in swiperList" :key='index'>
        <navigator class="swiper-item" :url="'/subpkg/goods_detail/goods_detail?goods_id='+item.goods_id">
          <!-- 响应数据里面有一个image_src属性 -->
          <image :src="item.image_src"></image>
        </navigator>
      </swiper-item>
    </swiper>

    <!-- 分类导航区域 -->
    <view class="nav-list">
      <view class="nav-item" v-for="(item,index) in navList " :key='index' @click="navClickHandler(item)">
        <image :src="item.image_src" class="nav-img"></image>
      </view>
    </view>

    <!-- 楼层区域 -->
    <view class="floor-list">
      <!-- 楼层 item 项 -->
      <view class="floor-item" v-for="(item, index) in floorList" :key="index">
        <!-- 楼层标题 -->
        <image :src="item.floor_title.image_src" class="floor-title"></image>
        <!-- 楼层图片区域 -->
       <view class="floor-image-box">
         <!-- 左侧大图片的盒子 -->
         <navigator class="left-img-box" :url="item.product_list[0].url">
           <!-- 对象语法绑定内联样式 -->
           <!-- 小程序里面mode="widthFix"可以让高度自适应 -->
           <image :src="item.product_list[0].image_src" :style="{width:item.product_list[0].image_width+'rpx'}"
             mode="widthFix"></image>
         </navigator>
         <!-- 右侧4个小图片的盒子 -->
         <view class="right-img-box">
           <!-- 当前这个楼层是item,楼层里的图片是item.product -->
           <navigator class="right-img-item" v-for="(item2, index2) in item.product_list" :key="index2" v-if="index2 !== 0" :url="item2.url">
                <image :src="item2.image_src" mode="widthFix" :style="{width: item2.image_width + 'rpx'}"></image>
              </navigator>
         </view>
       </view>
      </view>
    </view>
  </view>


</template>





<script>
  export default {
    data() {
      return {
        // 轮播图的数据列表，默认为空数组
        swiperList: [],
        // 分类导航的数据列表
        navList: [],
        //楼层的数据
        floorList: []
      }
    },
    onLoad() {
      // 调用获取轮播图数据的方法
      this.getSwiperList()
      // 调用获取分类导航数据的方法
      this.getNavList()
      // 调用获取楼层数据的方法
      this.getFloorList()
    },
    methods: {
      // 获取轮播图数据的方法
      async getSwiperList() {
        // 发起请求 解构出data属性，重命名为res
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/home/swiperdata')
        // 请求失败
        if (res.meta.status !== 200) {
          //如果失败就return出去，同时调用弹框方法
          return uni.$showMsg()
        }
        // 请求成功，为 swiperList 中的数据赋值
        this.swiperList = res.message
        console.log('1');
      },
      // 获取分类导航数据的方法
      async getNavList() {
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/home/catitems')
        if (res.meta.status !== 200) {
          //如果失败就return出去，同时调用弹框方法
          return uni.$showMsg()
        }
        // 请求成功，为 navList 中的数据赋值
        this.navList = res.message
      },

      // 获取楼层数据的方法
      async getFloorList() {
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/home/floordata')
        if (res.meta.status !== 200) {
          //如果失败就return出去，同时调用弹框方法
          return uni.$showMsg()
        }
        
        //由于api提供的要跳转到的页面地址和我们自己创建的地址不一致，所以要对数据进行处理
        res.message.forEach(floor=>{
          floor.product_list.forEach(prod=>{
            //把原来的数据分割出来的有用信息 与我们自己创建的地址进行拼接
            prod.url='/subpkg/goods_list/goods_list?'+prod.navigator_url.split('?')[1]
          })
        })
        // 请求成功，为 floorList 中的数据赋值
        this.floorList = res.message
      },
      navClickHandler(item) {
        if (item.name === '分类') {
          // 调用 api
          uni.switchTab({
            url: '/pages/cate/cate'
          })
        }
      }

    },
  }
</script>

<style lang="scss">
  swiper {
    /* 给轮播图加一个强制的高度 */
    height: 330rpx;

    .swiper-item,
    image {
      width: 100%;
      height: 100%;
    }
  }

  .nav-list {
    display: flex;
    /* 均匀排列每个元素 每个元素周围分配相同的空间 */
    justify-content: space-around;
    margin: 15px 0;

    .nav-img {
      width: 128rpx;
      height: 140rpx;
    }

  }

  .floor-title {
    width: 100%;
    height: 60rpx;
    
  }
  .right-img-box{
    display: flex;
    /* flex-wrap 属性指定 flex 元素单行显示还是多行显示,当属性值为wrap时 规定灵活的项目在必要的时候拆行或拆列。 */
    flex-wrap: wrap;
    /* justify-content 属性设置弹性盒子元素在横轴方向的对齐方式,当属性值为space-around时均匀排列每个元素每个元素周围分配相同的空间 */
    justify-content: space-around;
  }
  .floor-image-box{
    display: flex;
    padding-left: 10rpx;
  }
  
</style>
