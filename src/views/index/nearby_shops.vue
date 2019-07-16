<!--附近商家组件-->
<template>
  <div class="nearby-shops">
    <mt-navbar v-model="active">
      <mt-tab-item id="tab-container1">最实惠套餐</mt-tab-item>
      <mt-tab-item id="tab-container2">最近美食</mt-tab-item>
      <mt-tab-item id="tab-container3">最火点餐</mt-tab-item>
      <mt-tab-item id="tab-container4">最新菜品</mt-tab-item>
    </mt-navbar>
    <mt-tab-container v-model="active">
      <mt-tab-container-item id="tab-container1">
        <article>
          <router-link v-for="(item) in shopLists" :to="{path:'store',query:{id:item.id}}" :key="item.id" tag="section">
            <div class="img-show">
              <img :src="'http://img.zuofan.cn/thumb/s/2018/6/1528441469803231,c_fill,h_180,w_240.jpg'">
            </div>
            <div class="detail">
              <h4>{{item.name}}</h4>
              <div class="shops-message">
                <v-star :score="item.wm_poi_score"></v-star>
                <span class="sell-num">{{item.month_sales_tip}}</span>
                <div class="delivery-info">
                  <span class="deliver-time">{{item.delivery_time_tip}}/</span>
                  <span class="distance">{{item.distance}}</span>
                </div>
              </div>
              <div class="price-message">
                <span>{{item.min_price_tip}} | </span>
                <span>{{item.shipping_fee_tip}} | </span>
                <span>{{item.average_price_tip}}</span>
              </div>
              <div class="active-message">
                <ul>
                  <li v-for="(discount,index) in item.discounts2.slice(0, 1)" :key="index">
                    <div class="discount-left">
                      <img :src="discount.icon_url" class="icon">
                      <span class="info">{{discount.info}}</span>
                    </div>
                    <router-link to="/shops" v-if="index === 0">
                      <i class="iconfont icon-entry">&#xe645;</i>
                    </router-link>
                  </li>
                </ul>
              </div>
            </div>
          </router-link>
        </article>
      </mt-tab-container-item>
      <mt-tab-container-item id="tab-container2"></mt-tab-container-item>
      <mt-tab-container-item id="tab-container3"></mt-tab-container-item>
    </mt-tab-container>
    <!--加载更多-->
    <div class="loading-wrap" ref="loading">
      <span class="loading" v-show="loading && !noMore">正在努力加载中…</span>
      <span class="no-more" v-show="noMore">已经到底了</span>
    </div>
  </div>
</template>

<script>
  import BScroll from 'better-scroll'
  import {getRestaurants} from '@/api/restaurant'
  import {mapGetters} from 'vuex'
  import {TabContainer,TabContainerItem,Navbar,TabItem} from 'mint-ui'

  export default {
    data() {
      return {
        shopLists: [],         //商家列表
        showSort: false,      //显示选择排序列表
        BScrollEvent: null,   //better-scroll实例
        loading: false,       //加载更多
        page: 1,               //当前餐馆列表加载到第几页
        limit: 4,              //每次拉去的餐馆数量
        noMore: false,        //没有更多数据了
        preventRepeat: false,   //避免重复请求
        active:'tab-container1'
      }
    },
    components: {
      [TabContainer.name]: TabContainer,
      [TabContainerItem.name]: TabContainerItem,
      [TabItem.name]: TabItem,
      [Navbar.name]: Navbar,
    },
    computed: {
      ...mapGetters(['address'])
    },
    props: ['scrollWrapper'], //进行better-scroll的DOM对象 餐馆类型 是否地位好准备开始拉去餐馆数据
    methods: {
      //监听better-scroll滚动事件  判断是否滑动到底部 加载更多
      listenScroll() {
        let _this = this;
        _this.BScrollEvent.on('scroll', function (obj) {
          //如果到达底部  请求加载更多数据
          if (Math.abs(obj.y) + _this.scrollWrapper.clientHeight >= _this.scrollWrapper.childNodes[0].clientHeight - 30) {
            if (!_this.loading) {   //避免加载过程中 重复请求
              _this.loading = true;
              //请求加载更多
              _this.getRestaurants(_this.page, _this.limit, function (data) {
                _this.page++;
                data.forEach((el) => {
                  _this.shopLists.push(el);
                });
                //DOM重新渲染完毕后 重新计算better-scroll
                _this.$nextTick(() => {
                  _this.loading = false;
                  _this.BScrollEvent.refresh();
                })
              })
            }
          }
        })
      },
      getRestaurants(page, limit, callback) { //获取餐馆列表
        if (this.noMore || this.preventRepeat)
          return;
        this.preventRepeat = true;
        let offset = (page - 1) * limit;
        let {lat, lng} = this.address;
        getRestaurants({offset, limit, lng, lat}).then((response) => {
          let data = response.data.data;
          this.preventRepeat = false;
          this.noMore = data.length < this.limit;
          callback(data);
        });
      },
      firstFetch() {
        let _this = this;
        //获取餐馆列表
        this.page = 1;
        this.getRestaurants(this.page, this.limit, function (data) {
          _this.page++;
          _this.shopLists = data;
          _this.$nextTick(() => {
            //dom渲染完成 初始化better-scroll
            _this.BScrollEvent = new BScroll(_this.scrollWrapper, {click: true, probeType: 2});
            //监听scroll事件
            _this.listenScroll();
          })
        })
      }
    },
    created() {
      // let {lat, lng} = this.address;
      // if (lat && lng) {//根据 定位信息判断是否是需要重新定位
        this.shopLists = [];
        this.firstFetch();
      // } else {
      //   this.$store.dispatch('location');
      // }
    },
    watch: {
      address(value) {    //地址发生变化，重新获取商家
        this.noMore = false;
        this.preventRepeat = false;
        let {lat, lng} = value;
        if (lat && lng) {
          this.shopLists = [];
          this.firstFetch();
        }
      }
    },
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import "../../style/mixin.scss";
  /*附近商家*/
  .nearby-shops {
    margin: 0.1rem 0;
    /*选择排序功能样式*/
    nav {
      border-bottom: 1px solid $bottomLine;
      position: relative;
      @include px2rem(line-height, 80);
      ul {
        display: flex;
        li {
          font-size: 0.4rem;
          text-align: center;
          padding: 0 0.1rem;
          flex: 1;
          .icon-sort {
            display: inline-block;
            @include px2rem(width, 20);
            font-size: 0.3rem;
          }
        }
      }
    }
    article {
      position: relative;
      section {
        border-radius: 10px;
        display: flex;
        padding: 0.3rem;
        margin: 0.2rem;
        background: #ffffff;
        /*border-bottom: 1px solid $mtGrey;*/
        .img-show {
          @include px2rem(width, 170);
          @include px2rem(height, 130);
          margin-right: 0.2rem;
          text-align: center;
          img {
            width: 100%;
            height: 100%;
          }
        }
        .detail {
          flex: 1;
          h4 {
            font-size: 0.45rem;
            font-weight: bold;
          }
          .shops-message {
            display: flex;
            margin-top: 0.3rem;
            align-items: center;
            span {
              display: inline-block;
              vertical-align: bottom;
              font-size: 0.3rem;
            }
            .sell-num {
              flex: 1;
            }
            .delivery-info {
              display: flex;
              align-items: center;
            }
          }
          .price-message {
            margin: 0.2rem 0;
            display: flex;
            align-items: center;
            span {
              font-size: 0.2rem;
            }
          }

          .active-message {
            ul {
              li {
                display: flex;
                justify-content: space-between;
                align-items: center;
                .discount-left {
                  display: flex;
                  margin: 0.1rem 0;
                  align-items: center;
                  .info {
                    color: #777272;
                    font-size: 0.3rem;
                    display: inline-block;
                    @include px2rem(width, 360);
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                  }
                  .icon {
                    margin-right: 0.15rem;
                    display: inline-block;
                    @include px2rem(width, 34);
                    @include px2rem(height, 34);
                  }
                }
                .icon-entry {
                  font-size: 0.3rem;
                  float: right;
                  margin-right: 0.2rem;
                }
              }
            }
          }
        }
      }
      .shade {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        background: rgba(47, 43, 43, 0.8);
      }
    }

    /*loading部分*/
    .loading-wrap {
      @include loading;
    }
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity 0.5s;
  }

  .fade-enter, .fade-leave {
    opacity: 0
  }
</style>
