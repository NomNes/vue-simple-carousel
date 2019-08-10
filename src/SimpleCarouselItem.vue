<template lang="pug">
    .vue-simple-carousel-item
        slot
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component
export default class SimpleCarouselItem extends Vue {

}
</script>
