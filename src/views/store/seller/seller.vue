<!--商家信息-->
<template>
  <div class="seller-info">
    <!--地址信息-->
    <div class="position container">
      <span class="location-icon icon"><i class="iconfont">&#xe605;</i></span>
      <p class="address describe">{{poi_info.address}}</p>
      <span class="phone-icon icon"><i class="iconfont">&#xe620;</i></span>
    </div>
    <div class="line-container">
      <div class="line"></div>
    </div>
    <!--安全档案-->
    <div class="safe-file container">
      <span class="safe-icon icon"><i class="iconfont">&#xe67a;</i></span>
      <span class="describe">查看食品安全档案</span>
      <span class="entry-icon icon"><i class="iconfont">&#xe63f;</i></span>
    </div>
    <!--配送服务-->
    <div class="delivery-info container">
      <span class="icon"><i class="iconfont">&#xe603;</i></span>
      <span class="describe">配送服务:由 商家 提供配送服务</span>
    </div>
    <div class="line-container">
      <div class="line"></div>
    </div>
    <!--配送时间-->
    <div class="delivery-time container">
      <span class="icon"><i class="iconfont">&#xe62f;</i></span>
      <span class="describe">配送时间: {{poi_info.shopping_time_start}} - {{poi_info.shopping_time_end}}</span>
    </div>
    <!--公告-->
    <div class="notification container" v-if="poi_info.bulletin">
      <span class="icon"><i class="iconfont">&#xe6aa;</i></span>
      <span class="describe ellipsis">{{poi_info.bulletin}}</span>
      <span class="entry-icon icon"><i class="iconfont">&#xe63f;</i></span>
    </div>
    <div class="line-container">
      <div class="line"></div>
    </div>
    <!--商家服务-->
    <div class="poi-service container">
      <span class="icon"><i class="iconfont">&#xe632;</i></span>
      <span class="service">商家服务</span>
      <span v-for="(service,index) in sellerInfo.poi_service" :key="index">
        <i class="icon-service" :style="{backgroundImage:'url('+ service.icon +')'}"></i>
        <span>{{service.content}}</span>
      </span>
    </div>
    <div class="line-container">
      <div class="line"></div>
    </div>
    <div class="discount">
      <ul>
        <li v-for="(discount,index) in poi_info.discounts2" :key="index">
          <span class="active-icon" :style="{backgroundImage:'url('+ discount.icon_url +')'}"></span>
          <span>{{discount.info}}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  import {mapGetters} from 'vuex'

  export default {
    data() {
      return {
        sellerInfo: []
      }
    },
    computed: {
      ...mapGetters(['poi_info'])
    }
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import "../../../style/mixin.scss";
  @import "./seller.scss";

</style>
