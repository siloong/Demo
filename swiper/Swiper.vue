<template>
  <div class="swiper-wrap" :style="{width: swiperWidth + 'px', height: swiperHeight + 'px'}">
    <ul class="swiper" :style="swiperStyle"
    @touchstart="handleTouchStart"
    @touchmove="handleTouchMove"
    @touchend="handleTouchEnd">
      <li class="swiper-item"
      v-for="(item, index) in imgList"
      :key="index">
        <img :src="item" :style="{width: swiperWidth + 'px', height: swiperHeight + 'px'}" />
      </li>
    </ul>

    <ol class="swiper-point">
      <li v-for="(item, index) in swiperItemCount" :key="index" :class="{current: index  === pointIndex }"></li>
    </ol>
  </div>
</template>



<script>
  export default {
    name: 'Swiper',
    props: {
      imgList: {
        type: Array,
        default() {
          return [];
        }
      },
      // swiper's width
      swiperWidth: {
        type: Number,
        default: 375
      },
      // swiper's height
      swiperHeight: Number,
      //slide timing ms
      interval: {
        type: Number,
        default: 1000
      },
      // tionsition timing ms
      animDuration: {
        type: Number,
        default: 500
      }
    },
    data() {
      return {
        // 定时器的ID
        timerID: 0,
        // 当前view的swiper-item
        currentIndex: 1,
        swiperItemWidth: {},
        swiperItemCount: 0,
        swiperStyle: {},
        scrolling: false
      }
    },
    mounted() {
      setTimeout(() => {
        this.hanleDom();
        this.timingStart();
      }, 100);
    },
    computed: {
      // swiper bar current's index
      pointIndex() {
        const index = this.currentIndex - 1
        if (index >= 5) {
          return 0
        } else {
          return index
        }
      }
    },
    methods: {
      // 启动定时器
      timingStart() {
        window.clearInterval(this.timerID);
        this.timerID = window.setInterval(() => {
          this.currentIndex++;
          this.swiper(-this.currentIndex * this.swiperItemWidth);
        }, this.interval);
      },

      // 关闭定时器
      timingStop() {
        window.clearInterval(this.timerID);
      },
      
      // swiper实现汇总
      swiper(currentPostition) {
        this.scrolling = true;
        this.swiperStyle.transition = `transform ${this.animDuration}ms`;
        this.setTransform(currentPostition);
        this.revisePostitionX();
        this.scrolling = false;
      },

      // dom node handle method
      hanleDom() {
        // 获取操作元素
        let swiperEl = document.querySelector('.swiper');
        let swiperItemEls = swiperEl.querySelectorAll('.swiper-item');

        // 保存swiper item个数
        this.swiperItemCount = swiperItemEls.length;
        // 如果swiper item个数大于1，则给前和尾添加swiper item用于衔接
        if(this.swiperItemCount > 1) {
          // deepin clone node element
          let cloneFirst = swiperItemEls[0].cloneNode(true);
          let cloneLast = swiperItemEls[this.swiperItemCount - 1].cloneNode(true);
          // swiper's end  and start insert node element
          swiperEl.appendChild(cloneFirst);
          swiperEl.insertBefore(cloneLast, swiperItemEls[0]);
          this.swiperItemWidth = cloneFirst.offsetWidth;
        }
        this.setTransform(-this.swiperItemWidth);
      },

      setTransform(positionX) {
        this.$set(this.swiperStyle, 'transform', `translateX(${positionX}px)`)
      },

      revisePostitionX() {
        window.setTimeout(() => {
          this.swiperStyle.transition = 'none';
          if(this.currentIndex >= this.swiperItemCount + 1) {
            // this.timingStop();
            this.currentIndex = 1;
            this.setTransform(-this.swiperItemWidth * this.currentIndex);
            // this.setTransform(-this.currentIndex * this.swiperItemWidth + this.distance);
          } else if (this.currentIndex <= 0) {
            this.swiperStyle.transition = 'none';
            this.currentIndex = this.swiperItemCount;
            this.setTransform(-this.swiperItemWidth * this.currentIndex);
          }

        }, this.animDuration);
        
      },

      handleTouchStart(event) {
        if(!this.scrolling) {
          this.timingStop();
          this.startX = event.touches[0].clientX;
        }
      },

      handleTouchMove(event) {
        let moveX = event.touches[0].clientX;
        this.distance = moveX - this.startX;
        // console.log(this.distance);
        this.setTransform(-this.currentIndex * this.swiperItemWidth + this.distance);
        event.preventDefault();
      },

      handleTouchEnd() {
        if(Math.abs(this.distance) > 50) {
          if(this.distance > 0) {
            this.currentIndex--;
          } else {
            this.currentIndex++;
          }
          this.swiper(-this.currentIndex * this.swiperItemWidth);
        } else {
          this.swiper(-this.currentIndex * this.swiperItemWidth);
        }
        this.timingStart();

      }


    }

  }

</script>


<style>
  ul,
  ol,
  li,
  img,
  div {
    margin: 0; padding: 0;
  }
  ul,
  ol {
    list-style: none;
  }

  .swiper-wrap {
    position: relative;
    width: 100%;
    overflow: hidden;
  }

  /* swiper's style start */
  .swiper {
    display: flex;
  }
  .swiper-item {
    width: 100%;
  }
  .swiper-item img {
    width: 375px;
  }

  /* swiper's point start */
  .swiper-point {
    position: absolute;
    left: 10px;
    bottom: 10px;
    display: flex;
  }
  .swiper-point li {
    width: 6px;
    height: 6px;
    border-radius: 6px;
    background-color: rgba(255, 255, 255, 0.8);
  }
  .swiper-point li:nth-child(n+2) {
    margin-left: 4px;
  }

  /* current active swiper's point style */
  .swiper-point .current {
    transition: all .3s;
    width: 20px;
    background-color: #fff;
  }
</style>
