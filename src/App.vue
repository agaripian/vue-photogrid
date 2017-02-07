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
function updateLastRowStyle() {
  console.log('updateLastRowStyle');
  console.log($grid);
  $grid.find('.js-last-elem').remove();
  const gridWrapped = $('.photo-container:first').outerHeight() <= $grid.innerHeight();
  const lastItemMuchLarger = $('.photo-container:last').innerHeight() >= $('.photo-container:first').innerHeight() * 1.40;
  $grid.removeClass('grid-container-after');
  let prevRatio = null;
  let gridItems = {};
  let sameRatios = false;


  if (gridWrapped) {

  }
  $('.photo-container').get().reverse().some(function (item) {
    $item = $(item);
    const currentRatio = $item.innerHeight() / $item.outerHeight();
    const currentTop = $item.offset().top;
    gridItems[currentTop] = gridItems[currentTop] ? gridItems[currentTop]+=1 : 1;
  });




//TODO remove logic from loop, just loop to build the array
  setTimeout(() => {
    if (gridWrapped) {
      let gridRows;
      let gridItems = {};
      let $item;
      let divAdded = $('.photo-container').get().reverse().some(function (item) {

        $item = $(item);
        const currentRatio = $item.innerHeight() / $item.outerHeight();
        const currentTop = $item.offset().top;

        gridItems[currentTop] = gridItems[currentTop] ? gridItems[currentTop]+=1 : 1;

        gridRows = Object.keys(gridItems);
        const lastRowCount = gridItems[gridRows[gridRows.length - 1]];

        if (prevLastRowCount && prevLastRowCount === lastRowCount) {
          return true;
        }

        if (prevRatio && (currentRatio != prevRatio) && lastItemMuchLarger) {
          console.log('adding after!!!!')
          $grid.addClass('grid-container-after');
          return true;
        }


        if (gridRows.length === 3) {
          addDivAtEnd(gridItems, gridRows, $item);
          return true;
        }

        prevRatio = currentRatio;
        prevLastRowCount = lastRowCount;
      });

      if (!divAdded) {
        addDivAtEnd(gridItems, gridRows, $item);
      }

    }
  }, 0)


function addDivAtEnd(gridItems, gridRows, $item) {
  gridRows.sort((a, b) => { const numA = Number(a); const numB = Number(b); return numA > numB ? 1 : numA < numB ? -1 : 0; });
  const numOfItemsLastRow = gridItems[gridRows[gridRows.length - 1]];
  const numOfItemsSecondToLastRow = gridItems[gridRows[gridRows.length-2]];
  const difference = numOfItemsSecondToLastRow - numOfItemsLastRow;
  if (difference) {
    const psudeoFlexGrow = $item.css('flex-grow');//difference * $item.css('flex-grow');
    const psudeoWidth = difference * $item.outerWidth() - parseInt($item.css("border-left-width"), 10) - parseInt($item.css("border-right-width"), 10);
    var lastDiv = $("<span class='js-last-elem'></span>");
    lastDiv.css({ width: psudeoWidth, border: '16px solid transparent' })
    $('.grid-container').append(lastDiv);
    console.log('psudeoFlexGrow: ' + psudeoFlexGrow);
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
