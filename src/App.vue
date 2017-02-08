<template>
  <div id="app">
    <input type="radio" id="LastRow" value="LastRow" v-model="type"  v-on:click="calcType('LastRow')">
    <label for="LastRow">Last Row Only</label>
    <input type="radio" id="ExactFit" value="ExactFit" v-model="type" v-on:click="calcType('ExactFit')">
    <label for="ExactFit">Grid Exact Fit</label>
 <span> ---------- </span>
  <input type="radio" id="differentSize" value="differentSize" v-model="imageType">
    <label for="differentSize">Different Sizes</label>
    <input type="radio" id="sameSize" value="sameSize" v-model="imageType">
    <label for="sameSize">Same Size</label>
    <Grid v-if="imageType === 'differentSize'" v-lastrow :imagesArray="imagesArrayDifferent"></Grid>
    <Grid v-if="imageType === 'sameSize'" v-lastrow :imagesArray="imagesArraySame"></Grid>
  </div>
</template>

<script>
  /* eslint-disable */
import $ from 'jquery';
import Grid from './components/Grid';
import { imagesArraySame, imagesArrayDifferent} from './assets/images';


//last row only solution------------------------
let $grid = $('.grid-container');
let prevLastRowCount = 0;
let $lastElem;
function updateLastRowStyle() {
   var start = window.performance.now();
  if ($lastElem) {
    $lastElem.remove();
  }
  $grid.removeClass('grid-container-after');
  const gridWrapped = $('.photo-container:first').outerHeight() < $grid.innerHeight();

  let prevRatio = null;
  let gridItems = {};
  let gridRows;
  let sameRatios = true;
  let difference = 0;
  let $item;


  if (gridWrapped) {
    $('.photo-container').get().reverse().some(function (item) {
      $item = $(item);
      const currentRatio = $item.css('flex-grow');//$item.innerHeight() / $item.outerHeight();
      const currentTop = $item.offset().top;
      gridItems[currentTop] = gridItems[currentTop] ? gridItems[currentTop]+=1 : 1;
      gridRows = Object.keys(gridItems);

      if (prevRatio && (currentRatio != prevRatio)) {
        sameRatios = false;
      }

      if (gridRows.length === 3) {
        return true;
      }

      prevRatio = currentRatio;
    });

    // When ratios are the same then we need to calc the exact spacer div to add at the end to keep images the same size
    if (sameRatios) {
       // Sort grid rows by highest top value (last row)
      gridRows.sort((a, b) => { const numA = Number(a); const numB = Number(b); return numA > numB ? 1 : numA < numB ? -1 : 0; });
      const numOfItemsLastRow = gridItems[gridRows[gridRows.length - 1]];
      const numOfItemsSecondToLastRow = gridItems[gridRows[gridRows.length-2]];
      difference = numOfItemsSecondToLastRow - numOfItemsLastRow;

      // if there is no difference between items in last row and items and in 2nd to last row then we dont need to add any spacer div
      if (!difference) {
        //console.log('there is no difference and ratios are the same :' + difference);
        var end = window.performance.now();
        console.warn(end - start)
        return;
      }
    }

    addDivAtEnd(sameRatios, difference, $item);
     var end = window.performance.now();
     console.warn(end - start)
  }

function addDivAtEnd(sameRatios, difference, $item) {
  $lastElem = $("<span class='js-last-elem'></span>").css({ border: '16px solid transparent' });

  if (sameRatios) {
    const psudeoFlexGrow = $item.css('flex-grow');//difference * $item.css('flex-grow');
    const psudeoWidth = difference * $item.outerWidth() - parseInt($item.css("border-left-width"), 10) - parseInt($item.css("border-right-width"), 10);
    $lastElem.css({ width: psudeoWidth });
   // console.log('psudeoFlexGrow: ' + psudeoFlexGrow);
    $('.grid-container').append($lastElem);
  }
  else {
    const lastItemMuchLarger = $('.photo-container:last').innerHeight() >= $('.photo-container:first').innerHeight() * 1.30; // 40% larger or whatever we think is acceptable
    if (lastItemMuchLarger) {
      $lastElem.css({ 'flex-grow': '99999999999' });
      $('.grid-container').append($lastElem);
    }
  }

}
}
//----last row only solution !!!!!!!!

//exact fit solution@@@@@@@@@@
const images = [];

  //TODO: insert psuedo element when we give up trying to jiggle
  //TODO: don't attempt to make any image larger than it's natural size!
  //TODO: get parameters from the design team!

  function calcWidths() {
    $('.photo-container').each(function() {
      const $elem = $(this);
      const width = $elem.width();
      const height = $elem.height();
      const flexWidth = parseFloat($elem.css('flex-grow'));
      const flexHeight = flexWidth * height / width;
      $elem.find('.width').remove();
      $elem.find('.max').remove();
      images.push({
        $elem,
        flexWidth,
        flexHeight
      });
    });
  }

  function adjustWidths(flexModifier) {
    images.forEach(function({ $elem, flexWidth }) {
      const newFlexWidth = flexModifier * flexWidth;

      $elem.css({
        flexGrow: newFlexWidth,
        width: newFlexWidth
      });
    });
  }

  function recalcWidths() {
    console.time('recalcWidths');

    const screenWidth = $(window).width(); // 1400
    const maxRatio = 1.25;
    const maxModifier = 1.5;

    let flexModifier = 1;
    let flexGrowth = .02;
    let last = 2;
    let avg = 1;
    let direction = -1;
    let rest;
    let rowHeights;

    while ( last / avg > maxRatio && flexModifier < maxModifier ) {
      flexGrowth += .05;
      direction *= -1;
      flexModifier = 1 + (flexGrowth * direction);
      rowHeights = getRowHeights(screenWidth, flexModifier);
      [ last, ...rest ] = rowHeights.reverse();
      avg = rest.reduce((sum, height) => sum + height, 0) / rest.length;
    }

    // If a good modifier was not found, use the original flexGrow
    if (flexModifier < maxModifier) {
      console.log('FOUND SUITABLE MODIFIER! Adjusting widths...');
      adjustWidths(flexModifier);
    }
    else {
      console.log('GAVE UP! Inserting large invisible div after last image...');
      adjustWidths(1);
      // TODO: insert psuedo-element so we have an incomplete last row,
      // which is preferable to a giant image!
    }

    console.timeEnd('recalcWidths');
  }

  function getRowHeights(screenWidth, flexModifier) {
    var rowHeights = [];
    var currentRow = [];
    var remaining = screenWidth;
    var sumOfWidths;


    images.forEach((image, index) => {
      var modifiedWidth = flexModifier * image.flexWidth;
      var modifiedHeight = flexModifier * image.flexHeight;

      if (remaining >= modifiedWidth) {
        remaining -= modifiedWidth;
      } else {
        sumOfWidths = currentRow.reduce(function(a, b) {
          return a + b.modifiedWidth;
        }, 0);

        rowHeights.push(currentRow[0].modifiedHeight * screenWidth / sumOfWidths);
        remaining = screenWidth - modifiedWidth;
        currentRow = [];
      }

      currentRow.push({
        modifiedWidth,
        modifiedHeight
      });
    });

    sumOfWidths = currentRow.reduce(function(a, b) {
      return a + b.modifiedWidth;
    }, 0);

    rowHeights.push(currentRow[0].modifiedHeight * screenWidth / sumOfWidths);

    return rowHeights
  }

//end exact fit solution @@@@
function bind(calcType) {
  $('.js-last-elem').remove();
  $(window).off('resize');
  if (calcType === 'LastRow') {
    $(window).resize(updateLastRowStyle);
    updateLastRowStyle();
  }
  if (calcType === 'ExactFit') {
    $(window).resize(recalcWidths);
    calcWidths();
    recalcWidths();
  }
}
export default {
  name: 'app',
  components: {
    Grid,
  },
  data() {
    return {
      type: 'ExactFit',
      imageType: 'differentSize',
      imagesArrayDifferent,
      imagesArraySame
    }
  },
  mounted: function () {
    //bind(this.type);
  },
 methods: {
    calcType: function (type) {
      bind(type);
    }
  },

  watch: {
    // whenever question changes, this function will run
    imageType: function (value) {
      bind(this.type);

    }
  },
  directives: {
    lastrow: {
      bind: function(value) {
          // $(window).resize(updateLastRowStyle);
          // $(window).resize(calcWidths);
      },
      inserted: (value, instance, vue) => {
          $grid = $(value);
          //updateLastRowStyle();
         // calcWidths();
      }
    }
  },
//  computed: {
//    imagesArray: function() {
//      if (this.imageType === 'differentSize') {
//        debugger;
//           return imagesArrayDifferent
//        }
//      if (this.imageType === 'sameSize') {
//          return imagesArraySame
//      }
//      return [ ]
//    }
//  }
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
