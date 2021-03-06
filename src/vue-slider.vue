<template>
  <div id="slider">
    <div id="slider-wrap" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd">
      <div class="slider-item" v-for="item in images" :key="item">
        <img class="slider-img" :src="item" alt="">
      </div>
    </div>
    <div class="dot">
      <span
        class="dot-item"
        :class="currentIndex === index ? 'active' : ''"
        v-for="index in images.length"
        :key="index"
      >
      </span>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'VueSlider',
    props: {
      images: {
        type: Array,
        default: () => []
      }
    },
    data () {
      return {
        currentIndex: 1
      }
    },
    mounted () {
      if (this.images.length) {
        this.initShard()
        this.initSlider()
      }
    },
    methods: {
      setStyle (el, styleName, style) {
        el && (el.style[styleName] = style)
      },
      initShard () {
        this.shared = {
          windowWidth: window.innerWidth,
          currentLeft: window.innerWidth,
          wrap: document.getElementById('slider-wrap'),
          items: document.getElementsByClassName('slider-item'),
          len: this.images.length
        }
        this.touch = {}
        this.timer = null
      },
      initSlider () {
        let { wrap, items, len, windowWidth, currentLeft } = this.shared
        const loopLength = len + 2
        this.handleLoop(wrap, items)
        this.setStyle(wrap, 'width', `${windowWidth * loopLength}px`)
        this.transitionWrap(-currentLeft, 0)
        for (let i = 0; i < items.length; i++) {
          this.setStyle(items[i], 'width', `${windowWidth}px`)
        }
        this.loopStart()
      },
      handleLoop () {
        let { wrap, items } = this.shared
        let firstChild = items[0].cloneNode(true)
        let lastChild = items[items.length - 1].cloneNode(true)
        wrap.insertBefore(lastChild, items[0])
        wrap.appendChild(firstChild)
        wrap.addEventListener('transitionend', () => {
          let { currentLeft, len, windowWidth } = this.shared
          if (currentLeft > windowWidth * len) {
            currentLeft = windowWidth
            this.transitionWrap(-currentLeft, 0)
            this.updateShared({ currentLeft })
          } else if (currentLeft === 0){
            currentLeft = windowWidth * len
            this.transitionWrap(-currentLeft, 0)
            this.updateShared({ currentLeft })
          }
        })
      },
      transitionWrap (left, duration) {
        let { wrap } = this.shared
        this.setStyle(wrap, 'transitionDuration', `${duration}ms`)
        this.setStyle(wrap, 'transform', `translate3d(${left}px, 0px, 0px)`)
      },
      next () {
        let { wrap, windowWidth, currentLeft } = this.shared
        currentLeft += windowWidth
        this.transitionWrap(-currentLeft, 300)
        this.updateCurrentIndex(true)
        this.updateShared({ currentLeft })
      },
      prev () {
        let { wrap, windowWidth, currentLeft } = this.shared
        currentLeft -= windowWidth
        this.transitionWrap(-currentLeft, 300)
        this.updateCurrentIndex(false)
        this.updateShared({ currentLeft })
      },
      loopStart () {
        !this.timer && (this.timer = setInterval(() => {
          this.next()
        }, 4000))
      },
      loopStop () {
        clearInterval(this.timer)
        this.timer = null
      },
      updateCurrentIndex (add) {
        let { len } = this.shared
        let cur  = this.currentIndex
        cur = add
          ? ++ cur > len
            ? 1 : cur
          : -- cur < 1
            ? len : cur
        this.currentIndex = cur
      },
      updateShared (o) {
        Object.keys(o).forEach(key => {
          this.shared[key] = o[key]
        })
      },
      onTouchStart (e) {
        this.loopStop()
        let touch = e.touches[0]
        this.touch.pageY = touch.pageY
        this.touch.pageX = touch.pageX
      },
      onTouchMove (e) {
        let { wrap, currentLeft } = this.shared
        let touch = e.touches[0]
        let diff = touch.pageX - this.touch.pageX
        this.touch.diff = diff
        this.transitionWrap(-currentLeft + diff, 0)
      },
      onTouchEnd () {
        let { diff } = this.touch
        let { currentLeft, wrap, windowWidth } = this.shared
        let left = diff + -currentLeft
        if (Math.abs(diff) > windowWidth / 2) {
          if (diff < 0) {
            this.next()
          } else {
            this.prev()
          }
        } else {
          this.transitionWrap(-currentLeft, 300)
        }
        this.touch.diff = 0
        this.loopStart()
      }
    }
  }
</script>


<style>
  #slider {
    width: 100%;
    font-size: 0;
    overflow: hidden;
    position: relative;
  }
  #slider-wrap {
    overflow: hidden;
  }
  .slider-item {
    float: left;
  }
  .slider-img {
    width: 100%;
    -webkit-backface-visibility: hidden
  }
  .dot {
    position: absolute;
    bottom: 5px;
    left: 50%;
    transform: translateX(-50%)
  }
  .dot-item {
    display: inline-block;
    height: 8px;
    width: 8px;
    background: rgb(255, 255, 255, 0.5);
    border-radius: 50%;
    margin-right: 5px;
  }
  .dot-item:last-child {
    margin-right: 0px;
  }
  .dot-item.active {
    background: #fff;
  }
</style>
