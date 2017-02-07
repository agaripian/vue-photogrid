<template>
  <div id="app">
    <Grid v-lastrow></Grid>
  </div>
</template>

<script>
  /* eslint-disable */
import $ from 'jquery';
import Grid from './components/Grid';
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
    $grid.append($lastElem);
  }
  else {
    const lastItemMuchLarger = $('.photo-container:last').innerHeight() >= $('.photo-container:first').innerHeight() * 1.40; // 40% larger or whatever we think is acceptable
    if (lastItemMuchLarger) {
      $lastElem.css({ 'flex-grow': '99999999999' });
      $grid.append($lastElem);
    }
  }

}

};
export default {
  name: 'app',
  components: {
    Grid,
  },
  directives: {
    lastrow: {
      bind: function(value) {
           $(window).resize(updateLastRowStyle);
      },
      inserted: (value) => {
          $grid = $(value);
          updateLastRowStyle();
      }
    }
  },
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
