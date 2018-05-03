<template>
    <view class="refresh-content refresh-container" :class="className" @touchstart="touchstart" @touchmove="touchmove" @touchend="touchend" :animation="animationData">
        <view class="refresh-load">
            <view class="refresh-pull-arrow"></view>
            <slot></slot>
        </view>
    </view>
</template>
<script>
export default {
  props: {
    scrollTop: {
      type: Number
    }
  },
  data () {
    return {
      touchYDelta: '',
      isLoading: false,
      loadWrapH: '',
      winfactor: 0.2,
      translateVal: '',
      isMoved: false,
      firstTouchY: '',
      initialScroll: '',
      friction: 2.5,
      triggerDistance: 100,
      className: '',
      animationData: {}
    }
  },
  methods: {
    scroll (ev) {
      console.log(ev)
    },
    touchstart (ev) {
      if (this.isLoading) {
        return
      }
      this.isMoved = false
      this.sDuration = '0ms'
      this.touchYDelta = ''
      const touchobj = ev.touches[0]
      this.firstTouchY = parseInt(touchobj.clientY)
      this.initialScroll = this.scrollTop
    },
    touchmove (ev) {
      if (this.isLoading) {
        return
      }
      let self = this
      let moving = function () {
        let touchobj = ev.touches[0]
        let touchY = parseInt(touchobj.clientY)
        let touchYDelta = touchY - self.firstTouchY
        self.touchYDelta = touchYDelta
        if (
          self.initialScroll > 0 ||
          self.scrollTop > 0 ||
          (self.scrollTop === 0 && touchYDelta < 0)
        ) {
          self.firstTouchY = touchY
          return
        }
        /* eslint-enable */
        let translateVal = Math.pow(touchYDelta, 0.85)
        let animation = wx.createAnimation({
          duration: 0,
          timingFunction: 'linear'
        })
        animation.translate3d(0, translateVal, 0).step()
        self.animationData = animation.export()
        self.isMoved = true
        if (touchYDelta > self.triggerDistance) {
          self.className = 'refresh-pull-up'
        } else {
          self.className = 'refresh-pull-down'
        }
      }
      this.throttle(moving(), 10)
    },
    touchend (ev) {
      let self = this
      if (this.isLoading || !this.isMoved) {
        this.isMoved = false
        return
      }
      // 根据下拉高度判断是否加载
      if (this.touchYDelta >= this.triggerDistance) {
        this.isLoading = true
        let animation = wx.createAnimation({
          duration: 300,
          timingFunction: 'linear'
        })
        animation.translate3d(0, 60, 0).step()
        self.animationData = animation.export()
        self.className = 'refreshing'
        this.$emit('refresh')
      } else {
        let animation = wx.createAnimation({
          duration: 300,
          timingFunction: 'linear'
        })
        animation.translate3d(0, 0, 0).step()
        self.animationData = animation.export()
      }
      this.isMoved = false
    },
    // scrollY () {
    //   wx
    //     .createSelectorQuery()
    //     .select('#scroll-container')
    //     .scrollOffset(res => {
    //       this.scrollTop = res.scrollTop
    //       this.$apply()
    //     })
    //     .exec()
    // },

    reset () {
      this.isLoading = false
      this.className = 'refresh-pull-up'
      let animation = wx.createAnimation({
        duration: 300,
        timingFunction: 'linear'
      })
      animation.translate3d(0, 0, 0).step()
      this.animationData = animation.export()
      this.className = 'refresh-pull-down'
      this.$apply()
    },

    throttle (fn, delay) {
      let allowSample = true
      return function (e) {
        if (allowSample) {
          allowSample = false
          setTimeout(function () {
            allowSample = true
          }, delay)
          fn(e)
        }
      }
    }
  }
}
</script>
<style lang="scss">
page {
  overflow: hidden;
}
.new-block {
  width: 100%;
  height: 100px;
  background-color: #333;
  &:nth-child(even) {
    background-color: #fff;
  }
}
.refresh-loader {
  width: 100%;
  height: 100%;
}
@keyframes rotate {
  0% {
    -webkit-transform: rotate(0deg) scale(1);
    transform: rotate(0deg) scale(1);
  }
  50% {
    -webkit-transform: rotate(180deg) scale(1);
    transform: rotate(180deg) scale(1);
  }
  100% {
    -webkit-transform: rotate(360deg) scale(1);
    transform: rotate(360deg) scale(1);
  }
}
@-webkit-keyframes rotate {
  0% {
    -webkit-transform: rotate(0deg) scale(1);
    transform: rotate(0deg) scale(1);
  }
  50% {
    -webkit-transform: rotate(180deg) scale(1);
    transform: rotate(180deg) scale(1);
  }
  100% {
    -webkit-transform: rotate(360deg) scale(1);
    transform: rotate(360deg) scale(1);
  }
}
.refresh-content {
  position: relative;
  -webkit-overflow-scrolling: touch;
  -webkit-transform: translate3d(0, 0, 0);
  transform: translate3d(0, 0, 0);
}
.refresh-load {
  width: 100%;
  height: 30px;
  position: absolute;
  top: -30px;
  left: 0;
  padding: 8px 0;
}
.refresh-pull-arrow {
  width: 20px;
  height: 20px;
  position: absolute;
  left: 50%;
  top: 50%;
  visibility: visible;
  margin-left: -10px;
  margin-top: -12px;
  background: no-repeat center;
  background-image: url('data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBzdGFuZGFsb25lPSJubyI/PjwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIgImh0dHA6Ly93d3cudzMub3JnL0dyYXBoaWNzL1NWRy8xLjEvRFREL3N2ZzExLmR0ZCI+PHN2ZyB0PSIxNTIzMzMzMjQ5ODAwIiBjbGFzcz0iaWNvbiIgc3R5bGU9IiIgdmlld0JveD0iMCAwIDEwMjQgMTAyNCIgdmVyc2lvbj0iMS4xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHAtaWQ9IjI4OTIiIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiB3aWR0aD0iNDgiIGhlaWdodD0iNDgiPjxkZWZzPjxzdHlsZSB0eXBlPSJ0ZXh0L2NzcyI+PC9zdHlsZT48L2RlZnM+PHBhdGggZD0iTTg3My42IDQxOS4ybC0zNTUuMi0zNjEuNmMtOS42LTkuNi0yMi40LTkuNi0zMiAwbC0zNTUuMiAzNjhjLTkuNiA5LjYtOS42IDIyLjQgMCAzMiA5LjYgOS42IDIyLjQgOS42IDMyIDBsMzE2LjgtMzI5LjYgMCA4MjguOGMwIDEyLjggOS42IDIyLjQgMjIuNCAyMi40czIyLjQtOS42IDIyLjQtMjIuNGwwLTgyMi40IDMxMC40IDMxNi44YzkuNiA5LjYgMjIuNCA5LjYgMzIgMEM4ODMuMiA0NDEuNiA4ODMuMiA0MjUuNiA4NzMuNiA0MTkuMnoiIHAtaWQ9IjI4OTMiPjwvcGF0aD48L3N2Zz4=');
  background-size: cover;
  z-index: 10;
  -webkit-transform: rotate(0deg) translate3d(0, 0, 0);
  transform: rotate(0deg) translate3d(0, 0, 0);
  -webkit-transition-duration: 300ms;
  transition-duration: 300ms;
}
.refresh-content.refresh-pull-down .refresh-pull-arrow {
  -webkit-transform: rotate(0deg) translate3d(0, 0, 0);
  transform: rotate(0deg) translate3d(0, 0, 0);
}
.refresh-content.refresh-pull-up .refresh-pull-arrow {
  -webkit-transform: rotate(180deg) translate3d(0, 0, 0);
  transform: rotate(180deg) translate3d(0, 0, 0);
}
.refreshing .refresh-pull-arrow {
  background-image: url('data:image/svg+xml;base64,PHN2ZyBjbGFzcz0iciIgd2lkdGg9JzEyMHB4JyBoZWlnaHQ9JzEyMHB4JyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIj4KICAgIDxyZWN0IHg9IjAiIHk9IjAiIHdpZHRoPSIxMDAiIGhlaWdodD0iMTAwIiBmaWxsPSJub25lIiBjbGFzcz0iYmsiPjwvcmVjdD4KICAgIDxyZWN0IHg9JzQ2LjUnIHk9JzQwJyB3aWR0aD0nNycgaGVpZ2h0PScyMCcgcng9JzUnIHJ5PSc1JyBmaWxsPScjRTlFOUU5JwogICAgICAgICAgdHJhbnNmb3JtPSdyb3RhdGUoMCA1MCA1MCkgdHJhbnNsYXRlKDAgLTMwKSc+CiAgICA8L3JlY3Q+CiAgICA8cmVjdCB4PSc0Ni41JyB5PSc0MCcgd2lkdGg9JzcnIGhlaWdodD0nMjAnIHJ4PSc1JyByeT0nNScgZmlsbD0nIzk4OTY5NycKICAgICAgICAgIHRyYW5zZm9ybT0ncm90YXRlKDMwIDUwIDUwKSB0cmFuc2xhdGUoMCAtMzApJz4KICAgICAgICAgICAgICAgICByZXBlYXRDb3VudD0naW5kZWZpbml0ZScvPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyM5Qjk5OUEnCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSg2MCA1MCA1MCkgdHJhbnNsYXRlKDAgLTMwKSc+CiAgICAgICAgICAgICAgICAgcmVwZWF0Q291bnQ9J2luZGVmaW5pdGUnLz4KICAgIDwvcmVjdD4KICAgIDxyZWN0IHg9JzQ2LjUnIHk9JzQwJyB3aWR0aD0nNycgaGVpZ2h0PScyMCcgcng9JzUnIHJ5PSc1JyBmaWxsPScjQTNBMUEyJwogICAgICAgICAgdHJhbnNmb3JtPSdyb3RhdGUoOTAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyNBQkE5QUEnCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSgxMjAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyNCMkIyQjInCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSgxNTAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyNCQUI4QjknCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSgxODAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyNDMkMwQzEnCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSgyMTAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyNDQkNCQ0InCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSgyNDAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyNEMkQyRDInCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSgyNzAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyNEQURBREEnCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSgzMDAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0PgogICAgPHJlY3QgeD0nNDYuNScgeT0nNDAnIHdpZHRoPSc3JyBoZWlnaHQ9JzIwJyByeD0nNScgcnk9JzUnIGZpbGw9JyNFMkUyRTInCiAgICAgICAgICB0cmFuc2Zvcm09J3JvdGF0ZSgzMzAgNTAgNTApIHRyYW5zbGF0ZSgwIC0zMCknPgogICAgPC9yZWN0Pgo8L3N2Zz4=');
  -webkit-animation: rotate 1s 0s linear infinite;
  animation: rotate 1s 0s linear infinite;
}

.loading-load {
  width: 100%;
  height: 30px;
  position: relative;
  padding: 8px 0;
}
</style>
