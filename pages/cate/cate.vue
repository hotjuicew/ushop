<template>
  <view>
    <view class="scroll-view-container">
      <!-- 左侧 的滑动区域 -->
      <scroll-view class="left-scroll-view" scroll-y="true" :style="{height: wh+'px'}">
        <block v-for="(item,index) in cateList" :key="index">
          <view :class="['left-scroll-view-item', index === active ? 'active' : '']" @click="activeChanged(index)">
            {{item.cat_name}}
          </view>
        </block>
      </scroll-view>
      <!-- 右侧的滑动区域 -->
      <!-- scrollTop是小程序中的属性 是这个元素的内容顶部（卷起来的）到它的视口可见内容（的顶部）的距离的度量 -->
      <scroll-view class="right-scroll-view" scroll-y="true" :style="{height: wh+'px'}" scrollTop="scrollTop">
        <view class="cate-lv2" v-for='(item2,i2) in cateLevel2' :key='i2'>
          <!-- 二级分类的标题 -->
          <view class="cate-lv2-title">/{{item2.cat_name}}/</view>
          <!-- 三级分类列表 -->
          <view class="cate-lv3-list">
            <!-- 三级分类项 -->
            <view class="cate-lv3-item" v-for="(item3,i3) in item2.children" :key='i3' @click="gotoGoodsList(item3)">
              <image :src="item3.cat_icon"></image>
              <text>{{item3.cat_name}}</text>
            </view>
          </view>
        </view>

      </scroll-view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        // 当前设备可用的高度
        wh: 0,
        cateList: [],
        active: 0, //初始值默认第一项为激活项
        // 二级分类列表
        cateLevel2: [],
        // 滚动条距离顶部的距离
        scrollTop: 0,
      };
    },
    onLoad() {
      // 这个api是可以获取当前设备信息的
      const sysInf = uni.getSystemInfoSync()
      // 可使用的窗口高度
      this.wh = sysInf.windowHeight
      // 小程序method方法和onload方法里面this指向的都是当前page对象
      // 获取分类列表数据
      this.getCateList()
    },
    methods: {
      // 获取分类列表数据的方法
      async getCateList() {
        // 发起请求
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/categories')
        // 判断是否获取失败
        if (res.meta.status !== 200) return uni.$showMsg()
        // 转存数据
        this.cateList = res.message
        //为二级分类赋值
        this.cateLevel2 = res.message[0].children
      },
      activeChanged(index) {
        this.active = index
        //重新为二级分类赋值
        this.cateLevel2 = this.cateList[index].children
        
        //使右侧区域重新回到顶部
       // 由于刚开始的时候 它的值是=0的,用户点击切换后又重新把值改成0,值是一样的,就不会进行滚动条备选重置
       //为解决上述问题 让 scrollTop 的值在 0 与 0.1之间切换 虽然会有一点点像素的偏差 但是用户感觉不出来
        this.scrollTop=this.scrollTop===0?0.1:0
      },
      //跳转到商品列表页面
      gotoGoodsList(item3){
        // .navigateTo 是从当前页面，跳转到应用内的某个指定页面的API
        uni.navigateTo({
          //一个问题：为什么取名为cid？
          url:'/subpkg/goods_list/goods_list?cid=' + item3.cat_id
        })
      }


    }
  }
</script>

<style lang="scss">
  .scroll-view-container {
    /* 让上下两个滑动区域左右布局 */
    display: flex;

    .left-scroll-view {
      width: 120px;

      .left-scroll-view-item {
        line-height: 60px;
        background-color: #f7f7f7;
        text-align: center;
        font-size: 12px;

        /* 激活项的样式 */
        // 指一个元素既包含left-scroll-view-item类名又包含active类名
        &.active {
          background-color: #ffffff;
          position: relative;

          &::before {
            content: ' ';
            display: block;
            width: 3px;
            height: 30px;
            background-color: #c00000;
            position: absolute;
            left: 0;
            top: 50%;
            /* 往回走50%的高度 */
            transform: translateY(-50%);
          }
        }
      }
    }
  }

  .cate-lv2-title {
    font-size: 12px;
    font-weight: bold;
    text-align: center;
    padding: 15px 0;
  }

  .cate-lv3-list {
    display: flex;
    /* 盛不下自动换行 */
    flex-wrap: wrap;

    .cate-lv3-item {
      width: 33.33%;
      margin-bottom: 10px;
      display: flex;
      flex-direction: column;
      /* 使文本和图片纵向布局 */
      align-items: center;

      /* 对齐弹性容器中的弹性项目 */
      image {
        width: 60px;
        height: 60px;
      }

      text {
        font-size: 12px;

      }
    }
  }
</style>
