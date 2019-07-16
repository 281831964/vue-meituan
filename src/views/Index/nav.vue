<!--首页nav组件-->
<template>
  <nav>
    <mt-swipe :auto="8000" :show-indicators="true" :speed="3000">
      <mt-swipe-item class="lists" v-for="(item,index) in categoryList" :key="index">
        <router-link class="type" :to="{path:'/category',query:{type:item.name}}" :key="item.id">
          <!--<div class="category-img">-->
            <img :src="item.url" :alt="item.name">
          <!--</div>-->
        </router-link>
      </mt-swipe-item>
    </mt-swipe>
  </nav>
</template>

<script>

  import {Swipe, SwipeItem} from 'mint-ui';
  import {mapGetters} from 'vuex'

  export default {
    data() {
      return {
        categoryList: [
          {
            name: '美食',
            url: 'http://img.zuofan.cn/thumb/s/2018/6/1528441469803231,c_fill,h_180,w_240.jpg',
            type: 'food'
          },
          {
            name: '陈皮排骨',
            url: 'http://www.ym020.com/pic/gif/1219576730.jpg',
            type: 'supermarket'
          },
          {
            name: '生鲜果蔬',
            url: 'http://www.57yx.com/adm/uploads/allimg/151114/14-15111409445U60.png',
            type: 'fruit'
          },
          {
            name: '下午茶',
            url: 'http://img.zuofan.cn/thumb/s/2018/6/1528441469803231,c_fill,h_180,w_240.jpg',
            type: 'tea'
          },
          {
            name: '正餐优选',
            url: 'http://img.zuofan.cn/thumb/s/2018/6/1528441469803231,c_fill,h_180,w_240.jpg',
            type: 'dinner'
          },
          {
            name: '汉堡披萨',
            url: 'http://img.zuofan.cn/thumb/s/2018/6/1528441469803231,c_fill,h_180,w_240.jpg',
            type: 'pizza'
          },
          {
            name: '跑腿代购',
            url: 'http://www.57yx.com/adm/uploads/allimg/151114/14-15111409445U60.png',
            type: 'buyOnSomebody'
          }
        ],
        categoryListSort: []
      }
    },
    components: {
      [Swipe.name]: Swipe,
      [SwipeItem.name]: SwipeItem,
    },
    computed: {
      ...mapGetters(['address'])
    },
    created() {
      // 对列表进行排序为2维数组 用于轮播图 每页轮播图有8个item
      // let resArr = [...this.categoryList];
      // for (let i = 0, j = 0; i < this.categoryList.length; i += 8, j++) {
      //   this.categoryListSort[j] = resArr.splice(0, 8);
      // }
    }
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import "../../style/mixin";
  nav {
    padding-top: 0.1rem;
    @include px2rem(height, 370);
    .lists {
        height: 100%;
        width: 100vw;
        display: inline-block;
        vertical-align: top;
        text-align: center;
        background: #ffffff;
        .type {
          /*width: 25%;*/
          /*@include px2rem(height, 140);*/
          margin: 0.2rem 0;
          display: inline-flex;
          flex-direction: column;
          /*<!--@include px2rem(width, 100);-->*/
          /*<!--@include px2rem(height, 95);-->*/
          img {
            width: 240px;
            height: 180px;
            margin-bottom: 0.3rem;
          }
        }
    }
  }
</style>
