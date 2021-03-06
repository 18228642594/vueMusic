<template>
  <div class="music-list">
      <div class="back" @click="back"><!--此处是指点击返回按钮的时候返回父组件-->
          <i class="icon-back"></i>
      </div>
      <h1 class="title" v-html="title"></h1>
      <div class="bg-image" :style="bgStyle" ref="bgImage">
          <div class="play-wrapper">
              <div class="play"  v-show="songs.length>0" ref="playBtn" @click="random"><!--此处的v-show是指当songs列表的数据加载完成才显示按钮-->
                  <i class="icon-play"></i>
                  <span class="text">随机播放全部</span>
              </div>
          </div>
          <div class="filter" ref="filter"></div>
      </div>
      <div class="bg-layer" ref="layer"></div><!--是有推动效果的-->
      <scroll 
      :data="songs" 
      class="list" 
      @scroll="scroll"
      :probe-type="probeType"
      :listen-scroll="listenScroll"
      ref="list"><!--封装好的scroll组件-->
        <div class="song-list-wrapper">
            <song-list :rank="rank" :songs="songs"  @select="selectItem" ></song-list>
        </div>
        <div class="loading-container" v-show="songs.length"></div>
      </scroll>
  </div>    
</template>

<script>
import Scroll from '../../base/scroll/scroll'
import SongList from '../../base/song-list/song-list'
import {prefixStyle} from '../../common/js/dom'
import {mapActions} from 'vuex'
import {playlistMixin} from '../../common/js/mixin'
const RESERVED_HEIGHT = 40
const transform = prefixStyle('transform')
const backdrop = prefixStyle('backdrop-filter')
export default {
  name: 'music-list',
  mixins: [playlistMixin],
  props: {
    bgImage: {
      type: String,
      default: ''
    },
    songs: {
      type: Array,
      default() {
        return []
      }
    },
    title: {
      type: String,
      default: ''
    },
    rank: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      scrollY: 0
    }
  },
  computed: {
    bgStyle() {
      return `background-image:url(${this.bgImage})`
    }
  },
  created() { // 用于实时监听
    this.probeType = 3
    this.listenScroll = true
  },
  mounted() { // list的top是指背景图的高度
    this.imageHeight = this.$refs.bgImage.clientHeight
    this.minTransalteY = -this.imageHeight + RESERVED_HEIGHT
    // this.$refs.list.$el.style.top = `${this.imageHeight}px`
    this.$refs.list.$el.style.top = `${this.imageHeight}px`
  },
  methods: {
    handlePlaylist(playList) {
      const bottom = playList.length > 0 ? '60px' : ''
      this.$refs.list.$el.style.bottom = bottom
      this.$refs.list.refresh()// 让scroll重新刷新一次
    },
    scroll(pos) {
      this.scrollY = pos.y // 此处是指y抽的坐标
    },
    back() {
      this.$router.back()
    },
    selectItem(item, index) {
      this.selectPlay({
        list: this.songs,
        index
      })
    },
    random() { // 次吃是random随机播放事件
      this.randomPlay({
        list: this.songs
      })
    },
    ...mapActions([
      'selectPlay',
      'randomPlay'
    ])
  },
  watch: {
    scrollY(newVal) {
      let translateY = Math.max(this.minTransalteY, newVal)
      let zIndex = 0// 定义一个变量   设置z-index的值为0
      let scale = 1 // 定义一个scale是计算图片的缩放比的
      let blur = 0  // 滚动文字的时候图片有个高丝模糊的效果
      // 设置变量  修改两个地方   因为文字滚动到顶部的时候，会有图片 遮挡
      this.$refs.layer.style[transform] = `translate3d(0,${translateY}px,0)`
     // this.$refs.layer.style['webkitTransform'] = `translate3d(0,${translateY}px,0)`
      const percent = Math.abs(newVal / this.imageHeight)// 图片放大逻辑  math.abs()是指求绝对值
    //  console.log(percent)
      if (newVal > 0) { // 图片放大逻辑
        scale = 1 + percent
        zIndex = 10
      } else {
        blur = Math.min(20 + percent, 20)
      }
      this.$refs.filter.style[backdrop] = `blur(${blur}px)`
      // this.$refs.filter.style['webkitBackdrop-filter'] = `blur(${blur}px)`
      // this.$refs.layer.style['transform'] = `translate3d(0,${newVal}px,0)`
      if (newVal < this.minTransalteY) { // 判断scroll滚动的值 如果小于minTransform的值（图片的高度-40）就进行一下逻辑
        zIndex = 10
        this.$refs.bgImage.style.paddingTop = 0// 如果小于  图片的padding值为0
        this.$refs.bgImage.style.height = `${RESERVED_HEIGHT}px`// 图片高度为40
        this.$refs.playBtn.style.display = 'none'
      } else {
        this.$refs.bgImage.style.paddingTop = `70%`
        this.$refs.bgImage.style.height = `${RESERVED_HEIGHT}px`
        this.$refs.playBtn.style.display = ''
      }
      this.$refs.bgImage.style.zIndex = zIndex// 如果滚动的值小于最小高度成立，就设置index样式属性
      this.$refs.bgImage.style[transform] = `scale(${scale})`// 图片放大逻辑
      // this.$refs.bgImage.style['webkitTransform'] = `scale(${scale})`// 图片放大逻辑
    }
  },
  components: {
    Scroll,
    SongList
  }
}
</script>

<style scoped="scoped" lang="stylus" rel="stylesheet/stylus">
 @import "~common/stylus/variable"
 @import "~common/stylus/mixin"
  .music-list
    position: fixed
    z-index: 100
    top: 0
    left: 0
    bottom: 0
    right: 0
    background: $color-background
    .back
      position absolute
      top: 0
      left: 6px
      z-index: 50
      .icon-back
        display: block
        padding: 10px
        font-size: $font-size-large-x
        color: $color-theme
    .title
      position: absolute
      top: 0
      left: 10%
      z-index: 40
      width: 80%
      no-wrap()
      text-align: center
      line-height: 40px
      font-size: $font-size-large
      color: $color-text
    .bg-image
      position: relative
      width: 100%
      height: 0
      padding-top: 70%
      transform-origin: top
      background-size: cover
      .play-wrapper
        position: absolute
        bottom: 20px
        z-index: 50
        width: 100%
        .play
          box-sizing: border-box
          width: 135px
          padding: 7px 0
          margin: 0 auto
          text-align: center
          border: 1px solid $color-theme
          color: $color-theme
          border-radius: 100px
          font-size: 0
          .icon-play
            display: inline-block
            vertical-align: middle
            margin-right: 6px
            font-size: $font-size-medium-x
          .text
            display: inline-block
            vertical-align: middle
            font-size: $font-size-small
      .filter
        position: absolute
        top: 0
        left: 0
        width: 100%
        height: 100%
        background: rgba(7, 17, 27, 0.4)
    .bg-layer
      position: relative
      height: 100%
      background: $color-background
    .list
      position: fixed
      top: 0
      bottom: 0
      width: 100%
      background: $color-background
      .song-list-wrapper
        padding: 20px 30px
      .loading-container
        position: absolute
        width: 100%
        top: 50%
        transform: translateY(-50%)
        
</style>
