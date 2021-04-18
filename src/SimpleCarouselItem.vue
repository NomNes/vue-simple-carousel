<template>
  <div class="vue-simple-carousel-item" :style="{width: `${100 / itemsOnScreen}%`}">
    <slot/>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component<SimpleCarouselItem>({
  mounted() {
    let parent = this.$parent
    while (true) {
      if (parent.$el && parent.$el.classList.contains('vue-simple-carousel-container')) {
        this.parent = parent
        break
      }
      if (!parent.$parent) {
        break
      }
      parent = parent.$parent
    }
  },
})
export default class SimpleCarouselItem extends Vue {
  parent: Vue | null = null

  get itemsOnScreen() {
    // @ts-ignore
    return this.parent ? this.parent.itemsOnScreen : 1
  }
}
</script>
