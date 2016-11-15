<template>
  <div class="grid-container">
    <template v-for="(item, index) in imagesArray" >
      <div :style="{ width: item.width*200/item.height + 'px',  'flex-grow': item.width*200/item.height }"
      class="photo-container" @dragenter="dragEnter($event, index)" @drop="newcomponentdrop"
           @dragend="dragend"  @dragstart.self="dragnewcompont" @dragleave.stop.prevent draggable="true">
        <Photo :image="item" > </Photo>
      </div>
      <div  v-bind:class="{ separator: item.separator }"></div>
    </template>
  </div>
</template>
<!--<template>
  <div class="grid-container">
    <div v-for="(item, index) in imagesArray" class="photo-container" :style="{ width: item.width*200/item.height + 'px',  'flex-grow': item.width*200/item.height }"
       @dragenter="dragEnter($event, index)" @drop="newcomponentdrop"
         @dragend="dragend"  @dragstart.self="dragnewcompont" @dragleave.stop.prevent  draggable="true">
      <Photo :image="item" @dra="drop"></Photo>
    </div>
  </div>
</template>-->

<script>
  /* eslint-disable */
import Photo from './Photo';
import { imagesArray } from '../assets/images';

export default {
  name: 'Grid',
  components: {
    Photo,
  },
  methods: {
    hasClass(el, name) {
      if (el && el.className) {
        return new RegExp('(?:^|\\s+)' + name + '(?:\\s+|$)').test(el.className);
      }
      return false;
    },

    addClass(el, name) {
      if (!this.hasClass(el, name)) {
        el.className = el.className ? [el.className, name].join(' ') : name;
      }
    },

    makeElement(){
      const newNode = document.createElement("div");
      newNode.classList.add( "separator");
      return newNode;
    },

    drop(e) {
      return false;
      console.log('dragOver', e)
    },
    dragEnter(e, index) {
      console.log('dragEnter grid', index, e.target);
      if (this.separatorIndex) {
        //delete[ this.imagesArray[this.separatorIndex].separator];
        this.$delete(this.imagesArray[this.separatorIndex], 'separator');
      }
      this.separatorIndex = index;
      this.$set(this.imagesArray[this.separatorIndex], 'separator', true)

      // if (!this.shadow){
      //   this.shadow = this.makeElement();
      //   this.shadow.classList.add('separator');
      // }
      // if (this.shadowIndex) {
      //   this.imagesArray.splice(this.shadowIndex, 1);
      // }
      // this.shadowIndex = index;

      // this.imagesArray.splice(index, 0, { separator: true });
      // delete[this.shadowIndex];


      //e.target.parentNode.insertBefore(this.shadow, e.target);

    },
    dragend(e){
      console.log('dragend')
    },
    dragnewcompont: function(e) {
      console.log('dragnewcomp')
      e.dataTransfer.setData('text', dataset );
      //call some event or set data to drag transfer
    },
    newcomponentdrop: function(e) {
      console.log('newcomponentdrop');
      var data = e.dataTransfer.getData("text");
      ////call some event or manaipulate data
    },
  },
  data() {
    return {
      imagesArray,
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  .grid-container {
    display: flex;
    flex-wrap: wrap;
  }
  .grid-container::after {
    content: '';
    flex-grow: 999999999;
    margin: 0 auto;
    text-align: center;
  }

  @keyframes straightLine {
    50% {
      transform: translate3D(100px, -100px, 0);
    }
  }

  @keyframes flex-out {
    0% {
      color:rgba(0,0,0,0);
      width:auto;
    }
    100% {
      color:rgba(0,0,0,1);
      width:500px;
    }
  }
  @keyframes flex-in {
    0% {
      color:rgba(0,0,0,1);
      width:500px;
    }
    100% {
      color:rgba(0,0,0,0);
      width:auto;
    }
  }

  .photo-container {
    border: 6px solid transparent;
    position: relative;

  &.image-holder:not(.gu-mirror) {
     border: 16px solid transparent;
     animation:scaleIt 350ms;
   }
  }

  .gu-mirror {
    position: fixed !important;
    margin: 0 !important;
    z-index: 9999 !important;
    opacity: 1;
    -ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=80)";
    filter: alpha(opacity=80);
    transition: 1s all;
    transform: scale(.5);

  // scale above isnt working with the animation below
  // webkit-animation: wiggle 0.3s 0s infinite ease-in;
  // animation: wiggle 0.3s 0s infinite ease-in;
  }

  @keyframes scaleIt {

    0% {
      display: none;
      opacity: 0;
    }
    1% {
      display: block;
      opacity: 0;
      transform: scale(0);
    }
    100% {
      opacity: .2;
      transform: scale(1);
    }
  }

  @-webkit-keyframes wiggle {
    0% {
      -webkit-transform: rotate(0deg);
    }
    25% {
      -webkit-transform: rotate(2deg);
    }
    75% {
      -webkit-transform: rotate(-2deg);
    }
    100% {
      -webkit-transform: rotate(0deg);
    }
  }

  @keyframes wiggle {
    0% {
      transform: rotate(-2deg);
    }
    25% {
      transform: rotate(2deg);
    }
    75% {
      transform: rotate(-2deg);
    }
    100% {
      transform: rotate(0deg);
    }
  }

  .gu-hide {
    display: none !important;
  }
  .gu-unselectable {
    -webkit-user-select: none !important;
    -moz-user-select: none !important;
    -ms-user-select: none !important;
    user-select: none !important;
  }

  .ex-over{
    filter: drop-shadow(0px 0px 10px rgba(0,0,0,.5));
  }

  // .separator {
     //   width: 2px;
     //   position: relative;
     //   background-color: cornflowerblue;
     //   border-left: 2px solid cornflowerblue ;
     //   border-right: 2px solid cornflowerblue ;
     //   content: '';
     //   border-radius: 2px;
     // }

  .gu-copy:not(.gu-mirror){
    opacity: .2;
  }

  .dragOver {

    &::before {
      width: 0px !important;
      height: 100%;
      background-color: cornflowerblue;
      border-left: 20px solid cornflowerblue ;
      content: '';
      position: absolute;
      top: 0;
      left: -2px;
    }
  }

</style>

<style lang="scss">
  .separator {
    width: 0;
  //height: 100%;
    padding: 0;
    margin: 0;
    position: relative;
  }
  .separator::before {
    width: 0px !important;
    height: 100%;
    background-color: cornflowerblue;
    border-left: 4px solid cornflowerblue ;
    content: '';
    position: absolute;
    top: 0;
    left: -2px;
  }
</style>
