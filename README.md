# vue-simple-carousel
Simple carousel slider for Vue.js

## Installation
```
npm install vue-simple-slider
```

## Usage
```html
<template>
    <simple-carousel-container loop :watch-it="photos">
        <simple-carousel-item v-for="(photo, index) in photos" :key="index">
            <img :src="photo">
        </simple-carousel-item>
    </simple-carousel-container>
</template>

<script>
    import { SimpleCarouselContainer, SimpleCarouselItem } from 'vue-simple-carousel';
    export default {
      components: {
        SimpleCarouselContainer,
        SimpleCarouselItem
      },
      data() {
        return {
          photos: [
            'image-1.jpg',
            'image-2.jpg',
            'image-3.jpg',
          ]       
        }
      }
    };
</script>
```

### SimpleCarouselContainer props
Attribute | Type | Default | Description
:---|:---:|:---:|:---
**loop** | *Boolean* | *false* | Flag to make the carousel loop around when it reaches the end.
**watch-it** | *Any* | *undefined* | Mutable property for reactivity

### SimpleCarouselContainer methods
Method Name | Description | Parameter
:---|:---|:---:
**next** | Next slide | -
**previous** | Previous slide | -
**setCurrentIndex** | Set current slide | Slide index `Number`

### SimpleCarouselContainer events
Event name | Description | Parameter
:---|:---|:---:
**change** | Emitted when current index changed | Current index `Number`
