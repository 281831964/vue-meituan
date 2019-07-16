<!--商店底部购物栏-->
<template>
  <div id="bottom">
    <!--购物车详细列表-->
    <transition name="fade">
      <article class="cart-list-container" v-show="cartDetail">
        <section class="head">
          <span>餐盒费0元</span>
          <span><i class="iconfont">&#xe615;</i><span @click="emptyCart()">清空购物车</span></span>
        </section>
        <section class="cart-list" v-for="item in  restaurantCartList" :key="item.id">
          <span>{{item.name}} </span>
          <span class="price">￥{{ Number((item.price * item.num).toFixed(2))}}</span>
          <selector :name="item.name" :food_id="item.id" :price="item.price" style="bottom:0.3rem;"></selector>
        </section>
      </article>
    </transition>
    <!--底部bar-->
    <div class="bottom-bar">
      <div class="left" @click="showCartDetail();">
        <span class="shopping-cart" :class="{cartActive:totalPrice>0,ballInCart:ballInCart}" ref="iconCartContainer">
          <i class="iconfont icon-cart">&#xe66b;</i>
          <span class="food-num" v-if="foodNum">{{foodNum}}</span>
        </span>
        <div class="price-container">
          <span class="total-price" v-if="totalPrice">￥{{totalPrice}}</span>
          <span class="deliver-fee" v-if="!shipping_fee">免配送费</span>
          <span class="deliver-fee" v-else>另需要配送费￥{{shipping_fee}}</span>
        </div>
      </div>
      <span class="submit" v-if="!totalPrice">{{min_price_tip}}</span>
      <span class="submit" v-else-if="totalPrice < min_price">还差{{min_price - totalPrice}}</span>
      <span @click="prepareOrder()" class="submit go-buy" v-else>去结算</span>
    </div>
    <transition name="fade">
      <div class="shade" v-show="cartDetail" @click="cartDetail=false;"></div>
    </transition>
  </div>
</template>

<script>
  import {mapGetters} from 'vuex'
  import {getInfo} from '@/utils/auth'
  import selector from '@/components/selector'
  export default {
    data() {
      return {
        cartDetail: false,
        restaurant_id: 0,
        shipping_fee: 0,
        shipping_fee_tip: 0,
        min_price: 0,
        min_price_tip: '￥0起送',
      }
    },
    computed: {
      ...mapGetters([
        'poi_info', 'cartList', 'ballInCart'
      ]),
      totalPrice() {  //计算购物车总价格
        return this.cartList[this.restaurant_id] ? this.cartList[this.restaurant_id].totalPrice : 0;
      },
      foodNum() {    //商品数量
        return this.cartList[this.restaurant_id] ? this.cartList[this.restaurant_id].totalNum : 0;
      },
      food_spu_tags() {
        return this.restaurant_info.food_spu_tags;
      },
      restaurantCartList() { //当前商店购物车商品
        let lists = this.cartList[this.restaurant_id];
        let p;
        let arr = [];
        for (p in lists) {
          if (Number(p))
            arr.push(lists[p]);
        }
        return arr;
      }
    },
    methods: {
      showCartDetail() {   //点击底部显示购物车详情
        if (this.cartDetail)    //如果当前是显示状态 再次点击为隐藏
          this.cartDetail = false;
        else      //如果当前是关闭状态   判断购物车有没有商品 如果有才能显示详情
          this.cartDetail = this.totalPrice > 0;
      },
      prepareOrder() {    //准备下订单
        let data = {
          restaurant_id: this.restaurant_id,
          foods: this.cartList[this.restaurant_id]
        };

        if(getInfo){
          localStorage.setItem('confirmOrderData', JSON.stringify(data));
          this.$router.push({path: '/confirm_order'})
        }else{
          this.$router.push({path:'/login'})
        }

      },
      emptyCart() {
        this.$store.dispatch('emptyCart', {restaurant_id: this.restaurant_id});
        this.cartDetail = false;
      }
    },
    created() {
      this.restaurant_id = this.$route.query.id;
    },
    mounted() {
      let _this = this
      this.$refs['iconCartContainer'].addEventListener('webkitAnimationEnd', () => {
        _this.$store.dispatch('ballInCart', false);       //触发底部小车晃动
      })
    },
    watch: {
      totalPrice(val) { //watch totalPrice 如果为0 就隐藏购物车底部点击后显示出来的商品
        if (val === 0)
          this.cartDetail = false;
      },
      poi_info(val) {  //监听vuex的reataurant_info信息
        this.shipping_fee = val.shipping_fee;   //配送信息
        this.min_price = val.min_price          //最低价起送信息
        this.min_price_tip = val.min_price_tip
        this.shipping_fee_tip = val.shipping_fee_tip;
      }
    },
    components: {
      selector
    }
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import "../../../style/mixin";
  @import "./bottom.scss";
</style>
