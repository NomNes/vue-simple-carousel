<template>
  <div class="vcs">
    <div ref="overflow" class="vcs__overflow" :style="{ height: overflowHeight }">
      <div class="vcs__wrapper">
        <div class="vcs__slides" :style="slidesStyle">
          <div
            v-for="(item, index) in items"
            :key="index"
            ref="slides"
            :class="['vcs__slide', { 'vcs__slide--on_screen': index >= current && index <= maxOnScreen }]"
            :style="{ width: `${slideWidth}%` }"
            @dragstart.prevent
          >
            <slot :item="item" :index="index"/>
          </div>
        </div>
        <template v-if="sideNav && pagesCount && pagesCount > 1">
          <div
            :class="['vcs__nav-btn', 'vcs__nav-btn--side', 'vcs__nav-btn--prev', { 'vcs__nav-btn--disabled': !hasPrev }]"
            @mousedown="holdPrev"
            @touchstart="holdPrev"
          />
          <div
            :class="['vcs__nav-btn', 'vcs__nav-btn--side', 'vcs__nav-btn--next', { 'vcs__nav-btn--disabled': !hasNext }]"
            @mousedown="holdNext"
            @touchstart="holdNext"
          />
        </template>
      </div>
    </div>
    <div v-if="pages && pagesCount && pagesCount > 1" class="vcs__pages">
      <div
        :class="['vcs__nav-btn', 'vcs__nav-btn--prev', { 'vcs__nav-btn--disabled': !hasPrev }]"
        @mousedown="holdPrev"
        @touchstart="holdPrev"
      />
      <div
        v-for="p in pagesListShort"
        :key="p"
        :class="['vcs__page', { 'vcs__page--current': p === page }]"
        @click="goToPage(p)"
      >
        {{ p < 9 ? '0' : '' }}{{ p + 1 }}
      </div>
      <div
        :class="['vcs__nav-btn', 'vcs__nav-btn--next', { 'vcs__nav-btn--disabled': !hasNext }]"
        @mousedown="holdNext"
        @touchstart="holdNext"
      />
    </div>
    <div v-if="dots && pagesCount > 1" class="vcs__dots">
      <div
        v-for="p in pagesList"
        :key="p"
        :class="['vcs__dot', { 'vcs__dot--current': p === page }]"
        @click="goToPage(p)"
      />
    </div>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from 'vue'

function requestIdleCallback(cb: Function, options?: { timeout: number }): number | undefined {
  // @ts-ignore
  const ric = window.requestIdleCallback as Function
  if (ric) {
    return ric(cb, options)
  }
  cb()
}

export default Vue.extend({
  name: 'SimpleCarousel',
  props: {
    value: {
      type: Number,
      default: 0,
    },
    items: {
      type: Array as PropType<any[]>,
      required: true,
    },
    perPage: {
      type: [Number, Array] as PropType<number | [number, number][]>,
      default: 1,
    },
    slideByItem: {
      type: Boolean,
      default: false,
    },
    loop: {
      type: Boolean,
      default: false,
    },
    adjustable: {
      type: Boolean,
      default: false,
    },
    sideNav: {
      type: Boolean,
      default: false,
    },
    pages: {
      type: Boolean,
      default: false,
    },
    dots: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      overflowWidth: 0,
      overflowHeight: 'auto',
      current: 0,

      isTouch: typeof window !== 'undefined' && ('ontouchstart' in window),

      dragging: false,
      dragStartX: 0,
      dragStartY: 0,
      dragOffset: 0,
      sliderStartTranslate: 0,
      startTime: 0,
    }
  },
  computed: {
    overflow(): HTMLDivElement {
      return this.$refs.overflow as HTMLDivElement
    },
    currentPerPage(): number {
      if (typeof this.perPage === 'number') {
        return this.perPage
      }
      const pp = [...this.perPage].reverse()
      for (let i = 0; i < pp.length; i++) {
        if (this.overflowWidth >= pp[i][0]) {
          return pp[i][1]
        }
      }
      return 1
    },
    slideWidth(): number {
      return 100 / this.currentPerPage
    },
    sliderTranslate(): number {
      return 0 - this.slideWidth * this.current
    },
    slidesStyle(): Record<string, string> {
      return {
        transform: `translateX(${this.dragging ? this.dragTranslate : this.sliderTranslate}%)`,
        transition: this.dragging ? '' : '.5s all ease',
        alignItems: this.adjustable ? 'flex-start' : 'stretch',
      }
    },
    maxOnScreen(): number {
      return this.current + this.currentPerPage - 1
    },
    dragTranslate(): number {
      const percent = (this.dragOffset / this.overflowWidth) * 100
      return this.sliderStartTranslate - percent
    },
    hasPrev(): boolean {
      return this.current > 0
    },
    hasNext(): boolean {
      return this.current < this.maxIndex
    },
    maxIndex(): number {
      return this.items.length - this.currentPerPage
    },
    pagesCount(): number {
      if (this.slideByItem) {
        return this.items.length
      }
      return Math.ceil(this.items.length / this.currentPerPage)
    },
    page(): number {
      return Math.ceil(this.current / this.currentPerPage)
    },
    pagesList(): number[] {
      const list = []
      for (let i = 0; i < this.pagesCount; i++) {
        list.push(i)
      }
      return list
    },
    pagesListShort(): number[] {
      let min = this.page - 1
      let max = this.page + 2
      if (min < 0) {
        min = 0
        max += 1
      }
      if (max > this.pagesCount) {
        max = this.pagesCount
        if (min > 0) {
          min -= 1
        }
      }
      return this.pagesList.slice(min, max)
    },
  },
  watch: {
    current() {
      if (this.current !== this.value) {
        this.$emit('input', this.current)
      }
      this.updateOverflowHeight()
    },
    value() {
      if (this.current !== this.value) {
        this.current = this.value
      }
    },
    overflowWidth() {
      this.updateOverflowHeight()
      if (this.current > this.maxIndex) {
        this.current = this.maxIndex
      }
    },
    currentPerPage() {
      this.$emit('perPage', this.currentPerPage)
    },
  },
  mounted() {
    requestIdleCallback(() => {
      this.updateLayout()
      window.addEventListener('resize', this.updateLayout, { passive: true })
      if (this.isTouch) {
        this.overflow.addEventListener('touchstart', this.onStart, { passive: true })
      }
    })
  },
  beforeDestroy() {
    this.overflow.removeEventListener('touchstart', this.onStart)
    window.removeEventListener('resize', this.updateLayout)
  },
  methods: {
    updateLayout() {
      this.overflowWidth = this.overflow.clientWidth
    },
    updateOverflowHeight() {
      if (this.adjustable) {
        const slides = this.$refs.slides as Element[]
        if (slides) {
          let max = 0
          for (let i = this.current; i <= this.maxOnScreen; i++) {
            const h = slides[i].clientHeight
            if (h > max) {
              max = h
            }
          }
          if (max) {
            this.overflowHeight = `${max}px`
            return
          }
        }
      }
      this.overflowHeight = 'auto'
    },
    onStart(e: Event) {
      const event = e as TouchEvent
      document.addEventListener('touchend', this.onEnd, {
        passive: true,
        capture: true,
      })
      document.addEventListener('touchmove', this.onDrag, {
        passive: false,
        capture: true,
      })
      this.startTime = event.timeStamp
      this.dragging = true
      this.dragStartX = event.touches[0].clientX
      this.dragStartY = event.touches[0].clientY
      this.sliderStartTranslate = this.sliderTranslate
    },

    onEnd(e: Event) {
      const event = e as TouchEvent

      const eventPosX = event.changedTouches[0].clientX
      this.dragOffset = this.dragStartX - eventPosX
      const dragMomentum = this.dragOffset / (event.timeStamp - this.startTime)

      const delta = this.dragOffset / this.overflowWidth

      let newCurrent
      if (delta > 0) {
        newCurrent = this.current + Math.max(Math.round(delta), Math.round(dragMomentum))
      } else {
        newCurrent = this.current + Math.min(Math.round(delta), Math.round(dragMomentum))
      }
      if (newCurrent < 0) {
        newCurrent = 0
      }
      if (newCurrent > this.maxIndex) {
        newCurrent = this.maxIndex
      }
      this.current = newCurrent
      this.dragOffset = 0
      this.dragging = false

      document.removeEventListener('touchend', this.onEnd, true)
      document.removeEventListener('touchmove', this.onDrag, true)
    },

    onDrag(e: Event) {
      if (e.cancelable) {
        // e.preventDefault()
      }
      const event = e as TouchEvent
      const eventPosX = event.touches[0].clientX
      const eventPosY = event.touches[0].clientY
      const newOffsetX = this.dragStartX - eventPosX
      const newOffsetY = this.dragStartY - eventPosY
      if (Math.abs(newOffsetX) < Math.abs(newOffsetY)) {
        return
      }
      e.stopImmediatePropagation()
      this.dragOffset = newOffsetX
    },

    goPrev() {
      if (this.hasPrev) {
        let prev = this.current - (this.slideByItem ? 1 : this.currentPerPage)
        if (prev < 0) {
          prev = 0
        }
        this.current = prev
      } else if (this.loop) {
        this.current = this.maxIndex
      }
    },

    goNext() {
      if (this.hasNext) {
        let next = this.current + (this.slideByItem ? 1 : this.currentPerPage)
        if (next > this.maxIndex) {
          next = this.maxIndex
        }
        this.current = next
      } else if (this.loop) {
        this.current = 0
      }
    },

    goToPage(page: number) {
      if (page >= 0 && page < this.pagesCount) {
        let newCurrent = page * this.currentPerPage
        if (newCurrent > this.maxIndex) {
          newCurrent = this.maxIndex
        }
        this.current = newCurrent
      }
    },
    holdMove(fn: Function) {
      fn()
      let interval: any
      const reset = () => {
        clearInterval(interval)
        clearTimeout(timeout)
      }
      window.addEventListener('mouseup', reset, {
        once: true,
        passive: true,
      })
      window.addEventListener('touchend', reset, {
        once: true,
        passive: true,
      })
      const timeout = setTimeout(() => {
        fn()
        interval = setInterval(fn, 300)
      }, 1000)
    },

    holdNext() {
      this.holdMove(this.goNext)
    },

    holdPrev() {
      this.holdMove(this.goPrev)
    },
  },

})
</script>

<style lang="stylus">
gray = #333
black = #000

.vcs
  &__wrapper
    margin auto
    position relative
    max-height 100%
  &__overflow
    overflow hidden
    transition .5s height ease
  &__slides
    display flex
  &__slide
    width 100%
    flex 0 0 auto
    opacity .5
    position relative
    user-select none
    backface-visibility hidden
    -webkit-touch-callout none
    -webkit-tap-highlight-color rgba(0, 0, 0, 0)
    transition .5s opacity ease
    &::after
      content ''
      position absolute
      left 0
      top 0
      width 100%
      height 100%
      z-index 10
    &--on_screen
      opacity 1
      &::after
        display none
  &__pages
    display flex
    align-items center
    justify-content center
    margin-top 30px
  &__page
    font-size 14px
    display flex
    align-items center
    justify-content center
    width 39px
    height 32px
    margin 0 2px
    color inherit
    cursor pointer
    &--current
      font-weight bold
      font-size 18px
  &__nav-btn
    width 32px
    height 32px
    background rgba(114, 127, 140, .2)
    border-radius 3px
    position relative
    cursor pointer
    &::before
      content ''
      width 14px
      height 0
      border-bottom 1px solid gray
      position absolute
      left 50%
      top 50%
      margin-left -7px
    &::after
      content ''
      width 7px
      height 7px
      border-top 1px solid gray
      border-right 1px solid gray
      position absolute
      left 50%
      top 50%
      margin-top -3px
    &--prev
      margin-right 10px
      &::after
        transform rotate(-135deg)
        margin-left -7px
    &--next
      margin-left 10px
      &::after
        transform rotate(45deg)
    &--side
      position absolute
      top 50%
      z-index 10
      transform translateY(-50%)
      &.vcs__nav-btn
        &--prev
          left 0
        &--next
          right 0
    &--disabled
      opacity .5
  &__dots
    display flex
    justify-content center
    flex-wrap wrap
  &__dot
    width 10px
    height 10px
    background gray
    border-radius 50%
    margin 5px
    cursor pointer
    flex 0 0 auto
    &--current
      background black
</style>
