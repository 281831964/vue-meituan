<!--商店主页-->
<template>
    <div id="store">
        <!--skeleton-screen-loading-->
        <div class="skeleton-loading" v-if="!poi_info.name">
            <img src="../../assets/restaurant.jpg">
        </div>
        <!--头部商家信息-->
        <div class="head-container">
            <!--头部-->
            <v-head :title="poi_info.name" goBack="true" color="#fff" bgColor="#333" more="true"></v-head>
            <!--商家信息-->
            <div class="store-info-wapper">
                <h3>{{poi_info.name}}</h3>
                <div class="store-info">
                    <div class="deliver-info">
                        <span>{{poi_info.min_price_tip}}  | {{poi_info.shipping_fee_tip}} | {{poi_info.delivery_time_tip}}</span>
                        <p><i class="iconfont icon-broadcast">&#xe62d;</i>{{poi_info.bulletin}}</p>
                    </div>
                    <div class="logo">
                        <img :src="poi_info.pic_url">
                    </div>
                </div>
                <!--活动列表-->
                <div class="actives">
                    <ul :style=" 'transform: translateY('+ positionY % discountsLength * -0.9 +'rem)'">
                        <li v-for="(item, index) in poi_info.discounts2" :key="index">
                            <i class="icon"
                               :style="{backgroundImage:'url('+ item.icon_url+')'}"></i>
                            <span>{{item.info}}</span>
                        </li>
                    </ul>
                    <span class="active-number" @click="showStoreDetail();">详情></span>
                </div>
            </div>
        </div>
        <!--导航 有3个路由  点菜 评价 和商家-->
        <div class="nav-background"></div>
        <div class="nav">
            <router-link :to="{path:'/store/menu',query:{id:restaurant_id}}" class="menu" active-class="active">
                <span class="active">点菜</span>
            </router-link>
            <router-link :to="{path:'/store/comment',query:{id:restaurant_id}}" class="comment" active-class="active">
                <span>评价</span>
            </router-link>
            <router-link :to="{path:'/store/seller',query:{id:restaurant_id}}" class="seller" active-class="active">
                <span>商家</span>
            </router-link>
        </div>
        <!--商家详细信息 当点击活动列表右侧的 > 箭头时显示 -->
        <transition name="fade">
            <store-detail
                    class="store-detail"
                    v-show="showDetail"
                    :showFlag.sync="showDetail"
                    :poi_info="poi_info">
            </store-detail>
        </transition>
        <!--点菜 评价 和商家-->
        <keep-alive>
            <router-view></router-view>
        </keep-alive>
    </div>
</template>

<script>

    import StoreDetail from './store_detail.vue'
    import {mapGetters} from 'vuex'

    export default {
        data() {
            return {
                showDetail: false, //商家详情显示
                restaurant_id: 0,   //商店id
                positionY: 0,         //活动滚动
            }
        },
        computed: {
            ...mapGetters(['poi_info']),
            discountsLength() { //打折列表数量
                return this.poi_info.discounts2 ? this.poi_info.discounts2.length : 0;
            }
        },
        methods: {
            //商家详情显示
            showStoreDetail() {
                this.showDetail = true;
            }
        },
        created() {
            //根据路由query获得商店id
            this.restaurant_id = this.$route.query.id;
            //根据商店id获取店家信息
            this.$store.dispatch('getRestaurant', this.restaurant_id);
            //活动列表不停滚动播放
            this.timer = setInterval(() => {
                this.positionY++;
            }, 4000);
        },
        components: {
            'store-detail': StoreDetail
        }
    }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
    @import "../../style/mixin.scss";
    @import "./store.scss";
</style>
