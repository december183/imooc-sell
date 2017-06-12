<template>
  <transition name="move">
    <div class="food" v-show="showFlag" ref="foodWrapper">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image" :alt="food.name" />
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <div class="title">{{food.name}}</div>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span>
            <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food" @add="addFood"></cartcontrol>
          </div>
          <transition name="fade">
            <div @click="addFirst" class="buy" v-show="!food.count || food.count === 0">加入购物车</div>
          </transition>
        </div>
        <split v-show="food.info"></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品介绍</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <ratingselect @select="selectRating" @toggle="toggleContent" :selectType="selectType" :onlyContent="onlyContent" :desc="desc" :ratings="food.ratings"></ratingselect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li v-show="needShow(rating.rateType, rating.text)" class="rating-item border-1px" v-for="rating in food.ratings">
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img class="avatar" width="12" height="12" :src="rating.avatar" />
                </div>
                <div class="time">{{rating.rateTime | formatDate}}</div>
                <p class="text"><span :class="{'icon-thumb_up': rating.rateType === 0, 'icon-thumb_down': rating.rateType === 1}"></span>{{rating.text}}</p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>
<script>
  import BScroll from 'better-scroll';
  import { formatDate } from 'common/js/date';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import split from 'components/split/split';
  import ratingselect from 'components/ratingselect/ratingselect';

  const ALL = 2;

  export default {
    name: 'food',
    props: {
      food: {
        type: Object
      }
    },
    data () {
      return {
        showFlag: false,
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',
          positive: '推荐',
          negative: '吐槽'
        }
      }
    },
    methods: {
      show() {
        this.showFlag = true;
        this.$nextTick(() => {
          if(!this.scroll) {
            this.scroll = new BScroll(this.$refs.foodWrapper, {
              click : true
            });
          } else {
            this.scroll.refresh();
          }
        });
      },
      hide() {
        this.showFlag = false;
      },
      addFirst(event) {
        if(!event._constructed) {
          return;
        }
        this.$emit('add', event.target);
        this.$set(this.food, 'count', 1);
      },
      addFood(target) {
        this.$emit('add', target);
      },
      selectRating(type) {
        this.selectType = type;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },
      toggleContent() {
        this.onlyContent = !this.onlyContent;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },
      needShow(type, text) {
        if(this.onlyContent && !text) {
          return false;
        }
        if(this.selectType === ALL) {
          return true;
        } else {
          return this.selectType === type;
        }
      }
    },
    filters: {
      formatDate(time) {
        let date = new Date(time);
        return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },
    components: {
      cartcontrol,
      split,
      ratingselect
    }
  }
</script>
<style lang="stylus" scoped>
  @import '../../common/stylus/mixin.styl';
  
  .move-enter-active, .move-leave
    transition: all 0.2s linear
    transform: translate3d(0, 0, 0)
  .move-enter, .move-leave-active
    transition: all 0.2s linear
    transform: translate3d(100%, 0, 0)
  .food
    position: fixed
    z-index: 30
    top: 0
    left: 0
    bottom: 48px
    width: 100%
    background: #fff
    .food-content
      .image-header
        position: relative
        width: 100%
        height: 0
        padding-top: 100%
        img
          position: absolute
          top: 0
          left: 0
          width: 100%
          height: 100%
        .back
          position: absolute
          top: 10px
          left: 0
          .icon-arrow_lift
            display: block
            padding: 10px
            font-size: 20px
            color: #fff
      .content
        position: relative
        padding: 18px
        .title
          font-size: 14px
          font-weight: 700
          line-height: 14px
          color: rgb(7, 17, 27)
        .detail
          padding-top: 8px
          padding-bottom: 16px
          .sell-count, .rating
            font-size: 10px
            color: rgb(147, 153, 159)
            line-height: 10px
          .sell-count
            margin-right: 12px
        .price
          font-weight: 700
          line-height: 24px
          .now
            margin-right: 8px
            font-size: 14px
            color: rgb(240, 20, 20)
          .old
            text-decoration: line-through
            font-size: 10px
            color: rgb(147, 153, 159)
        .cartcontrol-wrapper
          position: absolute
          right: 12px
          bottom: 12px
        .buy
          position: absolute
          z-index: 10
          right: 18px
          bottom: 18px
          height: 24px
          line-height: 24px
          padding: 0 12px
          box-sizing: border-box
          border-radius: 12px
          font-size: 10px
          color: #fff
          background: rgb(0, 160, 220)
        &.fade-enter-active, &.fade-leave
          transition: all 0.2s ease
          opaticy: 0
        &.fade-enter, &.fade-leave-active
          opaticy: 1
      .info
        padding: 18px
        .title
          font-size: 14px
          line-height: 14px
          color: rgb(7, 17, 27)
          margin-bottom: 6px
        .text
          font-size: 12px
          font-weight: 200
          color: rgb(77, 85, 93)
          line-height: 24px
          padding: 0 8px
      .rating
        padding-top: 18px
        .title
          font-size: 14px
          line-height: 14px
          color: rgb(7, 17, 27)
          margin-left: 18px
        .rating-wrapper
          padding: 0 18px
          .rating-item
            position: relative
            padding: 16px 0
            border-1px(rgba(7, 17, 27, 0.1))
            .user
              position: absolute
              right: 0
              top: 16px
              font-size: 0
              line-height: 12px
              .avatar
                border-radius: 50%
              .name
                display: inline-block
                vertical-align: top
                margin-right: 6px
                font-size: 10px
                color: rgb(147, 153, 159)
            .time
              margin-bottom: 6px
              font-size: 10px
              color: rgb(147, 153, 159)
              line-height: 12px
            .text
              font-size: 12px
              color: rgb(7, 17, 27)
              line-height: 16px
              .icon-thumb_up, .icon-thumb_down
                line-height: 24px
                margin-right: 4px
              .icon-thumb_up
                color: rgb(0, 160, 220)
              .icon-thumb_down
                color: rgb(147, 153, 159)
                
</style>