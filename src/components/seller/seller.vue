<style lang="stylus" scoped>
.seller-wrapper
  position absolute
  top 174px
  bottom 0
  left 0
  width 100%
  overflow hidden
  .seller-content
    .info
      padding 18px 0
      margin 0 18px
      .title
        padding-bottom 18px
        border-bottom 1px solid rgba(7,17,27,0.1)
        .text
          font-size 14px
          color rgb(7,17,27)
          line-height 14px
        .star-wrapper
          padding-top 8px
          font-size 0
          .star
            display inline-block
            vertical-align top
          .rate-count,.sell-count
            display inline-block
            font-size 10px
            color rgb(77,85,93)
            line-height 18px
          .rate-count
            padding 0 12px 0 8px
        .collect
          position absolute
          top 18px
          right 8px
          width 50px
          text-align center
          .icon-favorite
            font-size 24px
            line-height 24px
            color #d4d6d9
            &.active
              color rgb(240,20,20)
          .text
            display block
            font-size 10px
            color rgb(77,85,93)
            line-height 10px
            padding-top 4px
      .remark
        display flex
        .block
          flex 1
          margin-top 18px
          text-align center
          border-right 1px solid rgba(7,17,27,0.1)
          &:last-child
            border none
          h2
            font-size 10px
            color rgb(147,153,159)
            line-height 10px
            margin-bottom 4px
          .content
            font-size 10px
            color rgb(7,17,27)
            line-height 24px
            font-weight 200
            .num
              position relative
              top 2px
              font-size 24px
    .activities
      padding-top 18px
      .bulletin
        margin 0 18px
        border-bottom 1px solid rgba(7,17,27,0.1)
        h1
          font-size 14px
          color #07111b
          line-height 14px
        .content
          padding 8px 12px 16px 12px
          font-size 12px
          font-weight 200
          color rgb(240,20,20)
          line-height 24px
    .supports
      margin 0 18px
      .item
        padding 16px
        border-bottom 1px solid rgba(7,17,27,0.1)
        font-size 0
        .iconMap
          width 16px
          height 16px
          vertical-align top
          margin-right 6px
        .text
          font-size 12px
          font-weight 200
          color rgb(7,17,27)
          line-height 16px
    .seller-imgs
      margin 18px
      white-space nowrap
      overflow hidden
      h1
        font-size 14px
        line-height 14px
        margin-bottom 12px
      img
        margin-right 6px
    .seller-info
      h1
        margin 0 18px
        padding 18px 0 12px 0
        border-bottom 1px solid rgba(7,17,27,0.1)
      .info-list
        .info
          font-size 12px
          font-weight 200
          color rgb(7,17,27)
          line-height 16px
          padding 16px 12px
          border-bottom 1px solid rgba(7,17,27,0.1)
          &:last-child
            border none
</style>

<template lang="html">
  <div class="seller-wrapper" ref="sellerWrapper">
    <div class="seller-content">
      <div class="info">
        <div class="title">
          <div class="text">{{seller.name}}</div>
          <div class="star-wrapper">
            <star :size="36" :score="computedScore(seller.gradeServer,seller.gradeCook)"></star>
            <span class="rate-count">({{seller.likeCount}})</span>
            <span class="sell-count">月售{{seller.monthlyCounts}}单</span>
          </div>
          <div class="collect" @click="collectShop">
            <span class="icon-favorite" :class="{'active':seller.collectFlag}"></span>
            <span class="text" >{{seller.collectFlag?'已收藏':'收藏'}}</span>
          </div>
        </div>
        <div class="remark">
          <div class="block">
            <h2>起送价</h2>
            <div class="content">
              <span class="num">{{seller.minPrice}}</span>元
            </div>
          </div>
          <div class="block">
            <h2>商家配送</h2>
            <div class="content">
              <span class="num">{{seller.deliveryMoney}}</span>元
            </div>
          </div>
          <div class="block">
            <h2>平均配送时间</h2>
            <div class="content">
              <span class="num">{{seller.deliveryTime}}</span>分钟
            </div>
          </div>
        </div>
      </div>
      <div class="divider"></div>
      <div class="activities">
        <div class="bulletin">
          <h1>公告与活动</h1>
          <div class="content">
            {{seller.introduce}}
          </div>
        </div>
      </div>
      <div class="supports">
        <ul>
          <li class="item" v-for="item in seller.discountDescList">
            <iconMap :iconType="item.id-1"></iconMap>
            <span class="text">{{item.discountDesc}}</span>
          </li>
        </ul>
      </div>
      <div class="divider"></div>
      <div class="seller-imgs">
        <h1>商家实景</h1>
        <div class="img-wrapper" ref="picsWrapper">
          <div ref="picList">
            <img v-for="pic in seller.imgList" :src="pic" width="120" height="90">
          </div>
        </div>
      </div>
      <div class="divider"></div>
      <div class="seller-info">
        <h1>商家信息</h1>
        <ul class="info-list">
          <li class="info">{{seller.address}}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import BScroll from 'better-scroll'
import star from 'components/star/star'
import iconMap from 'components/iconMap/iconMap'

export default {
  props: {
    shopId: String
  },
  components: {
    star: star,
    iconMap: iconMap
  },
  data() {
    return {
      seller: {}
    }
  },
  created() {
    this._init()
  },
  methods: {
    _init() {
      axios.get('/api/shop/' + this.shopId).then((res) => {
        this.seller = res.data
        this.$nextTick(() => {
          this.sellerScroll = new BScroll(this.$refs.sellerWrapper, {
            click: true
          })
          this._initPicScroll()
        })
      })
    },
    _initPicScroll() {
      if (this.picsScroll) {
        return
      }
      const PIC_WIDTH = 120
      const MARGIN = 6
      let picLen = this.seller.imgList.length
      this.$refs.picList.style.width = PIC_WIDTH * picLen + MARGIN * (picLen - 1) + 'px'
      this.picsScroll = new BScroll(this.$refs.picsWrapper, {
        scrollX: true
      })
    },
    computedScore(score1, score2) {
      return (parseFloat(score1) + parseFloat(score2)) / 2
    },
    collectShop() {
      if (this.seller.collectFlag) {
        axios.post('/api/shop/uncollect', {'shopId': this.seller.id}, {
          headers: {
            'Content-Type': 'application/json'
          }
        }).then((res) => {
          if (res.data === 'success') {
            this.seller.collectFlag = false
          }
        })
      } else {
        axios.post('/api/shop/collect', {'shopId': this.seller.id}, {
          headers: {
            'Content-Type': 'application/json'
          }
        }).then((res) => {
          if (res.data === 'success') {
            this.seller.collectFlag = true
          }
        })
      }
    }
  }
}

</script>
