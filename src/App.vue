<template>
  <div id="app">
    <Grid v-lastrow></Grid>
  </div>
</template>

<script>
  /* eslint-disable */
import $ from 'jquery';
import Grid from './components/Grid';
let $grid = '';
function updateLastRowStyle() {
  console.log('updateLastRowStyle');
  console.log($grid);
  $('.grid-container').find('.js-last-elem').remove();
  const gridWrapped = $('.photo-container:first').outerHeight() <= $grid.innerHeight();
  const lastItemMuchLarger = $('.photo-container:last').innerHeight() >= $('.photo-container:first').innerHeight() * 1.40;

  let prevRatio = null;
  let prevTop = null;
  let rowCountFromBottom = 1;
  let numItemsInSecondtoLastRow = 1;
  let numItemsInLastRow = 1;
  let gridItems = {};

  // if (gridWrapped && lastItemMuchLarger) {
  //  $grid.addClass('grid-container-after');
  // }

  if (gridWrapped) {
    $('.photo-container').get().reverse().some(function (item) {
      const $item = $(item);
      const currentRatio = $item.innerHeight() / $item.outerHeight();
      const currentTop = $item.offset().top;

      gridItems[currentTop] = gridItems[currentTop] ? gridItems[currentTop]+=1 : 1;

      const gridRows = Object.keys(gridItems);
      if (gridRows.length === 3) {
        gridRows.sort((a, b) => { const numA = Number(a); const numB = Number(b); return numA > numB ? 1 : numA < numB ? -1 : 0; });
        const numOfItemsLastRow = gridItems[gridRows[2]];
        const numOfItemsSecondToLastRow = gridItems[gridRows[1]];
        const difference = numOfItemsSecondToLastRow - numOfItemsLastRow;
        if (difference) {
          const psudeoFlexGrow = $item.css('flex-grow');//difference * $item.css('flex-grow');
          const psudeoWidth = difference * $item.outerWidth() - parseInt($item.css("border-left-width"), 10) - parseInt($item.css("border-right-width"), 10);
          var lastDiv = $("<span class='js-last-elem'></span>");
          lastDiv.css({ width: psudeoWidth, border: '16px solid transparent' })
          $('.grid-container').append(lastDiv);
          console.log('psudeoFlexGrow: ' + psudeoFlexGrow);
        }

        return true;
      }

      prevRatio = currentRatio;
    });
  }

function modifyAfterPsudoFlexGrow() {

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
