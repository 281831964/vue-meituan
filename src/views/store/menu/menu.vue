<!--菜单-->
<template>
  <div id="menu">
    <div class="left" ref="left">
      <ul>
        <li v-for="(category,index) in foodsData" :key="category.id" @click="mappingScroll(index)"
            :class="{'activity-menu': index == menuIndex}">
          {{category.name}}
        </li>
      </ul>
    </div>
    <div class="right" ref="right">
      <article ref="categorys">
        <section v-for="category in foodsData" :key="category.id">
          <h2>{{category.name}}</h2>
          <article>
            <section v-for="spus in category.spus" :key="spus.id">
              <div class="img">
                <img :xsrc="spus.pic_url" src="../../../assets/shoploading.png">
              </div>
              <div class="info">
                <div class="name">{{spus.name}}</div>
                <div class="sell-num">月售 {{spus.month_saled_content}}</div>
                <span class="price"> ￥{{spus.skus[0].price}}</span>
              </div>
              <selector @showDot="showDotFun" :name="spus.name" :food_id="spus.skus[0].id" :price="spus.skus[0].price" :pic="spus.pic_url"></selector>
            </section>
          </article>
        </section>
      </article>
    </div>
    <transition appear @after-appear='afterEnter' @before-appear="beforeEnter" v-for="(ball,index) in balls" :key="index">
      <div class="balls" ref="balls" v-if="ball">
        <span class="ball"></span>
      </div>
    </transition>
    <Bottom v-if="getInfoReady"></Bottom>
  </div>
</template>

<script>
  import BScroll from 'better-scroll'
  import {getFoods} from '@/api/restaurant'
  import Bottom from './bottom'
  import selector from '@/components/selector'

  export default {
    data() {
      return {
        categoryPosition: [],    //存放各个分类的categoryPosition 为了点击左侧让右侧对应滚动
        menuIndex: 0,   //左侧当前是第几个分类
        balls: [],      //加入购物车时的小球
        elRight: 0, //当前点击加按钮在网页中的绝对top值
        elBottom: 0, //当前点击加按钮在网页中的绝对left值
        fontSize: 0,      //用来获取当前1rem等于多少px 因为小球用transition运动 需要用到px 而左下角购物车的位置是rem为单位 需要转换成px 才能让小球找到落地点
        foodsData: [],       //食物数据
        getInfoReady: false,
        hasLoadArr: []    //用来保存已经点击过的分类 如果点击过就不用再去判断懒加载
      }
    },
    methods: {
      mappingScrolli(pos) {   //右侧滚动时  判断当前左侧是第几个分类
        for (let i = 0; i < this.categoryPosition.length; i++) {
          if (this.categoryPosition[i] <= Math.ceil(Math.abs(pos.y)) && i === this.categoryPosition.length - 1 || this.categoryPosition[i + 1] > Math.ceil(Math.abs(pos.y))) {
            this.menuIndex = i;
            this.lazyLoadImg(i);
            this.lazyLoadImg(i + 1);  //下一页提前加载
            break;
          }
        }
      },
      mappingScroll(index) {    //点击左侧  右侧滚动到对应的位置
        this.rightScroll.off('scroll', this.mappingScrolli)
        this.rightScroll.scrollTo(0, -this.categoryPosition[index], 500);
        this.menuIndex = index;
        this.lazyLoadImg(index);
        setTimeout(this.listenScroll, 500);
      },
      listenScroll() {  //监听滚动事件
        this.rightScroll.on('scroll', this.mappingScrolli)
      },
      showDotFun(elRight, elBottom) { //触发购物车小球
        this.balls.push(true)
        this.elRight = elRight;
        this.elBottom = elBottom;
      },
      beforeEnter(el) {
        let _this = this;
        this.fontSize = document.documentElement.style.fontSize.split("px")[0];
        el.style.transform = `translate3d(${this.elRight - this.fontSize * 1}px,${this.elBottom - this.fontSize * 1}px,0)`
        el.children[0].addEventListener('transitionend', function () {
          _this.balls.splice(0, 1);
          _this.$store.dispatch('ballInCart', true);
        })
      },
      afterEnter(el) {
        el.style.transform = `translate3d(${this.elRight - this.fontSize * 1}px,${window.innerHeight - 50}px,0)`
        el.children[0].style.transform = `translate3d(-${this.elRight - this.fontSize * 2}px,0px,0)`
        el.style.transition = 'transform .55s cubic-bezier(0.29,-0.25, 0.79,-0.14)';
        el.children[0].style.transition = 'transform .55s linear';
      },
      lazyLoadImg(index) {
        if (index > this.categoryPosition.length - 1 || this.hasLoadArr.indexOf(index) !== -1)    //如果该分类已经调用过懒加载，就不用执行下面函数了
          return;
        this.hasLoadArr.push(index)
        let imgLists = this.$refs.categorys.children[index].querySelectorAll('.img');
        imgLists = Array.from(imgLists)
        imgLists.forEach((el) => {
          let img = el.childNodes[0];
          let xsrc = img.getAttribute('xsrc');
          if (img.src === xsrc)
            return
          img.src = xsrc;
        })
      }
    },
    created() {
      this.windowHeight = window.innerHeight;       //设备显示高度
      let restaurant_id = this.$route.query.id;
      //根据餐馆id获取食物
      getFoods({restaurant_id}).then((response) => {
        this.foodsData = response.data.data;
        this.getInfoReady = true;
        this.$nextTick(() => {       //初始化better-scroll
          this.leftScroll = new BScroll(this.$refs.left, {click: true});
          this.rightScroll = new BScroll(this.$refs.right, {click: true, probeType: 3,});
          let dom = (this.$refs.right.children)[0];   //确定各分类categoryPosition
          let listsArr = Array.from(dom.childNodes);
          listsArr.forEach((item, index) => {
            this.categoryPosition[index] = item.offsetTop;
          });
          this.listenScroll();    //监听滚动
          this.lazyLoadImg(0);  //第一个分类首先加载图片  其它监听活动时懒加载
        })
      })
    },
    components: {
      Bottom,
      selector
    },
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import "../../../style/mixin.scss";
  @import "./menu.scss";
</style>
