<!--商店评论-->
<template>
  <div id="comment">
    <!--评分部分-->
    <div ref="comment" class="scroll-container">
      <article>
        <div class="comment-score-container">
          <div>
            <span class="comment-score">{{poi_info.wm_poi_score}}</span>
            <h3>商家评分</h3>
          </div>
          <div>
            <div>
              <span>口味</span>
              <v-star :score="poi_info.food_score"></v-star>
              <span class="food-score">{{poi_info.food_score}}</span>
            </div>
            <div>
              <span>包装</span>
              <v-star :score="poi_info.pack_score"></v-star>
              <span class="pack-score">{{poi_info.pack_score}}</span>
            </div>
          </div>
          <div>
            <span class="delivery-score">{{poi_info.delivery_score}}</span>
            <h3>配送评分</h3>
          </div>
        </div>
        <!--评分类型部分-->
        <ul class="comment-score-type-info">
          <li class="active">全部</li>
          <li>有图</li>
          <li><i class="iconfont">&#xe741;</i> 点评(0.0分)</li>
        </ul>

        <ul class="comment-score-type-tip">
          <li v-for="tip in commentData.comment_score_type_infos" :key="tip.comment_score_title">
            {{tip.comment_score_title}} {{tip.total_count}}
          </li>
          <li v-for="tip in commentData.labels" :key="tip.label_id">{{tip.content}} {{tip.label_count}}</li>
        </ul>

        <!--评价部分-->
        <article class="comments-container">
          <section v-for="item in commentData" :key="item.id">
            <div class="user-pic-url">
              <img :src="item.avatar">
            </div>
            <div class="comment-main-part">
              <div>
                <span class="user-name">{{item.user_name}}</span>
                <span class="comment-time">{{item.comment_time.slice(0, 10)}}</span>
              </div>
              <div class="order-comment-score"><span>评分 <v-star :score="item.food_score"></v-star></span>
              </div>
              <p class="comment">{{item.comment_data}}</p>
              <div class="comment-pic">
                <div v-for="(pic,index) in  item.pic_url" @click="show_big_pic_event(pic)" :key="index">
                  <img :src="pic">
                </div>
              </div>
              <div class="poi-reply-contents-container" v-if="item.add_comment_list">
              </div>
            </div>
          </section>
        </article>
        <!--加载更多-->
        <div class="loading-container" ref="loading">
          <span class="loading" v-show="loading && !noMore">正在努力加载中…</span>
          <span class="no-more" v-show="noMore">已经到底了</span>
        </div>
      </article>
    </div>
    <!--图片大图-->
    <transition>
      <div class="show-big-pic" v-show="show_big_pic" @click="show_big_pic = false;">
        <div>
          <img :src="big_pic_url">
        </div>
      </div>
    </transition>

  </div>
</template>

<script>
  import BScroll from 'better-scroll'
  import {restaurantComment, getRestaurant} from '@/api/restaurant'

  export default {
    data() {
      return {
        commentData: [],
        comment_score_type: {0: '全部', 1: '好评', 5: '有图好评'},
        big_pic_url: '',   //大图url
        show_big_pic: false,  //显示大图
        poi_info: {},            //商店信息
        noMore: false,          //没有更多
        loading: false,
        offset: 0,              //跳过几页
        BScrollEvent: null,   //better-scroll实例
      }
    },
    mounted() {
      let _this = this;
      _this.restaurant_id = _this.$route.query.id;
      _this.fetchComment((response) => {
        _this.commentData = response.data.data;
        _this.$nextTick(() => {       //初始化better-scroll
          _this.BScrollEvent = new BScroll(_this.$refs.comment, {click: true, probeType: 2});
          //监听scroll事件
          _this.listenScroll();
        })
      })
      //根据商店id获取店家信息
      getRestaurant({restaurant_id: _this.restaurant_id}).then((response) => {
        _this.poi_info = response.data.data;
      })
    },
    methods: {
      fetchComment(callback) {   //获取评论
        restaurantComment({restaurant_id: this.restaurant_id, offset: this.offset, limit: 5}).then((response) => {
          callback(response)
        })
      },
      date_format(time_stamp) {    //因为后端数据是时间戳 这里需要进行格式化
        let date = new Date(time_stamp * 1000);
        return date.getFullYear() + '.' + date.getMonth() + '.' + date.getDay()
      },
      show_big_pic_event(url) {     //显示大图
        this.big_pic_url = url;
        this.show_big_pic = true;
      },
      match_topic(string) {         //匹配tag
        let re = /(#[^#]*#)(.+)/g
        let match = re.exec(string);
        return !match ? string : `<strong style="color:#576b95; font-weight: normal">${match[1]}</strong>${match[2]}`
      },
      //监听better-scroll滚动事件  判断是否滑动到底部 加载更多
      listenScroll() {
        let _this = this;
        let commmentDOM = _this.$refs.comment;
        _this.BScrollEvent.on('scroll', function (obj) {
          //如果到达底部  请求加载更多数据
          if (Math.abs(obj.y) + commmentDOM.clientHeight >= commmentDOM.childNodes[0].clientHeight - 30) {
            if (!_this.loading) {   //避免加载过程中 重复请求
              _this.loading = true;
              //请求加载更多
              _this.fetchComment(function (response) {
                if (!response.data.data.length)
                  _this.noMore = true;
                else {
                  _this.offset++;
                  _this.commentData = _this.commentData.concat(response.data.data);
                  _this.$nextTick(() => {
                    _this.BScrollEvent.refresh();
                  })
                }
                _this.loading = false;
              })
            }
          }
        })
      },
    }
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  @import "../../../style/mixin.scss";
  @import "./comment.scss";
</style>
