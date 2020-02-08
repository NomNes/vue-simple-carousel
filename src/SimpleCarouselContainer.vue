<template lang="pug">
    .vue-simple-carousel-container
        .vsc-scroller(@touchstart="touchStart")
            .vsc-items(:style="{width: `${itemsLength * (100 / itemsOnScreen)}%`}" ref="items" :class="{moving}")
                slot
        .vsc-controls(v-if="itemsLength > 1")
            .arrow.prev(:class="{disabled: !hasPrev}" @click="previous")
              slot(name="arrow-prev")
                | &larr;
            .vsc-points
                .vsc-point(v-for="point in points" :key="point" :class="{current: currentIndex === point}" @click="setCurrentIndex(point)")
            .arrow.next(:class="{disabled: !hasNext}" @click="next")
              slot(name="arrow-next")
                | &rarr;
</template>

<script lang="ts">
import { Component, Vue, Prop, Watch } from 'vue-property-decorator'

@Component<SimpleCarouselContainer>({
    mounted() {
        this.setItemsLength();
        // @ts-ignore
        if (process.browser) {
            document.addEventListener("touchmove", this.touchMove.bind(this));
            document.addEventListener("touchend", this.touchEnd.bind(this));
        }
    },
    beforeDestroy() {
        // @ts-ignore
        if (process.browser) {
            document.removeEventListener("touchmove", this.touchMove.bind(this));
            document.removeEventListener("touchend", this.touchEnd.bind(this));
        }
    }
})
export default class SimpleCarouselContainer extends Vue {
  @Prop({ type: Boolean }) loop!: boolean;
  @Prop() watchIt!: any;
  @Prop({ type: Number, default: 1 }) itemsOnScreen!: number

  itemsLength = 0;

  public currentIndex = 0;
  public moving = false;

  setItemsLength () {
      this.itemsLength = this.$slots.default && this.$slots.default.length ? this.$slots.default.filter(s => s.tag).length : 0
  }

  get points () {
      return Array.from(Array(this.itemsLength - this.itemsOnScreen + 1).keys())
  }

  get hasNext () {
    return this.loop || this.currentIndex + 1 < this.points.length
  }
  get hasPrev () {
    return this.loop || this.currentIndex > 0
  }

  public previous () {
    if (this.hasPrev) {
      this.setCurrentIndex(this.currentIndex - 1)
    }
  }
  public next () {
    if (this.hasNext) {
      this.setCurrentIndex(this.currentIndex + 1)
    }
  }

  setCurrentIndex (index: number) {
    if (this.moving) return false
    if (index < 0) index = this.loop ? this.itemsLength - 1 : 0
    if (this.itemsLength < index + 1) index = this.loop ? 0 : this.itemsLength - 1
    this.currentIndex = index
  }

  updateOrder (index: number) {
    for (let i = 0; i < this.itemsLength; i++) {
      if (i >= index) {
        // @ts-ignore
        this.$refs.items.children[i].style.order = i - index
      } else {
        // @ts-ignore
        this.$refs.items.children[i].style.order = this.itemsLength - index + i
      }
    }
  }

  initTouchX: false | number = false;
  endTouchX: false | number = false;
  touchStart (event: TouchEvent) {
    if (event.touches && event.touches.length) {
      this.initTouchX = event.touches[0].clientX;
    }
  }
  touchEnd () {
    if (this.initTouchX !== false && this.endTouchX !== false) {
        const delta = this.initTouchX - this.endTouchX;
        if (Math.abs(delta) > 80) {
          if (delta < 0) {
              this.setCurrentIndex(this.currentIndex - 1)
          } else {
              this.setCurrentIndex(this.currentIndex + 1)
          }
        }
    }
    this.initTouchX = false;
    this.endTouchX = false;
  }
  touchMove (event: TouchEvent) {
      if (this.initTouchX !== false && event.touches && event.touches.length) {
          this.endTouchX = event.touches[0].clientX;
      }
  }

  @Watch('watchIt')
  onWatchChange() {
      this.setItemsLength();
  }

  @Watch('currentIndex')
  onCurrentIndexChange (nextIndex: number, previousIndex: number) {
    this.moving = true
    let delta = nextIndex - previousIndex
    if (this.loop && Math.abs(this.itemsLength / delta) < 2) delta = (this.itemsLength - Math.abs(delta)) * (delta > 0 ? -1 : 1)
    this.$emit('change', nextIndex)
    const itemWidth = 100 / this.itemsOnScreen
    if (delta > 0) {
      // @ts-ignore
      this.$refs.items.style.transition = '.2s'
      // @ts-ignore
      this.$refs.items.style.marginLeft = `${-itemWidth * delta}%`
      setTimeout(() => {
        // @ts-ignore
        this.$refs.items.style.transition = '0s'
        setTimeout(() => {
          // @ts-ignore
          this.$refs.items.style.marginLeft = '0'
          this.updateOrder(nextIndex)
          this.moving = false
        })
      }, 200)
    } else {
      this.updateOrder(nextIndex)
      // @ts-ignore
      this.$refs.items.style.marginLeft = `${itemWidth * delta}%`
      setTimeout(() => {
        // @ts-ignore
        this.$refs.items.style.transition = '.2s'
        setTimeout(() => {
          // @ts-ignore
          this.$refs.items.style.marginLeft = '0'
          setTimeout(() => {
            // @ts-ignore
            this.$refs.items.style.transition = '0s'
            this.moving = false
          }, 200)
        }, 50)
      }, 30)
    }
  }
}
</script>

<style lang="stylus">
    .vue-simple-carousel-container
      width 100%
      .vsc-scroller
        overflow hidden
        .vsc-items
          display flex
          min-width 100%
          .vue-simple-carousel-item
            width 100%
            min-width 100vw
          &.moving
            .vue-simple-carousel-item
              position relative
              &::before
                content ""
                position absolute
                top 0
                left 0
                width 100%
                height 100%
                z-index 1
      .vsc-controls
        display flex
        justify-content space-between
        align-items center
        user-select none
        .vsc-points
          display flex
          justify-content center
          flex-wrap nowrap
          .vsc-point
            width 10px
            height 10px
            border-radius 50%
            margin 2px
            border 1px solid #000
            &.current
              background #000
        .arrow
          &.disabled
            display none
</style>
