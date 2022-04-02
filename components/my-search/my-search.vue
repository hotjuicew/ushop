<template>
  <view class="my-search-container" :style="{'background-color': bgcolor}"  @click="searchBoxHandler">
    <!-- 使用 view 组件模拟 input 输入框的样式 -->
    <view class="my-search-box" :style="{'border-radius':radius+'px'}">
      <!-- 使用uni-ui提供的图标组件 -->
      <uni-icons type="search" size="17">
           
      </uni-icons>
       <text class="placeholder">搜索</text>
    </view>


  </view>
</template>

<script>
  export default {
    name: "my-search",
    props:{
      // 为了增强组件的通用性，我们允许使用者自定义搜索组件的 背景颜色 和 圆角尺寸。
      bgcolor: {
         type: String,
         default: '#C00000'
       },
      radius:{
        type:Number,
        default:18//px
      }
    },
    data() {
      return {

      };
    },
    methods:{
      searchBoxHandler(){
        //外界通过@为我们传递了一个事件，通过this.$emit()来触发这个事件
        this.$emit('myclick')
      }
    }
  }
</script>

<style lang="scss">
.my-search-container {
  height: 50px;
  padding: 0 10px;
  display: flex;
  align-items: center;
}

.my-search-box {
  height: 36px;
  background-color: #ffffff;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;

  .placeholder {
    font-size: 15px;
    margin-left: 5px;
  }
}
</style>
