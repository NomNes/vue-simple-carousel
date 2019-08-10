# vue-simple-carousel
Simple carousel slider for Vue.js

## Installation
`npm install vue-simple-slider`

## Usage
```vue
<template>
    <simple-carousel-container loop>
        <simple-carousel-item>
            <img src="image-1.jpg">
        </simple-carousel-item>
        <simple-carousel-item>
            <img src="image-2.jpg">
        </simple-carousel-item>
        <simple-carousel-item>
            <img src="image-3.jpg">
        </simple-carousel-item>
    </simple-carousel-container>
</template>

<script>
    import { SimpleCarouselContainer, SimpleCarouselItem } from 'vue-simple-carousel';
    export default {
      components: {
        SimpleCarouselContainer,
        SimpleCarouselItem
      }
    };
</script>
```

### Configuration
Property | Type | Default | Description
:---|:---:|:---:|:---
**loop** | *Boolean* | *false* | Flag to make the carousel loop around when it reaches the end.
