<template>
  <span class='wrapper'>
    <img :src="`../static/images/${image.url}`" @dragenter="">
    <i v-bind:style="{ paddingBottom: image.height / image.width * 100 + '%' }"></i>
  </span>
</template>

<script>
  /* eslint-disable */

  export default {
  name: 'Photo',
  data() {
    return {
      msg: 'Welcome to Your Vue.js App',
    };
  },
  props: [
    'image',
  ],

  methods: {
    dragenter(e) {
      console.log('in image drag enter');
      return false;
    },

    drop(e) {
      console.error('in photo drag over', e.target);

      return false;
    },

    setImageStyle() {
      const calcWidth = 100; // this.image.width*200/this.image.height;

      const styles = {
        width: `${calcWidth} + px`,
        'flex-grow': calcWidth,
      };
      return styles;
    },

    setPaddingBottom() {
     // return { 'padding-bottom': this.image.height / this.image.width*100 + '%'}
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  @keyframes growHorizontal {
    0% {
      opacity: 0;
      transform: scale(0);
    }
    100% {
      opacity: 1;
      transform: scale(1);
    }
  }

  .wrapper {
    display: flex;
    animation: growHorizontal 600ms;
  }

  img {
    position: absolute;
    top: 0;
    width: 100%;
    vertical-align: bottom;

  &:hover {
     filter: brightness(80%)
   }
  }

  i{
    display: block;
  }
</style>
