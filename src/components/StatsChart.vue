<template>
  <container class="stats-chart-container">
    <div class="chart-container">
      <div 
        v-for="(bar, barIndex) in 6"
        :key="bar"
        class="bar-container" 
        id="one-guess-wins"
      >
        <p class="bar-label-text" :style="isCurrentGuessCorrect && barIndex + 1 === highlightedNumber ? 'color: rgb(255, 174, 0);' : ''">{{ barIndex + 1 }}</p>
        <div 
          class="win-side-green" 
          :style="`width: ${percentages[barIndex]}%;`" 
        />
        <p class="stat-text-white">{{ fakeStats[barIndex] }}</p>
      </div>
    </div>
  </container>
</template>

<script lang='ts'>
import { computed, defineComponent } from "vue";

export default defineComponent({
  name: "StatsChart",
  props: {
    totalWins: {
      type: Number,
      required: true
    },
    highlightedNumber: {
      type: Number,
      required: true,
    },
    isCurrentGuessCorrect: {
      type: Boolean,
      required: true,
    }
  },
  setup(props) {
    const fakeStats = [
      5,
      12,
      25,
      42,
      56,
      100
    ]

    const percentages = computed(() => {
      const oneGuessWinPercentage = fakeStats[0] / props.totalWins
      const twoGuessWinPercentage = fakeStats[1] / props.totalWins
      const threeGuessWinPercentage = fakeStats[2] / props.totalWins
      const fourGuessWinPercentage = fakeStats[3] / props.totalWins
      const fiveGuessWinPercentage = fakeStats[4] / props.totalWins
      const sixGuessWinPercentage = fakeStats[5] / props.totalWins

      return [
        // Math.round(num * 100) / 100
        Math.round((oneGuessWinPercentage * 10000)) / 100,
        Math.round((twoGuessWinPercentage * 10000)) / 100,
        Math.round((threeGuessWinPercentage * 10000)) / 100,
        Math.round((fourGuessWinPercentage * 10000)) / 100,
        Math.round((fiveGuessWinPercentage * 10000)) / 100,
        Math.round((sixGuessWinPercentage * 10000)) / 100,
      ]
    })

    return {
      fakeStats,
      percentages
    };
  },
});
</script>

<style scoped>
.stats-chart-container {
  height: 70%;
  display: flex;

}

.chart-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  padding-left: 30%;
  padding-right: auto;
}

.bar-container {
  display: flex;
  width: 100%;
  align-items: stretch;
  margin-top: 4px;
  height: 10%;
}

.bar-labels-container {
  width: fit-content;
}

.win-side-green {
  background-color: rgb(62, 172, 62);
  align-self: stretch;
  border-radius: 0px 6px 6px 0px;
  padding-left: 4px;
}

.stat-text-white {
  color: white;
  font-family: 'Bungee Hairline', sans-serif;
  margin: 0;
  padding-left: 4px;
  font-size: 100%;
}

.bar-label {
  display: flex;
  justify-content: end;
  margin-top: 4px;
  padding-right: 6px;
  max-height: 22px;
  min-height: 22px;
}

.bar-label-text {
  font-family: "Monofett", sans-serif;
  color: white;
  margin: 0;
  font-size: 2em;
  line-height: 0.9;
  padding-left: 6px;
  padding-right: 6px;
  box-shadow: 0 0 0 2px rgb(62, 172, 62) inset;
  border-radius: 6px 0px 0px 6px;
  display: flex;
  align-items: center;
}

container {
  display: flex;
  justify-content: center;
  width: 100%;
}
</style>
