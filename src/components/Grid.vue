<template>
  <div>
    <transition-group name="flip-list" tag="div" class="grid-container">
    <div v-for="(item, index) in imagesArray" v-bind:key="index" :style="{ width: item.width*200/item.height + 'px',  'flex-grow': item.width*200/item.height}" v-bind:class="{ separator: item.className === 'separator', separatorend: item.className === 'separatorend'}"
      class="photo-container" v-lastrow @dragenter="dragEnter($event, index)" @drop="componentdrop(index)" @dragover.prevent="dragover($event, index)" ref="image"
           @dragend="dragend"  @dragstart="dragstart($event, index)" @dragleave.stop.prevent >
        <Photo :image="item" > </Photo>
      </div>
    </transition-group>
  </div>


</template>

<script>
  /* eslint-disable */
import Photo from './Photo';
export default {
  name: 'Grid',
  components: {
    Photo,
  },
  props: [
    'imagesArray'
  ],
  directives: {
    lastrow: {

    },
  },
  methods: {

    dragover(e, index){
      console.log("overimage index: ", index);
      var overImage = this.$refs.image[index];
      var xPos = e.x - overImage.offsetLeft;
      let className = 'separator';


      var tmpIndex = (xPos < (overImage.offsetWidth/2)) ? index : index + 1;

      //nothing has changed just return
      if (this.separatorIndex === tmpIndex && tmpIndex === index ){
        return;
      }

      //end of list
      if (tmpIndex === this.imagesArray.length) {
        className = 'separatorend';
        tmpIndex = tmpIndex - 1;
      }

      if (typeof this.separatorIndex !== 'undefined') {
        this.$delete(this.imagesArray[this.separatorIndex], 'className');
      }

      this.separatorIndex = tmpIndex;

      this.$set(this.imagesArray[this.separatorIndex], 'className', className);
      console.log('separator index ', this.separatorIndex)

      e.dataTransfer.dropEffect = 'move';
      e.preventDefault();
    },

    dragEnter(e, index) {
      console.log('dragEnter grid', index, e.x);
      e.preventDefault();
    },

    dragend(e){
      console.log('dragend');
      e.target.style.opacity = "";
      if (this.separatorIndex){
         this.$delete(this.imagesArray[this.separatorIndex], 'className');
      }

    },
    dragstart: function(e, index) {
      //var dragImage = e.currentTarget.cloneNode(true);
      // var dragImage = document.createElement('img');
      // dragImage.src = `../static/images/${this.imagesArray[index].url}`;

      // dragImage.style.transform = 'scale(.5)';
      // dragImage.style.border = '2px solid blue'


      // e.dataTransfer.setDragImage(dragImage, 0, 0);

      e.target.style.opacity = '.5';
      this.draggedItemIndex = index;
    },
    componentdrop: function(index) {
    this.$delete(this.imagesArray[this.separatorIndex], 'className');
    var image = this.imagesArray.splice(this.draggedItemIndex, 1)[0];

    let insertIndex = this.draggedItemIndex < this.separatorIndex ? this.separatorIndex - 1 : this.separatorIndex;

    this.$nextTick(function () {
        this.imagesArray.splice(insertIndex, 0, { url: image.url, height: image.height, width: image.width });
    })

    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  .flip-list-move {
   transition: transform 1s;

    // animation: scaleIt 500ms;
  }
  /*.flip-list-enter {
    width: 0px;
    height: 0px;
    transform: scale(0);
    //transform: translateY(30px);
    //transform: scale(.5);
  }

  .flip-list-enter-active {
    transition: all .3s ease;
    animation: scaleIt 500ms;
  }*/

  .grid-container {
    display: flex;
    flex-wrap: wrap;
  }
  .grid-container-after::after {
    content: '';
    flex-grow: 999999999;
    margin: 0 auto;
    text-align: center;
  }

  .photo-container {
    border: 16px solid transparent;
    position: relative;

  &.image-holder:not(.gu-mirror) {
     border: 16px solid transparent;
     animation:scaleIt 350ms;
   }
  }

</style>

<style lang="scss">
  .separator {
    width: 0;
    padding: 0;
    margin: 0;
    position: relative;

    &::before {
      width: 0px !important;
      height: 100%;
      background-color: cornflowerblue;
      border-left: 6px solid cornflowerblue ;
      content: '';
      position: absolute;
      top: 0;
      left: -18px;
    }
  }

  .separatorend {
    width: 0;
    padding: 0;
    margin: 0;
    position: relative;

    &::before {
      width: 0px !important;
      height: 100%;
      background-color: cornflowerblue;
      border-left: 6px solid cornflowerblue ;
      content: '';
      position: absolute;
      top: 0;
      right: -18px;
    }
  }

</style>
