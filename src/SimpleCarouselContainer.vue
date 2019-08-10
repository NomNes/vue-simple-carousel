<template lang="pug">
    .vue-simple-carousel-container
        .vsc-scroller(@touchstart="touchStart")
            .vsc-items(:style="{width: `${itemsLength * 100}%`}" ref="items" :class="{moving}")
                slot
        .vsc-controls
            .arrow.prev(v-if="hasPrev" @click="setCurrentIndex(currentIndex - 1)") &larr;
            .vsc-points
                .vsc-point(v-for="point in points" :key="point" :class="{current: currentIndex === point}" @click="setCurrentIndex(point)")
            .arrow.next(v-if="hasNext" @click="setCurrentIndex(currentIndex + 1)") &rarr;
</template>

<script lang="ts">
import { Component, Vue, Prop, Watch } from 'vue-property-decorator'

@Component
export default class SimpleCarouselContainer extends Vue {
  @Prop({ type: Boolean }) loop!: boolean;

  public currentIndex = 0;
  public moving = false;

  get itemsLength () {
    return this.$slots.default && this.$slots.default.length ? this.$slots.default.length : 0
  }

  get points () {
      return Array.from(Array(this.itemsLength).keys())
  }

  get hasNext () {
    return this.loop || this.currentIndex + 1 < this.itemsLength
  }
  get hasPrev () {
    return this.loop || this.currentIndex > 0
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

  touchStart () {

  }

  @Watch('currentIndex')
  onCurrentIndexChange (nextIndex: number, previousIndex: number) {
    this.moving = true
    let delta = nextIndex - previousIndex
    if (this.loop && Math.abs(this.itemsLength / delta) < 2) delta = (this.itemsLength - Math.abs(delta)) * (delta > 0 ? -1 : 1)
    if (delta > 0) {
      // @ts-ignore
      this.$refs.items.style.transition = '.2s'
      // @ts-ignore
      this.$refs.items.style.marginLeft = `${-100 * delta}%`
      setTimeout(() => {
        // @ts-ignore
        this.$refs.items.style.transition = '0s'
        this.updateOrder(nextIndex)
        setTimeout(() => {
          // @ts-ignore
          this.$refs.items.style.marginLeft = '0'
          this.moving = false
        })
      }, 200)
    } else {
      this.updateOrder(nextIndex)
      // @ts-ignore
      this.$refs.items.style.marginLeft = `${100 * delta}%`
      setTimeout(() => {
        // @ts-ignore
        this.$refs.items.style.transition = '.2s'
        // @ts-ignore
        this.$refs.items.style.marginLeft = '0'
        setTimeout(() => {
          // @ts-ignore
          this.$refs.items.style.transition = '0s'
          this.moving = false
        }, 200)
      })
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
                .vue-simple-carousel-item
                    width 100%
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
</style>
